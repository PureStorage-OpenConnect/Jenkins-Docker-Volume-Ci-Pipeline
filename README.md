# Overview

This repository contains a SQL Server data tools project and a Jenkinsfile written using the opinionated declarative syntax for a build pipeline that carries out the following workflow:

![workflow](https://user-images.githubusercontent.com/15145995/38548952-f5750942-3caa-11e8-8291-20f593eb1c34.PNG)

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

- A Windows server with Jenkins 2.0 installed along with the git, msbuild and pipeline plugins, refer to the following link:
  https://jenkins.io/download/
  this can either be a virtual or physical machine.   

- SQL Server client tools (minimum version 2016) installed on the Windows server

- DACFX installed on the windows server, refer to the following link:
  https://www.microsoft.com/en-us/download/details.aspx?id=55114
  
- A Linux server with a distribution of Linux that supports Docker and SQL Server client tools for Linux, when testing this CentOS was used, *note* that   
