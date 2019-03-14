# "Elephant in The Room"

![elephant](https://user-images.githubusercontent.com/15145995/46348469-0396eb00-c647-11e8-9102-61ae33966e5e.PNG)
---
# "Builder Pattern"

![builder pattern](https://user-images.githubusercontent.com/15145995/46348476-0bef2600-c647-11e8-9aa8-2a1e2b1ebd8f.PNG)
---
# Pipeline

![docker volume workflow](https://user-images.githubusercontent.com/15145995/46348665-9afc3e00-c647-11e8-9655-3f46a785a9ee.PNG)
---
# Workflow

1. Checks the project out from SCM
2. Uses msbuild (with SQL Server as the target) to create a DACPAC, this is an artefact that can be deployed to a SQL Server database via the data tier application framework
3. Spins up a container to deploy the DacPac to **in serial**, the container:
  - is named **[SQLLinux|branch name]**
  - is assigned a Docker volume named **[SCM project name|branch name|build number]**
  - has a unqiue external port so as to avoid port clashes
---
# Workflow cont . . .

4. Deploys the DacPac to the container
5. Performs tSQLt unit tests **in serial**
   - Renders the tSQLt unit test results to Jenkins using the JUnit plugin
   - Tears down the container
6. Removes the Docker volume if any of the tSQLt tests have failed
---
# Points Of Interest

![poi](https://user-images.githubusercontent.com/15145995/46348482-10b3da00-c647-11e8-9084-66b3d8ee29e5.PNG)
