def getNextFreePort() {
    def port = powershell(returnStdout: true, script: '((Get-NetTCPConnection | Sort-Object -Property LocalPort | Select-Object -Last 1).LocalPort) + 1')
    return port.trim()
}

def StartContainer() {
    bat "docker run -e \"ACCEPT_EULA=Y\" -e \"SA_PASSWORD=P@ssword1\" --name SQLLinuxMaster -d -i -p ${PORT_NUMBER}:1433 microsoft/mssql-server-linux:2017-GA"    
    powershell 'While (\$((docker logs SQLLinuxMaster | select-string ready | select-string client).Length) -eq 0) { Start-Sleep -s 1 }'    
    bat "sqlcmd -S localhost,${PORT_NUMBER} -U sa -P P@ssword1 -Q \"EXEC sp_configure 'show advanced option', '1';RECONFIGURE\""
    bat "sqlcmd -S localhost,${PORT_NUMBER} -U sa -P P@ssword1 -Q \"EXEC sp_configure 'clr enabled', 1;RECONFIGURE\""
    bat "sqlcmd -S localhost,${PORT_NUMBER} -U sa -P P@ssword1 -Q \"EXEC sp_configure 'clr strict security', 0;RECONFIGURE\""
}

def DeployDacpac() {
    def SqlPackage = "C:\\Program Files\\Microsoft SQL Server\\140\\DAC\\bin\\sqlpackage.exe"
    def SourceFile = "${SCM_PROJECT}\\bin\\Release\\${SCM_PROJECT}.dacpac"
    def ConnString = "server=localhost,${PORT_NUMBER};database=SsdtDevOpsDemo;user id=sa;password=P@ssword1"
    unstash 'theDacpac'
    bat "\"${SqlPackage}\" /Action:Publish /SourceFile:\"${SourceFile}\" /TargetConnectionString:\"${ConnString}\" /p:ExcludeObjectType=Logins"
}

def getScmProjectName() {
    def scmProjectName = scm.getUserRemoteConfigs()[0].getUrl().tokenize('/').last().split("\\.")[0]
    //def volumeName = "${repoName}_${env.BRANCH_NAME}_${env.BUILD_NUMBER}"
    return scmProjectName.trim()
}

pipeline {
    agent any
    
    environment {
        PORT_NUMBER = getNextFreePort()
        SCM_PROJECT = getScmProjectName()
    }
    
    parameters {
        booleanParam(defaultValue: true, description: '', name: 'HAPPY_PATH')
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
    
        //stage('start container') {
        //    steps {
        //        timeout(time: 20, unit: 'SECONDS') {
        //            StartContainer()
        //        }
        //    }
        //}
    
        //stage('deploy dacpac') {
        //    steps {
        //        timeout(time: 60, unit: 'SECONDS') {
        //           DeployDacpac()
        //        }
        //    }
        //}
        
        //stage('run tests (Happy path)') {
        //    when {
        //        expression {
        //            return params.HAPPY_PATH
        //        }
        //    }
        //    steps {
        //        bat "sqlcmd -S localhost,${PORT_NUMBER} -U sa -P P@ssword1 -d SsdtDevOpsDemo -Q \"EXEC tSQLt.Run \'tSQLtHappyPath\'\""
        //        bat "sqlcmd -S localhost,${PORT_NUMBER} -U sa -P P@ssword1 -d SsdtDevOpsDemo -y0 -Q \"SET NOCOUNT ON;EXEC tSQLt.XmlResultFormatter\" -o \"${WORKSPACE}\\${SCM_PROJECT}.xml\"" 
        //        junit '${SCM_PROJECT}.xml'
        //    }
        //}

        //stage('run tests (Un-happy path)') {
        //    when {
        //        expression {
        //            return !(params.HAPPY_PATH)
        //        }
        //    }
        //    steps {
        //        bat "sqlcmd -S localhost,${PORT_NUMBER} -U sa -P P@ssword1 -d SsdtDevOpsDemo -Q \"EXEC tSQLt.Run \'tSQLtUnhappyPath\'\""
        //        bat "sqlcmd -S localhost,${PORT_NUMBER} -U sa -P P@ssword1 -d SsdtDevOpsDemo -y0 -Q \"SET NOCOUNT ON;EXEC tSQLt.XmlResultFormatter\" -o \"${WORKSPACE}\\${SCM_PROJECT}.xml\"" 
        //        junit '${SCM_PROJECT}.xml'
        //    }
        //}
    }
    post {
        always {
            powershell 'If (\$((docker ps -a --filter \"name=SQLLinuxMaster\").Length) -eq 2) { docker rm -f SQLLinuxMaster }'
        }
        success {
            print 'post: Success'
        }
        unstable {
            print 'post: Unstable'
        }
        failure {
            print 'post: Failure'
        }
    }
}
