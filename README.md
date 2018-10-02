# The Container "Elephant in The Room"

![elephant](https://user-images.githubusercontent.com/15145995/46348469-0396eb00-c647-11e8-9102-61ae33966e5e.PNG)

# The "Builder Pattern"

![builder pattern](https://user-images.githubusercontent.com/15145995/46348476-0bef2600-c647-11e8-9aa8-2a1e2b1ebd8f.PNG)

# The Pipeline This Repo Demonstrates

![docker volume workflow](https://user-images.githubusercontent.com/15145995/46348665-9afc3e00-c647-11e8-9655-3f46a785a9ee.PNG)

This repository contains a SQL Server data tools project and a Jenkinsfile for a build pipeline that carries out the following workflow:

1. Checks the project out from SCM
2. Uses msbuild (with SQL Server as the target) to create a DACPAC, this is an artefact that can be deployed to a SQL Server database via the data tier application framework
3. Spins up a container to deploy the DacPac to **in serial**, the container:
  - is named **[SQLLinux|branch name]**
  - is assigned a Docker volume named **[SCM project name|branch name|build number]**
  - has a unqiue external port so as to avoid port clashes
4. Deploys the DacPac to the container
5. Performs tSQLt unit tests **in serial**
   - Renders the tSQLt unit test results to Jenkins using the JUnit plugin
   - Tears down the container
6. Removes the Docker volume if any of the tSQLt tests have failed

# Prerequisites

To use this sample pipeline the following pre-requisites must be met:

## Jenkins Windows Build master

1. Jenkins 2.0 installed along with the git, msbuild and pipeline plugins, refer to the following link:
  
   https://jenkins.io/download/
  
   this can either be a virtual or physical machine.   

2. DACFX installed on the windows server, refer to the following link:
   https://www.microsoft.com/en-us/download/details.aspx?id=55114
  
3. SQL Server client tools (minimum version 2016)

## Linux node Build slave

1. A server with one of the following distributions of Linux installed:
 - CentOS Linux 7.3
 - CoreOS (Ladybug 1298.6.0 and above)
 - RedHat RHEL7
 - Ubuntu (Trusty 14.04 LTS, Xenial 16.04.2 LTS)

2. An installation of GIT, *note* that the path to executable needs to be specified in the properties for the Linux node:

![node_properties](https://user-images.githubusercontent.com/15145995/38550704-254e645c-3caf-11e8-8fa4-97835a88b470.PNG)

3. SQL Server client tools (minimum version 2016)
  
4. Java 8 SDK   

5. The Pure Storage FlashArray Docker Volume plugin should bew downloaded and installed from the Docker store:
   https://store.docker.com/plugins/pure-docker-volume-plugin 

Documentation for setting up a Jenkins build node can be found by following this link:
https://wiki.jenkins.io/display/JENKINS/Step+by+step+guide+to+set+up+master+and+slave+machines+on+Windows

## Storage

At the time of writing the Linux node requires iSCSI connectivity to the FlashArray

# Points Of Interest From This Repo

![poi](https://user-images.githubusercontent.com/15145995/46348482-10b3da00-c647-11e8-9084-66b3d8ee29e5.PNG)





