---
title: Build, test, and deploy .NET Core apps
description: Use .NET Core to build apps with Azure Pipelines, Azure DevOps, & Team Foundation Server
ms.topic: conceptual
ms.assetid: 95ACB249-0598-4E82-B155-26881A5AA0AA
ms.reviewer: vijayma
ms.date: 07/29/2021
ms.custom: contperf-fy20q4
monikerRange: '>= tfs-2017'
---

# Build, test, and deploy .NET Core apps

[!INCLUDE [version-tfs-2017-rtm](../includes/version-tfs-2017-rtm.md)]

Use a pipeline to automatically build and test your .NET Core projects. Learn how to:

* Set up your build environment with [Microsoft-hosted](../agents/hosted.md) or [self-hosted](../agents/agents.md) agents.
* Restore dependencies, build your project, and test with the [.NET Core CLI task](../tasks/build/dotnet-core-cli.md) or a [script](../scripts/cross-platform-scripting.md).
* Use the [publish code coverage task](../tasks/test/publish-code-coverage-results.md) to publish code coverage results.
* Package and deliver your code with the [.NET Core CLI task](../tasks/build/dotnet-core-cli.md) and the [publish build artifacts task](../tasks/utility/publish-build-artifacts.md).
* Publish to a [NuGet feed](../artifacts/nuget.md).
* Deploy your [web app to Azure](../targets/webapp.md).


> [!NOTE]
> For help with .NET Framework projects, see [Build ASP.NET apps with .NET Framework](../apps/aspnet/build-aspnet-4.md).
> 

[!INCLUDE [temp](../includes/concept-rename-note.md)]

::: moniker range="tfs-2017"

> [!NOTE]
> 
> This guidance applies to TFS version 2017.3 and newer.

::: moniker-end

## Create your first pipeline

::: moniker range=">=azure-devops-2020"

> Are you new to Azure Pipelines? If so, then we recommend you try this section before moving on to other sections.

::: moniker-end

### Get the code

::: moniker range=">=azure-devops-2020"

[!INCLUDE [include](includes/get-code-before-sample-repo.md)]

::: moniker-end

::: moniker range="azure-devops-2019"

Import this repo into your Git repo in Azure DevOps Server 2019:

::: moniker-end

::: moniker range="< azure-devops-2019"

Import this repo into your Git repo in TFS:

::: moniker-end

```
https://github.com/MicrosoftDocs/pipelines-dotnet-core
```

::: moniker range=">=azure-devops-2020"

### Sign in to Azure Pipelines

[!INCLUDE [include](includes/sign-in-azure-pipelines.md)]

[!INCLUDE [include](includes/create-project.md)]

::: moniker-end

### Create the pipeline

::: moniker range=">=azure-devops-2020"

[!INCLUDE [include](includes/create-pipeline-before-template-selected.md)]

> When the **Configure** tab appears, select **ASP.NET Core**.

1. When your new pipeline appears, take a look at the YAML to see what it does. When you're ready, select **Save and run**.

   > [!div class="mx-imgBorder"] 
   > ![Save and run button in a new YAML pipeline](media/save-and-run-button-new-yaml-pipeline.png)

