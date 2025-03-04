---
title: Set up your client's npmrc
description: How to set up your project and authenticate to Azure Artifacts feeds
ms.assetid: A5364E3A-3918-4318-AAE0-430EA91AD5F1
ms.technology: devops-artifacts
ms.topic: conceptual
ms.date: 01/21/2022
monikerRange: '>= tfs-2017'
---

# Set up your project and connect to Azure Artifacts

[!INCLUDE [version-gt-eq-2017](../../includes/version-gt-eq-2017.md)]

With Azure Artifacts, you can publish different types of packages to your feeds such as npm, NuGet, Python, Maven, and Universal packages. You can also install packages from feeds and public registries such as npmjs.com.

To authenticate with Azure Artifacts, we must first set up our config file. Npm uses [.npmrc configuration files](https://docs.npmjs.com/files/npmrc) to store feed URLs and credentials.

## Project setup

We recommend using two **.npmrc_** files, the first one we will use to authenticate to Azure Artifacts, and the second one should be kept locally to store our credentials. This enables you to share your project's **.npmrc** while keeping your credentials secure.

::: moniker range=">= azure-devops-2019"

1. Select **Artifacts**, and then select **Connect to feed**.
 
    :::image type="content" source="../media/connect-to-feed-azure-devops-newnav.png" alt-text="Screenshot showing how to connect to a feed.":::

::: moniker-end

::: moniker range=">= tfs-2017 < azure-devops-2019"

1. Select **Packages**, and then select **Connect to feed**.

    :::image type="content" source="../media/connect-to-feed.png" alt-text="Screenshot showing how to connect to a feed in TFS.":::

::: moniker-end

2. Select **npm** from the list of package types.

3. If this is the first time using Azure Artifacts with npm, select **Get the tools** button and follow the instructions to install the prerequisites. 

::: moniker range=">= azure-devops"   

4. Follow the instructions under the **Project setup** section to set up your project.

    :::image type="content" source="../media/npm-azure-devops-newnav.png" alt-text="Screenshot showing the steps to set up the project and publish and restore packages.":::

::: moniker-end

::: moniker range="> tfs-2018 < azure-devops"

4. Follow the instructions in the **Project setup** section to set up your project.

   :::image type="content" source="../media/connect-to-feed-devops-server.png" alt-text="Screenshot showing the steps to set up the project and restore packages.":::

::: moniker-end

::: moniker range=">= tfs-2017 < azure-devops-2019"

4. Follow the instructions to set up your project.

    :::image type="content" source="../media/connect-to-feed-npm-registry.png" alt-text="Screenshot showing the steps to set up the project in TFS.":::

::: moniker-end
        
On your development machine, place the second *.npmrc* file in your *$HOME* for Linux/Mac or *$env.HOME* for Windows. This second file should contain all your registries' credentials.

## Credentials setup

> [!TIP]
> Multiple registries in .npmrc files are supported with [upstream sources](../concepts/upstream-sources.md) and [scopes](..//npm/scopes.md).

### [Windows](#tab/windows/)

If you are developing on Windows, we recommend that you use `vsts-npm-auth` to fetch the credentials and inject them into your *%USERPROFILE%\\.npmrc*.  The easiest way to set this up is to install `vsts-npm-auth` globally and then add a run script to your *package.json*.

- Install vsts-npm-auth globally:

    ```Command
    npm install -g vsts-npm-auth
    ```

- Add script to package.json:

    ```json
    "scripts": {
        "refreshVSToken" : "vsts-npm-auth -config .npmrc"
    }
    ```

### [Linux/Mac](#tab/linux/)

`vsts-npm-auth` is not supported in Linux/Mac. Follow the steps below to set up your credentials:

1. Copy the following snippet into your .npmrc file.

    - **Organization-scoped feed**:

        ```Command
        ; begin auth token
        //pkgs.dev.azure.com/<ORGANIZATION_NAME>/_packaging/<FEED_NAME>/npm/registry/:username=[ENTER_ANY_VALUE_BUT_NOT_AN_EMPTY_STRING]
        //pkgs.dev.azure.com/<ORGANIZATION_NAME>/_packaging/<FEED_NAME>/npm/registry/:_password=[BASE64_ENCODED_PERSONAL_ACCESS_TOKEN]
        //pkgs.dev.azure.com/<ORGANIZATION_NAME>/_packaging/<FEED_NAME>/npm/registry/:email=npm requires email to be set but doesn't use the value
        //pkgs.dev.azure.com/<ORGANIZATION_NAME>/_packaging/<FEED_NAME>/npm/:username=[ANY_VALUE_BUT_NOT_AN_EMPTY_STRING]
        //pkgs.dev.azure.com/<ORGANIZATION_NAME>/_packaging/<FEED_NAME>/npm/:_password=[BASE64_ENCODED_PERSONAL_ACCESS_TOKEN]
        //pkgs.dev.azure.com/<ORGANIZATION_NAME>/_packaging/<FEED_NAME>/npm/:email=npm requires email to be set but doesn't use the value
        ; end auth token
        ```
    
    - **Project-scoped feed**:

        ```Command
        ; begin auth token
        //pkgs.dev.azure.com/<ORGANIZATION_NAME>/<PROJECT_NAME>/_packaging/<FEED_NAME>/npm/registry/:username=[ENTER_ANY_VALUE_BUT_NOT_AN_EMPTY_STRING]
        //pkgs.dev.azure.com/<ORGANIZATION_NAME>/<PROJECT_NAME>/_packaging/<FEED_NAME>/npm/registry/:_password=[BASE64_ENCODED_PERSONAL_ACCESS_TOKEN]
        //pkgs.dev.azure.com/<ORGANIZATION_NAME>/<PROJECT_NAME>/_packaging/<FEED_NAME>/npm/registry/:email=npm requires email to be set but doesn't use the value
        //pkgs.dev.azure.com/<ORGANIZATION_NAME>/<PROJECT_NAME>/_packaging/<FEED_NAME>/npm/:username=[ENTER_ANY_VALUE_BUT_NOT_AN_EMPTY_STRING]
        //pkgs.dev.azure.com/<ORGANIZATION_NAME>/<PROJECT_NAME>/_packaging/<FEED_NAME>/npm/:_password=[BASE64_ENCODED_PERSONAL_ACCESS_TOKEN]
        //pkgs.dev.azure.com/<ORGANIZATION_NAME>/<PROJECT_NAME>/_packaging/<FEED_NAME>/npm/:email=npm requires email to be set but doesn't use the value
        ; end auth token
        ```

1. Generate a [personal access token](../../organizations/accounts/use-personal-access-tokens-to-authenticate.md) with **packaging read and write** scopes.

1. Encode your newly generated personal access token as follows:

    1. Run the following command in an elevated command prompt window, and then paste your personal access token when prompted:
        
        ```Command
        node -e "require('readline') .createInterface({input:process.stdin,output:process.stdout,historySize:0}) .question('PAT> ',p => { b64=Buffer.from(p.trim()).toString('base64');console.log(b64);process.exit(); })"
        ```

        You can also use the following command to convert your personal access token to Base64:

        - **LinuxMac**:
            ```Command
            echo -n "YOUR_PERSONAL_ACCESS-TOKEN" | base64
            ```
    1. Copy the Base64 encoded value.

1. Open your .npmrc file and replace the placeholder *[BASE64_ENCODED_PERSONAL_ACCESS_TOKEN]* with your encoded personal access token that you created in the previous step.

* * *

> [!NOTE]
> `vsts-npm-auth` is not supported in TFS and Azure DevOps Server.

## Set up authentication

Azure Artifacts recommend using the `npmAuthenticate` task to set up authentication for your pipeline tasks. When using a task runner such as gulp or Grunt, you'll need to add the **npm Authenticate** task at the beginning of your pipeline. This will inject your credentials into your project's *.npmrc* and persist them for the lifespan of the pipeline run. This allows subsequent steps to use the credentials in the *.npmrc*.

::: moniker range=">= azure-devops-2019"

1. Select **Azure Pipelines**, and then select your pipeline definition.

1. Select **Edit** to modify your pipeline.

1. Select `+` to add a task to your pipeline.

   > [!div class="mx-imgBorder"] 
   > ![Screenshot showing how to add the npm authenticate task to your pipeline](../../pipelines/media/get-started-designer/builds-tab-add-task-azure-devops-newnavon.png)

1. Search for the **npm Authenticate** task, and then select **Add** to add it to your pipeline.

   > [!div class="mx-imgBorder"] 
   > ![Screenshot showing the npm authenticate task added to the pipeline](../media/build-definition/build-definition-npm-auth-task-phase-newnav.png)

1. Select your .npmrc file.

   > [!div class="mx-imgBorder"]
   > ![Screenshot showing how to add your .npmrc file](../media/build-definition/build-definition-npm-auth-task-file.png)

1. Select **Save & queue** when you are done.

::: moniker-end

::: moniker range=">= tfs-2017 < azure-devops-2019"

1. Select **Build and Release**, and then select **Builds**.

   > [!div class="mx-imgBorder"]
   > ![Screenshot showing how to access your builds in TFS](../../pipelines/media/get-started-designer/navigate-to-builds-tab-tfs-2018-2.png)

1. Select your pipeline, and then select **Edit**.

1. Select `+` to add a task to your pipeline.

    > [!div class="mx-imgBorder"]
    > ![Screenshot showing how to add a new task to your pipeline](../../pipelines/media/get-started-designer/builds-tab-add-task-tfs-2018-2.png)

1. Search for the **npm Authenticate** task, and then select **Add** to add it to your pipeline.

   > [!div class="mx-imgBorder"]
   > ![Screenshot showing the npm authenticate task](../media/build-definition/build-definition-npm-auth-task-phase.png)

1. Select your .npmrc file.

   > [!div class="mx-imgBorder"]
   > ![Screenshot showing how to add your .npmrc file to the npm authenticate task](../media/build-definition/build-definition-npm-auth-task-file.png)

1. Select **Save & queue** when you are done.

::: moniker-end

> [!TIP]
> To allow your pipeline access to your feed, make sure you give the build service a **Contributor** role in your feed's settings. Select **Azure Artifacts** -> [YOUR_FEED] -> **Settings** -> **Permissions** -> set the build service role to **Contributor**.

> [!div class="mx-imgBorder"]
> ![tip screenshot](../media/project-collection-contributor.png)

## Troubleshooting

- Command is not recognized:

    If you are running into the following error:
    
    - Cmd: `'vsts-npm-auth' is not recognized as an internal or external command, operable program or batch file.`
    - PowerShell: `vsts-npm-auth : The term 'vsts-npm-auth' is not recognized as the name of a cmdlet, function, script file, or operable program.`
    
    Then it's likely that the npm modules folder is not in your path. To fix this issue, rerun the Node.js setup and make sure that the `Add to PATH` options are selected.
    
    > [!div class="mx-imgBorder"]
    > ![Screenshot showing how to set up node.js](./media/node-setup.png)
    
    Alternatively, you can edit the PATH variable `%APPDATA%\npm` (Command Prompt) or `$env:APPDATA\npm` (PowerShell) to add it to your path.

- Unable to authenticate:

    If you are running into a E401 error: `code E401 npm ERR! Unable to authenticate`. Run the `vsts-npm-auth` command with the **-F** flag to reauthenticate.

    ```Command
    vsts-npm-auth -config .npmrc -F
    ```

## Related articles

- [Publish npm packages (YAML/Classic)](../../pipelines/artifacts/npm.md)
- [Use npm scopes](./scopes.md)
- [Use npm audit](./npm-audit.md)

