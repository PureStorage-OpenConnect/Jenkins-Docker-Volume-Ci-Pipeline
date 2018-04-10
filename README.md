# Overview

This repository contains a SQL Server data tools project and a Jenkinsfile written using the opinionated declarative syntax for a build pipeline that:

![workflow](https://user-images.githubusercontent.com/15145995/38548952-f5750942-3caa-11e8-8291-20f593eb1c34.PNG)

- Checks the project out from SCM
- Spins up a container to deploy the DacPac to **in serial**, the container:
  - is named **[SQLLinux|branch name]**
  - is assigned a Docker volume named **[SCM project name|branch name|build number]**
  - has a unqiue external port so as to avoid port clashes
- Deploys the DacPac to the container
- Performs tSQLt unit tests **in serial**
- Renders the tSQLt unit test results to Jenkins using the JUnit plugin
- Tears down the container
- Removes the Docker volume if any of the tSQLt tests have failed
