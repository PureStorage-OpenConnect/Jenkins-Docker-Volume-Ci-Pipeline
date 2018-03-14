def GetNextFreePort() {
    def port = sh(returnStdout: true, script: 'ruby -e 'require \"socket\"; puts Addrinfo.tcp(\"\", 0).bind {|s| s.local_address.ip_port }'')
    return port.trim()
}

def StartContainer() {    
    sh "docker run -v ${VOLUME_NAME}:/var/opt/mssql -e \"ACCEPT_EULA=Y\" -e \"SA_PASSWORD=P@ssword1\" --name ${CONTAINER_NAME} -d -i -p ${PORT_NUMBER}:1433 microsoft/mssql-server-linux:2017-GA && sleep 10"    
    sh "/opt/mssql-tools/bin/sqlcmd -S localhost,${PORT_NUMBER} -U sa -P P@ssword1 -Q \"EXEC sp_configure 'show advanced option', '1';RECONFIGURE\""
    sh "/opt/mssql-tools/bin/sqlcmd -S localhost,${PORT_NUMBER} -U sa -P P@ssword1 -Q \"EXEC sp_configure 'clr enabled', 1;RECONFIGURE\""
    sh "/opt/mssql-tools/bin/sqlcmd -S localhost,${PORT_NUMBER} -U sa -P P@ssword1 -Q \"EXEC sp_configure 'clr strict security', 0;RECONFIGURE\""
}

def DeployDacpac() {
    def SqlPackage = "C:\\Program Files (x86)\\Microsoft SQL Server\\140\\DAC\\bin\\sqlpackage.exe"
    def SourceFile = "${SCM_PROJECT}\\bin\\Release\\${SCM_PROJECT}.dacpac"
    def ConnString = "server=${CONTAINER_HOST_IP_ADDR},${PORT_NUMBER};database=SsdtDevOpsDemo;user id=sa;password=P@ssword1"
    unstash 'theDacpac'
    bat "\"${SqlPackage}\" /Action:Publish /SourceFile:\"${SourceFile}\" /TargetConnectionString:\"${ConnString}\" /p:ExcludeObjectType=Logins"
}

def GetScmProjectName() {
    def scmProjectName = scm.getUserRemoteConfigs()[0].getUrl().tokenize('/').last().split("\\.")[0]
    return scmProjectName.trim()
}

pipeline {
    agent any
    
    environment {
        PORT_NUMBER            = GetNextFreePort()
        SCM_PROJECT            = GetScmProjectName()
        CONTAINER_NAME         = "SQLLinux${env.BRANCH_NAME}"
        VOLUME_NAME            = "${SCM_PROJECT}_${env.BRANCH_NAME}_${env.BUILD_NUMBER}"
    }

    parameters {
        booleanParam(defaultValue: true           , description: '', name: 'HAPPY_PATH')
              string(defaultValue: '10.223.112.98', description: '', name: 'CONTAINER_HOST_IP_ADDR')
    }
    
    stages {
        stage('git checkout') {     
            steps {
                timeout(time: 5, unit: 'SECONDS') {
                    checkout scm
                }
            }
        }
        stage('build dacpac') {
            steps {
                bat "\"${tool name: 'Default', type: 'msbuild'}\" /p:Configuration=Release"
                stash includes: "${SCM_PROJECT}\\bin\\Release\\${SCM_PROJECT}.dacpac", name: 'theDacpac'
            }
        }
    
        stage('start container') {
            agent {
                label "linux-agent"
            }
            steps {
                timeout(time: 20, unit: 'SECONDS') {
                    StartContainer()
                }
            }
        }
    
        stage('deploy dacpac') {
            steps {
                timeout(time: 60, unit: 'SECONDS') {
                   DeployDacpac()
                }
            }
        }
        
        stage('run tests (Happy path)') {
            when {
                expression {
                    return params.HAPPY_PATH
                }
            }
            steps {
                bat "sqlcmd -S ${CONTAINER_HOST_IP_ADDR},${PORT_NUMBER} -U sa -P P@ssword1 -d SsdtDevOpsDemo -Q \"EXEC tSQLt.Run \'tSQLtHappyPath\'\""
                bat "sqlcmd -S ${CONTAINER_HOST_IP_ADDR},${PORT_NUMBER} -U sa -P P@ssword1 -d SsdtDevOpsDemo -y0 -Q \"SET NOCOUNT ON;EXEC tSQLt.XmlResultFormatter\" -o \"${WORKSPACE}\\${SCM_PROJECT}.xml\"" 
                junit "${SCM_PROJECT}.xml"
            }
        }

        stage('run tests (Un-happy path)') {
            when {
                expression {
                    return !(params.HAPPY_PATH)
                }
            }
            steps {
                bat "sqlcmd -S ${CONTAINER_HOST_IP_ADDR},${PORT_NUMBER} -U sa -P P@ssword1 -d SsdtDevOpsDemo -Q \"EXEC tSQLt.Run \'tSQLtUnhappyPath\'\""
                bat "sqlcmd -S ${CONTAINER_HOST_IP_ADDR},${PORT_NUMBER} -U sa -P P@ssword1 -d SsdtDevOpsDemo -y0 -Q \"SET NOCOUNT ON;EXEC tSQLt.XmlResultFormatter\" -o \"${WORKSPACE}\\${SCM_PROJECT}.xml\"" 
                junit "${SCM_PROJECT}.xml"
            }
        }        
    }
    post {
        always {                  
            print 'post: Always'
            node ('linux-agent') {
                sh "docker rm -f ${CONTAINER_NAME}"
            }
        }
        success {
            //
            // tSQLt tests have passed, therefore take no action which should mean
            // that the Docker volume is available for future use
            //
            print 'post: Success'
        }
        unstable {
            print 'post: Unstable'
            node ('linux-agent') {
                sh "docker volume rm -f ${VOLUME_NAME}"
            }
        }
        failure {
            print 'post: Failure'
        }
    }
}
