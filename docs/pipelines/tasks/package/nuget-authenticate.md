---
title: NuGet Authenticate
ms.custom: seodec18
description: Configure NuGet tools to authenticate with Azure Artifacts and other NuGet repositories
ms.topic: reference
ms.date: 01/13/2021
monikerRange: 'azure-devops'
---

# Package: NuGet Authenticate

[!INCLUDE [version-eq-azure-devops](../../../includes/version-eq-azure-devops.md)]

Configure NuGet tools to authenticate with Azure Artifacts and other NuGet repositories.

> [!IMPORTANT]
> This task is only compatible with NuGet >= 4.8.0.5385, dotnet >= 2.1.400, or MSBuild >= 15.8.166.59604

## YAML snippet

[!INCLUDE [temp](../includes/yaml/NuGetAuthenticateV0.md)]

## Arguments


| Argument                                                                                           | Description                                                         |
| -------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------- |
| `nuGetServiceConnections`<br/>Service connection credentials for feeds outside this organization   | (Optional) Comma-separated list of [NuGet service connection](~/pipelines/library/service-endpoints.md#nuget-service-connection) names for feeds outside this organization/collection to additionally set up. If you only need feeds in this organization/collection, leave this blank; the build’s credentials are used automatically. |
| `forceReinstallCredentialProvider`<br/>Reinstall the credential provider even if already installed | (Optional) Reinstall the credential provider to the user profile directory even if already installed. This may upgrade (or potentially downgrade) the credential provider. |
| [!INCLUDE [temp](../includes/control-options-arguments-md.md)] | |

## Examples

### Restore and push NuGet packages within your organization

If all of the Azure Artifacts feeds you use are in the same organization as your pipeline, you can use the NuGetAuthenticate task without specifying any inputs. For project scoped feeds that are in a different project than where the pipeline is running in, you must manually give the project and the feed access to the pipeline's project's build service.

#### nuget.config
```XML
<configuration>
  <packageSources>
    <!-- 
      Any Azure Artifacts feeds within your organization will automatically be authenticated. Both dev.azure.com and visualstudio.com domains are supported.
      Project scoped feed URL includes the project, organization scoped feed URL does not.
    -->
    <add key="MyProjectFeed1" value="https://pkgs.dev.azure.com/{organization}/{project}/_packaging/{feed}/nuget/v3/index.json" />
    <add key="MyProjectFeed2" value="https://{organization}.pkgs.visualstudio.com/{project}/_packaging/{feed}/nuget/v3/index.json" />
    <add key="MyOtherProjectFeed1" value="https://pkgs.dev.azure.com/{organization}/{project}/_packaging/{feed@view}/nuget/v3/index.json" />
    <add key="MyOrganizationFeed1" value="https://pkgs.dev.azure.com/{organization}/_packaging/{feed}/nuget/v3/index.json" />
  </packageSources>
</configuration>
```

To use a service connection, specify the service connection in the `nuGetServiceConnections` input for the NuGet Authenticate task. You can then reference the service connection with `-ApiKey AzureArtifacts` in a task. 
#### nuget.exe
```YAML
- task: NuGetAuthenticate@0
  inputs:
    nuGetServiceConnections: OtherOrganizationFeedConnection, ThirdPartyRepositoryConnection
- task: NuGetToolInstaller@1 # Optional if nuget.exe >= 4.8.5385 is already on the path
  inputs:
    versionSpec: '*'
    checkLatest: true
- script: nuget restore
# ...
- script: nuget push -ApiKey AzureArtifacts -Source "MyProjectFeed1" MyProject.*.nupkg
```

#### dotnet
```YAML
- task: NuGetAuthenticate@0
  inputs:
    nuGetServiceConnections: OtherOrganizationFeedConnection, ThirdPartyRepositoryConnection
- task: UseDotNet@2 # Optional if the .NET Core SDK is already installed
- script: dotnet restore
# ...
- script: dotnet nuget push --api-key AzureArtifacts --source https://pkgs.dev.azure.com/{organization}/_packaging/{feed1}/nuget/v3/index.json MyProject.*.nupkg
```
In the above examples `OtherOrganizationFeedConnection` and `ThirdPartyRepositoryConnection` are the names of [NuGet service connections](~/pipelines/library/service-endpoints.md#nuget-service-connection) that have been configured and authorized for use in your pipeline, and have URLs that match those in your nuget.config or command line argument.

The package source URL pointing to an Azure Artifacts feed may or may not contain the project. An URL for a project scoped feed must contain the project, and a URL for a organization scoped feed must not contain the project. [Learn more](../../../artifacts/feeds/project-scoped-feeds.md).

### Restore and push NuGet packages outside your organization

If you use Azure Artifacts feeds from a different organization or use a third-party authenticated package repository, you'll need to set up [NuGet service connections](~/pipelines/library/service-endpoints.md#nuget-service-connection) and specify them in the `nuGetServiceConnections` input.
Feeds within your Azure Artifacts organization will also be automatically authenticated.

#### nuget.config
```XML
<configuration>
  <packageSources>
    <!-- Any Azure Artifacts feeds within your organization will automatically be authenticated -->
    <add key="MyProjectFeed1" value="https://pkgs.dev.azure.com/{organization}/{project}/_packaging/{feed}/nuget/v3/index.json" />
    <add key="MyOrganizationFeed" value="https://pkgs.dev.azure.com/{organization}/_packaging/{feed}/nuget/v3/index.json" />
    <!-- Any package source listed here whose URL matches the URL of a service connection in nuGetServiceConnections will also be authenticated.
         The key name here does not need to match the name of the service connection. -->
    <add key="OtherOrganizationFeed" value="https://pkgs.dev.azure.com/{otherorganization}/_packaging/{feed}/nuget/v3/index.json" />
    <add key="ThirdPartyRepository" value="https://{thirdPartyRepository}/index.json" />
  </packageSources>
</configuration>
```

#### nuget.exe
```YAML
- task: NuGetAuthenticate@0
  inputs:
    nuGetServiceConnections: OtherOrganizationFeedConnection, ThirdPartyRepositoryConnection
- task: NuGetToolInstaller@1 # Optional if nuget.exe >= 4.8.5385 is already on the path
  inputs:
    versionSpec: '*'
    checkLatest: true
- script: nuget restore
# ...
- script: nuget push -ApiKey AzureArtifacts -Source "MyProjectFeed1" MyProject.*.nupkg
```

#### dotnet
```YAML
- task: NuGetAuthenticate@0
  inputs:
    nuGetServiceConnections: OtherOrganizationFeedConnection, ThirdPartyRepositoryConnection
- task: UseDotNet@2 # Optional if the .NET Core SDK is already installed
- script: dotnet restore
# ...
- script: dotnet nuget push --api-key AzureArtifacts --source "MyProjectFeed1"  MyProject.*.nupkg
```
`OtherOrganizationFeedConnection` and `ThirdPartyRepositoryConnection` are the names of [NuGet service connections](~/pipelines/library/service-endpoints.md#nuget-service-connection) that have been configured and authorized for use in your pipeline, and have URLs that match those in your nuget.config or command line argument.

The package source URL pointing to an Azure Artifacts feed may or may not contain the project. An URL for a project scoped feed must contain the project, and a URL for a organization scoped feed must not contain the project. [Learn more](../../../artifacts/feeds/project-scoped-feeds.md).

## Open source

This task is open source [on GitHub](https://github.com/Microsoft/azure-pipelines-tasks). Feedback and contributions are welcome.

## FAQ

<!-- BEGINSECTION class="md-qanda" -->

### What tools are compatible with this task?

This task will configure tools that support [NuGet cross platform plugins](/nuget/reference/extensibility/nuget-cross-platform-plugins). Today, that includes nuget.exe, dotnet, and recent versions of MSBuild with built-in support for restoring NuGet packages.

Specifically, this task will configure:
* nuget.exe, version 4.8.5385 or higher
* dotnet / .NET Core SDK, version 2.1.400 or higher
* MSBuild, version 15.8.166.59604 or higher

However, upgrading to the latest stable version is recommended if you encounter any issues.  

### I get "A task was canceled" errors during a package restore. What should I do?

Known issues in NuGet and in the Azure Artifacts Credential Provider can cause this type of error and updating to the latest nuget may help.  

A [known issue](https://github.com/NuGet/Home/issues/8198) in some versions of nuget/dotnet can cause this error, especially during large restores on resource constrained machines. This issue is fixed in [NuGet 5.2](/nuget/release-notes/nuget-5.2-rtm), as well as .NET Core SDK 2.1.80X and 2.2.40X. If you are using an older version, try upgrading your version of NuGet or dotnet. The [.NET Core Tool Installer](~/pipelines/tasks/tool/dotnet-core-tool-installer.md) task can be used to install a newer version of the .NET Core SDK.  

There are also known issues with the Azure Artifacts Credential Provider (installed by this task), including [artifacts-credprovider/#77](https://github.com/microsoft/artifacts-credprovider/issues/77) and [artifacts-credprovider/#108](https://github.com/microsoft/artifacts-credprovider/issues/108). If you experience these issues, ensure you have the latest credential provider by setting the input `forceReinstallCredentialProvider` to `true` in the NuGet Authenticate task. This will also ensure your credential provider is automatically updated as issues are resolved.  

If neither of the above resolves the issue, please enable [Plugin Diagnostic Logging](https://github.com/NuGet/Home/wiki/Plugin-Diagnostic-Logging) and report the issue to [NuGet](https://github.com/NuGet/Home/issues) and the [Azure Artifacts Credential Provider](https://github.com/microsoft/artifacts-credprovider/issues).  

### How is this task different than the NuGetCommand and DotNetCoreCLI tasks?

This task configures nuget.exe, dotnet, and MSBuild to authenticate with Azure Artifacts or other repositories that require authentication.
After this task runs, you can then invoke the tools in a later step (either directly or via a script) to restore or push packages.

The NuGetCommand and DotNetCoreCLI tasks require using the task to restore or push packages, as authentication to Azure Artifacts is only configured within the lifetime of the task. This can prevent you from restoring or pushing packages within your own script. It may also prevent you from passing specific command line arguments to the tool.

The NuGetAuthenticate task is the recommended way to use authenticated feeds within a pipeline.

### When in my pipeline should I run this task? 

This task must run before you use a NuGet tool to restore or push packages to an authenticated package source such as Azure Artifacts. There are no other ordering requirements.
For example, this task can safely run either before or after a NuGet or .NET Core tool installer task.

### How do I configure a NuGet package source that uses ApiKey ("NuGet API keys"), such as nuget.org?

Some package sources such as nuget.org use API keys for authentication when pushing packages, rather than username/password credentials. Due to limitations in NuGet, this task cannot be used to set up a NuGet service connection that uses an API key.  

Instead:
1. Configure a [secret variable](~/pipelines/process/variables.md#secret-variables) containing the ApiKey
2. Perform the package push using `nuget push -ApiKey $(myNuGetApiKey)` or `dotnet nuget push --api-key $(myNuGetApiKey)`, assuming you named the variable `myNuGetApiKey`

### My agent is behind a web proxy. Will NuGetAuthenticate set up nuget.exe, dotnet, and MSBuild to use my proxy?

No. While this task itself will work behind a web proxy [your agent has been configured to use](~/pipelines/agents/proxy.md), it does not configure NuGet tools to use the proxy.

To do so, you can either:
* Set the environment variable `http_proxy` and optionally `no_proxy` to your proxy settings. See [NuGet CLI environment variables](/nuget/reference/cli-reference/cli-ref-environment-variables) for details. Please understand that these are commonly used variables which other non-NuGet tools (e.g. curl) may also use.
  >**Caution:**  
  >The `http_proxy` and `no_proxy` variables are case-sensitive on Linux and Mac operating systems and must be lowercase. Attempting to use an Azure Pipelines variable to set the environment variable will not work, as it will be converted to uppercase. Instead, set the environment variables on the self-hosted agent's machine and restart the agent.

* Add the proxy settings to the [user-level nuget.config](/nuget/consume-packages/configuring-nuget-behavior) file, either manually or using `nuget config -set` as described in the [nuget.config reference](/nuget/reference/nuget-config-file#config-section) documentation.
  >**Caution:**  
  >The proxy settings (such as `http_proxy`) must be added to the user-level config. They will be ignored if specified in a different nuget.config file.

### How do I debug if I have issues with this task?

To get verbose logs from the pipeline, add a pipeline variable system.debug to true.

### How does this task work?

This task installs the [Azure Artifacts Credential Provider](https://github.com/Microsoft/artifacts-credprovider) into the NuGet plugins directory if it is not already installed.  
It then sets environment variables such as `VSS_NUGET_URI_PREFIXES`, `VSS_NUGET_ACCESSTOKEN`, and `VSS_NUGET_EXTERNAL_FEED_ENDPOINTS` to configure the credential provider. These variables remain set for the lifetime of the job.  
When restoring or pushing packages, a NuGet tool executes the credential provider, which uses the above variables to determine if it should return credentials back to the tool.

See the credential provider documentation for more details.

### My Pipeline needs to access a feed in a different project

If the pipeline is running in a different project than the project hosting the feed, you must set up the other project to grant read/write access to the build service. See [Package permissions in Azure Pipelines](../../../artifacts/feeds/feed-permissions.md#pipelines-permissions) for more details.

<!-- ENDSECTION -->
