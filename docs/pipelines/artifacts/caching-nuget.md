---
title: Caching NuGet packages
description: Cache NuGet packages in Azure Pipelines
ms.topic: conceptual
ms.author: rabououn
author: ramiMSFT
ms.date: 11/30/2021
monikerRange: '>= tfs-2017'
"recommendations": "true"
---

# How to cache NuGet packages to reduce the build time

[!INCLUDE [version-gt-eq-2017](../../includes/version-gt-eq-2017.md)]

With pipeline caching, you can reduce your build time by caching your dependencies to be used in later runs. In this article, we will walk through the steps to cache and restore NuGet packages.

## Lock dependencies

To set up the cache task, we must first lock our project' dependencies and create a **package.lock.json** file. We will use the hash of the content of this file to generate a unique key for our cache. 

To lock your project's dependencies, set the **RestorePackagesWithLockFile** property in your `csproj` file to true. NuGet restore will generate a lock file **packages.lock.json** at your project's root directory.

```XML
<PropertyGroup>
  <RestorePackagesWithLockFile>true</RestorePackagesWithLockFile>
</PropertyGroup>
```

 Make sure to check in the **packages.lock.json** file to your source code repository.

## Cache NuGet packages

We will need to create a pipeline variable to point to the location of our packages on the agent running the pipeline. 

In this example, the content of the **packages.lock.json** will be hashed to produce a dynamic cache key. This will ensure that every time the file is modified, a new cache key will be generated.

:::image type="content" source="media/cache-key-hash.png" alt-text="Screenshot showing how the cache key is generated in Azure Pipelines.":::

```YAML
variables:
  NUGET_PACKAGES: ''

- task: Cache@2
  displayName: Cache
  inputs:
    key: 'nuget | "$(Agent.OS)" | **/packages.lock.json'
    path: '$(NUGET_PACKAGES)'
    restoreKeys: |
      nuget | "$(Agent.OS)"
      nuget
    cacheHitVar: 'CACHE_RESTORED'
```

## Restore cache

This task will only run if the `CACHE_RESTORED` variable is false.

```YAML
- task: NuGetCommand@2
  condition: ne(variables.CACHE_RESTORED, true)
  inputs:
    command: 'restore'
    restoreSolution: '**/*.sln'
```

## Performance comparison

Pipeline caching is a great way to speed up your pipeline execution. Here is a side-by-side performance comparison for two different pipelines. Before adding the caching task (right), the restore task took approximately 41 seconds. We added the caching task to a second pipeline (left) and configured the restore task to run when a cache miss is encountered. The restore task in this case took 8 seconds to complete. 

:::image type="content" source="media/caching-performance.png" alt-text="Screenshot showing pipeline performance with and without caching.":::

Below is the YAML pipeline for reference:

```YAML
pool:
  vmImage: 'windows-latest'

variables:
  solution: '**/*.sln'
  buildPlatform: 'Any CPU'
  buildConfiguration: 'Release'
  NUGET_PACKAGES: ''

steps:
- task: NuGetToolInstaller@1
  displayName: 'NuGet tool installer'

- task: Cache@2
  displayName: 'NuGet Cache'
  inputs:
    key: 'nuget | "$(Agent.OS)" | **/packages.lock.json'
    path: '$(NUGET_PACKAGES)'
    cacheHitVar: 'CACHE_RESTORED'

- task: NuGetCommand@2
  displayName: 'NuGet restore'
  condition: ne(variables.CACHE_RESTORED, true)
  inputs:
    command: 'restore'
    restoreSolution: '$(solution)'

- task: VSBuild@1
  inputs:
    solution: '$(solution)'
    platform: '$(buildPlatform)'
    configuration: '$(buildConfiguration)'

- task: VSTest@2
  inputs:
    platform: '$(buildPlatform)'
    configuration: '$(buildConfiguration)'
```

## Related articles

- [Pipeline caching](../release/caching.md)
- [Deploy from multiple branches](../release/deploy-multiple-branches.md)
- [Deploy pull request Artifacts](../release/deploy-pull-request-builds.md)