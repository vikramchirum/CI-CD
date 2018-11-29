# deployment.bootstrap

Files and instructions needed to setup a VS solution to build in bamboo using grunt tasks with proper versioning

## Pre-Setup (One time per machine.  Not necessary for each project)

1. Download and install nodejs [https://nodejs.org/en/download/](node.js)
2. Download the nuget cli [https://www.nuget.org/downloads](NuGet)
3. Run these from the directory where you downloaded the nuget.exe
    * ```nuget sources add -Name Nexus -source http://neer-nexus:8084/nexus/service/local/nuget/Nuget.Gexa/ -User phoenix -pass phoenix```
    * ```nuget.exe config -set http_proxy=http://webproxygo.fpl.com:8080```
4. Run this from any command window ```npm config set proxy http://webproxygo.fpl.com:8080```


## Initial Setup

1. Copy GruntFile.js and package.json to your solution folder
2. Change the package.json name and version to match yours
3. Run ```npm install```
4. If your package name differs from your solution or what you call your app in octopus or nexus then you may have to make changes to the GruntFile.js

## Web Projects

If your solution has web projects in it that need to be sent to octopus then:

1. Install this nuget package for each web project [MSBuild.Microsoft.VisualStudio.Web_WebApplication.Targets](https://www.nuget.org/packages/MSBuild.Microsoft.VisualStudio.Web_WebApplication.Targets/12.0.2)
2. Replace the first set of lines with the following line in any of the csproj files that are web projects

Set 1
```
  <Import Project="$(VSToolsPath)\WebApplications\Microsoft.WebApplication.targets" Condition="'$(VSToolsPath)' != ''" />
  <Import Project="$(MSBuildExtensionsPath32)\Microsoft\VisualStudio\v10.0\WebApplications\Microsoft.WebApplication.targets" Condition="false" />
```  

Set 2
```
  <Import Project="$(SolutionDir)\packages\MSBuild.Microsoft.VisualStudio.Web_WebApplication.Targets.12.0.2\tools\VSToolsPath\WebApplications\Microsoft.WebApplication.targets" />
```

This is because the build server may not have these target files installed on it (We can remove this at a later date if the build server has them)

**Important**

If your solution has web projects that reference the new application targets then when other people clone your repo for the first time they will need to run

```grunt nugetrestore``` from the command line or from visual studio task runner explorer so that the project will load

## Octopus Applications

For each project that needs to be deployed to octopus, install this nuget package

[https://www.nuget.org/packages/OctoPack/](OctoPack)

## Libraries

If your solution has projects that will become nexus nuget packages then you will have to modify the GruntFile.js at the bottom.

1. Make sure that ```grunt.registerTask('build', ['clean', 'assemblyinfo', 'nugetrestore', 'msbuild']);``` is commented out
2. Make sure that ```grunt.registerTask('build', ['clean', 'assemblyinfo', 'nugetrestore', 'msbuild', 'nugetpack']);``` is uncommented

## Test your setup

On a command line run ```grunt build --branch_name=dev```

Then try ```grunt build --branch_name=test_branch\with/special_characters```

If they both complete successfully you should have packages in either /.build/packages/octo or /.build/packages/nuget in your solution file and your build worked

## Testing

The only library included in this bootstrap is for mstest but there are many out there for NUnit and others

If you are using mstest then you can run ```grunt mstest``` and you should see your test output (```grunt msbuild``` or ```grunt build``` must have been run prior)

## Bamboo Setup

**Set these variables up in your plan**

Only include nexus variables if you have nuget packages to publish

Only include octopus variables if you have apps to publish

| Name        | Value                                                        |
| ----------- | ------------------------------------------------------------ |
| nexusApiKey | 6785b083-2697-3f8b-a734-27ee29b285ff                         |
| nexusSource | http://neer-nexus:8084/nexus/service/local/nuget/Nuget.Gexa/ |
| octoApiKey  | API-FLRBYNHIX14CXBV82R6CH7BGYEY                              |
| octoServer  | http://octopus/                                              |

**Tasks**

