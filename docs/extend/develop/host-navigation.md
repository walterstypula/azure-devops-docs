---
ms.technology: devops-ecosystem
title: Host Page Navigation | Extensions for Azure DevOps
description: Use host navigation to modify host URL hash or reload page for Azure DevOps.
ms.assetid: 9B188838-55FD-98A6-9062-2652C06BF381
ms.topic: conceptual
monikerRange: '>= tfs-2017'
ms.author: chcomley
author: chcomley
ms.date: 08/04/2016
---

# Host page navigation

[!INCLUDE [version-gt-eq-2017](../../includes/version-gt-eq-2017.md)]

The HostNavigationService provides APIs for interacting with the parent host frame, including refreshing it and accessing the hash of the URL:

![example](./media/hash-browser.png)

[!INCLUDE [extension-docs-new-sdk](../../includes/extension-docs-new-sdk.md)]

### Get the current hash value

```js
	// Get navigation service
    VSS.getService(VSS.ServiceIds.Navigation).then(function(navigationService) {
        // Get current hash value from host url
        navigationService.getHash().then(function (hash) {
            console.log("Host hash value: " + hash);                        
        });
    });
```

### Get notified when the hash value changes

```js
	// Get navigation service
    VSS.getService(VSS.ServiceIds.Navigation).then(function(navigationService) {
        navigationService.onHashChanged(function (hash) {
        	// Adding #something to the end of browser url executes this handler with the hash value "something"
            console.log("Hash changed to : " + hash);                        
        });
    });
```

### Change the hash value

Two methods are available for changing the hash value of the host page URL: 

* `setHash` adds a new entry to the browser history
* `replaceHash` does **not** add a new entry to the browser history

```js
	// Get navigation service
    VSS.getService(VSS.ServiceIds.Navigation).then(function(navigationService) {
    	// Adds a new entry to browser history
        navigationService.setHash("new-hash-value");
    });
```

## Refresh the host page

Following code piece shows how host page can be reloaded.

```js
	// Get navigation service
    VSS.getService(VSS.ServiceIds.Navigation).then(function(navigationService) {
    	// Reload whole page
        navigationService.reload();
    });
```