2. You're prompted to commit a new _azure-pipelines.yml_ file to your repository. After you're happy with the message, select **Save and run** again.

   If you want to watch your pipeline in action, select the build job.

   > You just created and ran a pipeline that we automatically created for you, because your code appeared to be a good match for the [ASP.NET Core](https://github.com/Microsoft/azure-pipelines-yaml/blob/master/templates/asp.net-core.yml) template.

   You now have a working YAML pipeline (`azure-pipelines.yml`) in your repository that's ready for you to customize!

3. When you're ready to make changes to your pipeline, select it in the **Pipelines** page, and then **Edit** the `azure-pipelines.yml` file.

4. See the sections below to learn some of the more common ways to customize your pipeline.

::: moniker-end

::: moniker range="azure-devops-2019"

### YAML
1. Add an `azure-pipelines.yml` file in your repository. Customize this snippet for your build. 

```yaml
trigger:
- master

pool: Default

variables:
  buildConfiguration: 'Release'

# do this before all your .NET Core tasks
steps:
- task: DotNetCoreInstaller@2
  inputs:
    version: '2.2.402' # replace this value with the version that you need for your project
- script: dotnet build --configuration $(buildConfiguration)
  displayName: 'dotnet build $(buildConfiguration)'
```
2. Create a pipeline (if you don't know how, see [Create your first pipeline](../create-first-pipeline.md)), and for the template select **YAML**.

3. Set the **Agent pool** and **YAML file path** for your pipeline. 

4. Save the pipeline and queue a build. When the **Build #nnnnnnnn.n has been queued** message appears, select the number link to see your pipeline in action.

5. When you're ready to make changes to your pipeline, **Edit** it.

6. See the sections below to learn some of the more common ways to customize your pipeline.

::: moniker-end
::: moniker range="< azure-devops" 
### Classic

1. Create a pipeline (if you don't know how, see [Create your first pipeline](../create-first-pipeline.md)). Select **Empty Pipeline** for the template. 

2. In the task catalog, find and add the **.NET Core** task. This task will run `dotnet build` to build the code in the sample repository.

3. Save the pipeline and queue a build. When the **Build #nnnnnnnn.n has been queued** message appears, select the number link to see your pipeline in action.

   You now have a working pipeline that's ready for you to customize!

4. When you're ready to make changes to your pipeline, **Edit** it.

5. See the sections below to learn some of the more common ways to customize your pipeline.

::: moniker-end

## Build environment

::: moniker range=">=azure-devops-2020"

Use Azure Pipelines to build your .NET Core projects on Windows, Linux, or macOS without needing to set up any infrastructure of your own. 
The [Microsoft-hosted agents](../agents/hosted.md) in Azure Pipelines include several released versions of the .NET Core SDKs preinstalled.

Ubuntu 18.04 is set here in the YAML file.  

```yaml
pool:
  vmImage: 'ubuntu-18.04' # examples of other options: 'macOS-10.15', 'windows-2019'
```

See [Microsoft-hosted agents](../agents/hosted.md) for a complete list of images and [Pool](../yaml-schema.md#pool) for further examples.

The Microsoft-hosted agents don't include some of the older versions of the .NET Core SDK. 
They also don't typically include prerelease versions. If you need these kinds of SDKs on Microsoft-hosted agents, add the [UseDotNet@2](../tasks/tool/dotnet-core-tool-installer.md) task to your YAML file.

To install the preview version of the 5.0.x SDK for building and 3.0.x for running tests that target .NET Core 3.0.x, add this snippet:

```yaml
steps:
- task: UseDotNet@2
  inputs:
    version: '5.0.x'
    includePreviewVersions: true # Required for preview versions

- task: UseDotNet@2
  inputs:
    version: '3.0.x'
    packageType: runtime
```

Windows agents already include a .NET Core runtime. To install a newer SDK, set `performMultiLevelLookup` to `true` in this snippet: 

```yaml
steps:
- task: UseDotNet@2
  displayName: 'Install .NET Core SDK'
  inputs:
    version: 5.0.x
    performMultiLevelLookup: true
    includePreviewVersions: true # Required for preview versions
```

> [!TIP]
>
> As an alternative, you can set up a [self-hosted agent](../agents/agents.md#install) and save the cost of running the tool installer. See [Linux](../agents/v2-linux.md), [MacOS](../agents/v2-osx.md), or [Windows](../agents/v2-windows.md). 
> You can also use self-hosted agents to save additional time if you have a large repository or you run incremental builds. A self-hosted agent can also help you in using the preview or private SDKs that are not officially supported by Azure DevOps or you have available on your corporate or on-premises environments only. 

::: moniker-end

::: moniker range="< azure-devops"

You can build your .NET Core projects by using the .NET Core SDK and runtime on Windows, Linux, or macOS. 
Your builds run on a [self-hosted agent](../agents/agents.md#install). 
Make sure that you have the necessary version of the .NET Core SDK and runtime installed on the agent.

::: moniker-end

## Restore dependencies

NuGet is a popular way to depend on code that you don't build. You can download NuGet packages and project-specific tools that are specified in the project file by running 
the `dotnet restore` command either through the 
[.NET Core](../tasks/build/dotnet-core-cli.md) task or directly in a script in your pipeline.

::: moniker range=">= tfs-2018"

You can download NuGet packages from Azure Artifacts, NuGet.org, or some other external or internal NuGet repository.
The **.NET Core** task is especially useful to restore packages from authenticated NuGet feeds.

This pipeline uses an artifact feed for `dotnet restore` in the [.NET Core CLI task](../tasks/build/dotnet-core-cli.md). 

```yaml
trigger:
- master

pool:
  vmImage: 'windows-latest'

variables:
  buildConfiguration: 'Release'

steps:
- task: DotNetCoreCLI@2
  inputs:
    command: 'restore'
    feedsToUse: 'select'
    vstsFeed: 'my-vsts-feed' # A series of numbers and letters

- task: DotNetCoreCLI@2
  inputs:
    command: 'build'
    arguments: '--configuration $(buildConfiguration)'
  displayName: 'dotnet build $(buildConfiguration)'
```

::: moniker-end

::: moniker range="< tfs-2018"

You can download NuGet packages from NuGet.org.

::: moniker-end

`dotnet restore` internally uses a version of `NuGet.exe` that is packaged with the .NET Core SDK. `dotnet restore` can only restore packages specified in the .NET Core project `.csproj` files. 
If you also have a Microsoft .NET Framework project in your solution or use `package.json` to specify your dependencies, you must also use the **NuGet** task to restore those dependencies.

::: moniker range="< tfs-2018"

In .NET Core SDK version 2.0 and newer, packages are restored automatically when running other commands such as `dotnet build`.

::: moniker-end

::: moniker range=">= tfs-2018"

In .NET Core SDK version 2.0 and newer, packages are restored automatically when running other commands such as `dotnet build`.
However, you might still need to use the **.NET Core** task to restore packages if you use an authenticated feed.

::: moniker-end

::: moniker range=">= tfs-2018"

If your builds occasionally fail when restoring packages from NuGet.org due to connection issues, 
you can use Azure Artifacts with [upstream sources](../../artifacts/concepts/upstream-sources.md) 
and cache the packages. The credentials of the pipeline are automatically used when connecting 
to Azure Artifacts. These credentials are typically derived from the **Project Collection Build Service** 
account.

If you want to specify a NuGet repository, put the URLs in a `NuGet.config` file in your repository. 
If your feed is authenticated, manage its credentials by creating a NuGet service connection in the **Services** tab under **Project Settings**.

::: moniker-end

::: moniker range=">=azure-devops-2020"

If you use Microsoft-hosted agents, you get a new machine every time your run a build, which means restoring the packages every time. 
This restoration can take a significant amount of time. To mitigate this issue, you can either use Azure Artifacts or a self-hosted agent, in which case, 
you get the benefit of using the package cache.

::: moniker-end

::: moniker range=">=azure-devops-2020"

To restore packages from an external custom feed, use the **.NET Core** task:

```yaml
# do this before your build tasks
steps:
- task: DotNetCoreCLI@2
  displayName: Restore
  inputs:
    command: restore
    projects: '**/*.csproj'
    feedsToUse: config
    nugetConfigPath: NuGet.config    # Relative to root of the repository
    externalFeedCredentials: <Name of the NuGet service connection>
# ...
```

For more information about NuGet service connections, see [publish to NuGet feeds](../artifacts/nuget.md).

::: moniker-end

::: moniker range="< azure-devops"

1. Select **Tasks** in the pipeline. Select the job that runs your build tasks. Then select **+** to add a new task to that job.

1. In the task catalog, find and add the **.NET Core** task.

1. Select the task and, for **Command**, select **restore**.

1. Specify any other options you need for this task. Then save the build.

> [!NOTE]
>
> Make sure the custom feed is specified in your `NuGet.config` file and that credentials are specified in the NuGet service connection.

::: moniker-end

## Build your project

You build your .NET Core project either by running the `dotnet build` command in your pipeline or by using the .NET Core task.

::: moniker range=">=azure-devops-2020"

To build your project by using the .NET Core task, add the following snippet to your `azure-pipelines.yml` file:

```yaml
steps:
- task: DotNetCoreCLI@2
  displayName: Build
  inputs:
    command: build
    projects: '**/*.csproj'
    arguments: '--configuration $(buildConfiguration)' # Update this to match your need
```

You can run any custom dotnet command in your pipeline. The following example shows how to install and use a .NET global tool, [dotnetsay](https://www.nuget.org/packages/dotnetsay/):

```yaml
steps:
- task: DotNetCoreCLI@2
  displayName: 'Install dotnetsay'
  inputs:
    command: custom
    custom: tool
    arguments: 'install -g dotnetsay'
```

::: moniker-end

::: moniker range="< azure-devops"

### Build

1. Select **Tasks** in the pipeline. Select the job that runs your build tasks. Then select **+** to add a new task to that job.

1. In the task catalog, find and add the **.NET Core** task.

1. Select the task and, for **Command**, select **build** or **publish**.

1. Specify any other options you need for this task. Then save the build.

### Install a tool

To install a .NET Core global tool like [dotnetsay](https://www.nuget.org/packages/dotnetsay/) in your build running on Windows, take the following steps:

1. Add the **.NET Core** task and set the following properties:
   * **Command**: custom.
     * **Path to projects**: _leave empty_.
   * **Custom command**: tool.
   * **Arguments**: `install -g dotnetsay`.

2. Add a **Command Line** task and set the following properties:
   * **Script:** `dotnetsay`.

::: moniker-end

## Run your tests

If you have test projects in your repository, then use the **.NET Core** task to run unit tests by using testing frameworks like MSTest, xUnit, and NUnit. For this functionality, the test project must reference [Microsoft.NET.Test.SDK](https://www.nuget.org/packages/Microsoft.NET.Test.SDK) version 15.8.0 or higher.
Test results are automatically published to the service. These results are then made available to you in the build summary and can be used for troubleshooting failed tests and test-timing analysis.

::: moniker range=">=azure-devops-2020"

Add the following snippet to your `azure-pipelines.yml` file:

```yaml
steps:
# ...
# do this after other tasks such as building
- task: DotNetCoreCLI@2
  inputs:
    command: test
    projects: '**/*Tests/*.csproj'
    arguments: '--configuration $(buildConfiguration)'
```

An alternative is to run the `dotnet test` command with a specific logger and then use the **Publish Test Results** task:

```yaml
steps:
# ...
# do this after your tests have run
- script: dotnet test <test-project> --logger trx
- task: PublishTestResults@2
  condition: succeededOrFailed()
  inputs:
    testRunner: VSTest
    testResultsFiles: '**/*.trx'
```

::: moniker-end

::: moniker range="< azure-devops"

Use the **.NET Core** task with **Command** set to **test**. 
**Path to projects** should refer to the test projects in your solution.

::: moniker-end


## Collect code coverage 

If you're building on the Windows platform, code coverage metrics can be collected by using the built-in coverage data collector. For this functionality, the test project must reference [Microsoft.NET.Test.SDK](https://www.nuget.org/packages/Microsoft.NET.Test.SDK) version 15.8.0 or higher. 
If you use the **.NET Core** task to run tests, coverage data is automatically published to the server. The **.coverage** file can be downloaded from the build summary for viewing in Visual Studio.

::: moniker range=">=azure-devops-2020"

Add the following snippet to your `azure-pipelines.yml` file:

```yaml
steps:
# ...
# do this after other tasks such as building
- task: DotNetCoreCLI@2
  inputs:
    command: test
    projects: '**/*Tests/*.csproj'
    arguments: '--configuration $(buildConfiguration) --collect "Code coverage"'
```

If you choose to run the `dotnet test` command, specify the test results logger and coverage options. Then use the [Publish Test Results](../tasks/test/publish-test-results.md) task:

```yaml
steps:
# ...
# do this after your tests have run
- script: dotnet test <test-project> --logger trx --collect "Code coverage"
- task: PublishTestResults@2
  inputs:
    testRunner: VSTest
    testResultsFiles: '**/*.trx'
```

::: moniker-end

::: moniker range="< azure-devops"

1. Add the .NET Core task to your build job and set the following properties:

   * **Command**: test.
   * **Path to projects**: _Should refer to the test projects in your solution_.
   * **Arguments**: `--configuration $(BuildConfiguration) --collect "Code coverage"`.

2. Ensure that the **Publish test results** option remains selected.

::: moniker-end


### Collect code coverage metrics with Coverlet
If you're building on Linux or macOS, you can use [Coverlet](https://github.com/tonerdo/coverlet) or a similar tool to collect code coverage metrics.

Code coverage results can be published to the server by using the [Publish Code Coverage Results](../tasks/test/publish-code-coverage-results.md) task. To use this functionality, the coverage tool must be configured to generate results in Cobertura or JaCoCo coverage format.

To run tests and publish code coverage with Coverlet:
* Add a reference to the `coverlet.msbuild` NuGet package in your test project(s) for .NET projects below .NET 5. For .NET 5, add a reference to the  `coverlet.collector` NuGet package.
* Add this snippet to your `azure-pipelines.yml` file:


# [.NET 5](#tab/dotnetfive)

  ```yaml
  - task: UseDotNet@2
    inputs:
      version: '5.0.x'
      includePreviewVersions: true # Required for preview versions
    
  - task: DotNetCoreCLI@2
    displayName: 'dotnet build'
    inputs:
      command: 'build'
      configuration: $(buildConfiguration)
    
  - task: DotNetCoreCLI@2
    displayName: 'dotnet test'
    inputs:
      command: 'test'
      arguments: '--configuration $(buildConfiguration) --collect:"XPlat Code Coverage" -- DataCollectionRunSettings.DataCollectors.DataCollector.Configuration.Format=cobertura'
      publishTestResults: true
      projects: 'MyTestLibrary' # update with your test project directory
    
  - task: PublishCodeCoverageResults@1
    displayName: 'Publish code coverage report'
    inputs:
      codeCoverageTool: 'Cobertura'
      summaryFileLocation: '$(Build.SourcesDirectory)/**/coverage.cobertura.xml'
  ```

# [.NET < 5](#tab/netearlierversions)

  ```yaml
  - task: DotNetCoreCLI@2
    displayName: 'dotnet test'
    inputs:
      command: 'test'
      arguments: '--configuration $(buildConfiguration) --collect:"XPlat Code Coverage" -- DataCollectionRunSettings.DataCollectors.DataCollector.Configuration.Format=cobertura'
      publishTestResults: true
      projects: '**/test-library/*.csproj' # update with your test project directory

  - task: PublishCodeCoverageResults@1
    displayName: 'Publish code coverage report'
    inputs:
      codeCoverageTool: 'Cobertura'
      summaryFileLocation: '$(Build.SourcesDirectory)/**/coverage.cobertura.xml'
  ```

---

 
## Package and deliver your code

After you've built and tested your app, you can upload the build output to Azure Pipelines or TFS, create and publish a NuGet package, 
or package the build output into a .zip file to be deployed to a web application.

::: moniker range=">=azure-devops-2020"

### Publish artifacts to Azure Pipelines

To publish the output of your .NET **build**, 
* Run `dotnet publish --output $(Build.ArtifactStagingDirectory)` on CLI or add the DotNetCoreCLI@2 task with publish command.
* Publish the artifact by using Publish artifact task.

Add the following snippet to your `azure-pipelines.yml` file:

```yaml
steps:

- task: DotNetCoreCLI@2
  inputs:
    command: publish
    publishWebProjects: True
    arguments: '--configuration $(BuildConfiguration) --output $(Build.ArtifactStagingDirectory)'
    zipAfterPublish: True

# this code takes all the files in $(Build.ArtifactStagingDirectory) and uploads them as an artifact of your build.
- task: PublishPipelineArtifact@1
  inputs:
    targetPath: '$(Build.ArtifactStagingDirectory)' 
    artifactName: 'myWebsiteName'

```

> [!NOTE]
> The `dotNetCoreCLI@2` task has a `publishWebProjects` input that is set to **true** by default. This publishes _all_ web projects in your repo by default. You can find more help and information in the [open source task on GitHub](https://github.com/microsoft/azure-pipelines-tasks).

To copy more files to Build directory before publishing, use [Utility: copy files](../tasks/utility/copy-files.md).

### Publish to a NuGet feed

To create and publish a NuGet package, add the following snippet:

```yaml
steps:
# ...
# do this near the end of your pipeline in most cases
- script: dotnet pack /p:PackageVersion=$(version)  # define version variable elsewhere in your pipeline
- task: NuGetAuthenticate@0
  input:
    nuGetServiceConnections: '<Name of the NuGet service connection>'
- task: NuGetCommand@2
  inputs:
    command: push
    nuGetFeedType: external
    publishFeedCredentials: '<Name of the NuGet service connection>'
    versioningScheme: byEnvVar
    versionEnvVar: version
```

For more information about versioning and publishing NuGet packages, see [publish to NuGet feeds](../artifacts/nuget.md).  

### Deploy a web app

To create a .zip file archive that's ready for publishing to a web app, add the following snippet:

```yaml
steps:
# ...
# do this after you've built your app, near the end of your pipeline in most cases
# for example, you do this before you deploy to an Azure web app on Windows
- task: DotNetCoreCLI@2
  inputs:
    command: publish
    publishWebProjects: True
    arguments: '--configuration $(BuildConfiguration) --output $(Build.ArtifactStagingDirectory)'
    zipAfterPublish: True
```

To publish this archive to a web app, see [Azure Web Apps deployment](../targets/webapp.md).  

::: moniker-end

::: moniker range="< azure-devops"

### Publish artifacts to Azure Pipelines

Use the **Publish Artifacts** task to publish the output of your build to Azure Pipelines or TFS.

### Publish to a NuGet feed

If you want to publish your code to a NuGet feed, take the following steps:

1. Use a .NET Core task with **Command** set to pack.

1. [Publish your package to a NuGet feed](../artifacts/nuget.md).

### Deploy a web app

1. Use a .NET Core task with **Command** set to publish.

1. Make sure you've selected the option to create a .zip file archive.

1. To publish this archive to a web app, see [Azure Web Apps deployment](../targets/webapp.md).

::: moniker-end

## Build an image and push to container registry

For your app, you can also [build an image](containers/build-image.md) and [push it to a container registry](containers/push-image.md).

<a name="troubleshooting"></a>
## Troubleshooting

If you're able to build your project on your development machine, but you're having trouble building it on Azure Pipelines or TFS, explore the following potential causes and corrective actions:

::: moniker range=">=azure-devops-2020"
* We don't install prerelease versions of the .NET Core SDK on Microsoft-hosted agents. After a new version of the .NET Core SDK is released, 
it can take a few weeks for us to roll it out to all the data centers that Azure Pipelines run on. You don't have to wait for us to finish 
this rollout. You can use the **.NET Core Tool Installer**, as explained in this guidance, to install the desired version of the .NET Core SDK 
on Microsoft-hosted agents.  

::: moniker-end

* Check that the versions of the .NET Core SDK and runtime on your development machine match those on the agent. 
You can include a command-line script `dotnet --version` in your pipeline to print the version of the .NET Core SDK. 
Either use the **.NET Core Tool Installer**, as explained in this guidance, to deploy the same version on the agent, 
or update your projects and development machine to the newer version of the .NET Core SDK.

* You might be using some logic in the Visual Studio IDE that isn't encoded in your pipeline. 
Azure Pipelines or TFS runs each of the commands you specify in the tasks one after the other in a new process. 
Look at the logs from the Azure Pipelines or TFS build to see the exact commands that ran as part of the build. 
Repeat the same commands in the same order on your development machine to locate the problem.

* If you have a mixed solution that includes some .NET Core projects and some .NET Framework projects, 
  you should also use the **NuGet** task to restore packages specified in `packages.config` files.
Similarly, you should add **MSBuild** or **Visual Studio Build** tasks to build the .NET Framework projects.

* If your builds fail intermittently while restoring packages, either NuGet.org is having issues, or there are 
networking problems between the Azure datacenter and NuGet.org. These aren't under our control, and you might 
need to explore whether using Azure Artifacts with NuGet.org as an upstream source improves the reliability of your builds.

* Occasionally, when we roll out an update to the hosted images with a new version of the .NET Core SDK or Visual Studio, 
something might break your build. This can happen, for example, if a newer version or feature of the NuGet tool 
is shipped with the SDK. To isolate these problems, use the **.NET Core Tool Installer** task to specify the version 
of the .NET Core SDK that's used in your build.

## FAQ

### Where can I learn more about Azure Artifacts and the TFS Package Management service?

[Package Management in Azure Artifacts and TFS](../../artifacts/index.yml)

### Where can I learn more about .NET Core commands?

[.NET Core CLI tools](/dotnet/core/tools/)

### Where can I learn more about running tests in my solution?

[Unit testing in .NET Core projects](/dotnet/core/testing/)

### Where can I learn more about tasks?

[Build and release tasks](../tasks/index.md)