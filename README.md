# Library Management System


## Installation

```console
dotnet resotre

```

## Build the Tool from source

You can build and package the tool using the following commands. The instructions assume that you are in the root of the repository.

```console
dotnet build
```

Knowing That I made an app service to deploy the app to it (https://librarymsbackend.azurewebsites.net/swagger/index.html) as you can see in the screenshot

![alt text](https://github.com/RamiAlkhateeb/LibraryManagementSystem/blob/master/azure.png?raw=true)

and I did several pipeline files and tried several way to configure CI/CD through them, 
the one I've created through dev ops didn't run because of the following error, 

![alt text](https://github.com/RamiAlkhateeb/LibraryManagementSystem/blob/master/pipeline.png?raw=true)

I sent a request to Microsoft, it needs 3 days to be approved
I will write here that pipeline
``` yaml
# YAML
trigger:
- master

pool:
  vmImage: 'windows-latest'

variables:
  solution: '**/*.sln'
  buildPlatform: 'Any CPU'
  buildConfiguration: 'Release'

steps:
- task: NuGetToolInstaller@1

- task: NuGetCommand@2
  inputs:
    restoreSolution: '$(solution)'

- task: VSBuild@1
  inputs:
    solution: '$(solution)'
    msbuildArgs: '/p:DeployOnBuild=true /p:WebPublishMethod=Package /p:PackageAsSingleFile=true /p:SkipInvalidConfigurations=true /p:DesktopBuildPackageLocation="$(build.artifactStagingDirectory)\WebApp.zip" /p:DeployIisAppPath="Default Web Site"'
    platform: '$(buildPlatform)'
    configuration: '$(buildConfiguration)'
```


## So I made a pipeline in github actions, which WORKED
https://github.com/RamiAlkhateeb/LibraryManagementSystem/actions

``` yaml
name: Publish

on:
  workflow_dispatch:
  push:
    branches:
    - master
    
env:
  AZURE_WEBAPP_NAME: LibraryMSBackend
  AZURE_WEBAPP_PACKAGE_PATH: "./publish"

jobs:
  Publish:
    runs-on: windows-latest
    
    steps:
      - uses: actions/checkout@v3
      
      - name: Setup .Net
        uses: actions/setup-dotnet@v2
        with:
          dotnet-version: '6.0.x'
          
      - name: Restore
        run: dotnet restore ./LibraryManagementSystem.sln
        
      - name: Build
        run: dotnet build ./LibraryManagementSystem.sln --configuration Release
        
      - name: Publish
        run: dotnet publish ./LibraryManagementSystem.sln --configuration Release --output './publish'
      
      - name: deployment
        uses: azure/webapps-deploy@v2
        with:
          app-name: ${{ env.AZURE_WEBAPP_NAME }}
          publish-profile: ${{ secrets.AZURE_PUBLISH_PROFILE }}
          package: ${{ env.AZURE_WEBAPP_PACKAGE_PATH }}

```

Knowing that screenshot of the app runing will be attached within the rect repo


#### Note: I didn't add any validation, only the happy way works