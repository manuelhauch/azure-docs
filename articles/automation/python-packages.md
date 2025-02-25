---
title: Manage Python 2 packages in Azure Automation
description: This article tells how to manage Python 2 packages in Azure Automation.
services: automation
ms.subservice: process-automation
ms.date: 08/13/2021
ms.topic: conceptual
ms.custom: devx-track-python
---

# Manage Python 2 packages in Azure Automation

Azure Automation allows you to run Python 2 runbooks on Azure and on Linux Hybrid Runbook Workers. To help in simplification of runbooks, you can use Python packages to import the modules that you need. This article describes how to manage and use Python packages in Azure Automation.

## Import packages

In your Automation account, select **Python 2 packages** under **Shared Resources**. Click **+ Add a Python 2 package**.

:::image type="content" source="media/python-packages/add-python-package.png" alt-text="Screenshot of the Python 2 packages page shows Python 2 packages in the left menu and Add a Python 2 package highlighted.":::

On the Add Python 2 Package page, select a local package to upload. The package can be a **.whl** or **.tar.gz** file. When the package is selected, click **OK** to upload it.

:::image type="content" source="media/python-packages/upload-package.png" alt-text="Screenshot shows the Add Python 2 Package page with an uploaded tar.gz file selected.":::

Once a package has been imported, it's listed on the Python 2 packages page in your Automation account. If you need to remove a package, select the package and click **Delete**.

:::image type="content" source="media/python-packages/package-list.png" alt-text="Screenshot shows the Python 2 packages page after a package has been imported.":::

## Import packages with dependencies

Azure automation doesn't resolve dependencies for Python packages during the import process. There are two ways to import a package with all its dependencies. Only one of the following steps needs to be used to import the packages into your Automation account.

### Manually download

On a Windows 64-bit machine with [Python2.7](https://www.python.org/downloads/release/latest/python2) and [pip](https://pip.pypa.io/en/stable/) installed, run the following command to download a package and all its dependencies:

```cmd
C:\Python27\Scripts\pip2.7.exe download -d <output dir> <package name>
```

Once the packages are downloaded, you can import them into your automation account.

### Runbook

 To obtain a runbook, [import Python 2 packages from pypi into Azure Automation account](https://github.com/azureautomation/import-python-2-packages-from-pypi-into-azure-automation-account) from the Azure Automation GitHub organization into your Automation account. Make sure the Run Settings are set to **Azure** and start the runbook with the parameters. The runbook requires a Run As account for the Automation account to work. For each parameter make sure you start it with the switch as seen in the following list and image:

* -s \<subscriptionId\>
* -g \<resourceGroup\>
* -a \<automationAccount\>
* -m \<modulePackage\>

:::image type="content" source="media/python-packages/import-python-runbook.png" alt-text="Screenshot shows the Overview page for  import_py2package_from_pypi with the Start Runbook pane on the right side.":::

The runbook allows you to specify what package to download. For example, use of the `Azure` parameter downloads all Azure modules and all dependencies (about 105).

Once the runbook is complete, you can check the **Python 2 packages** under **Shared Resources** in your Automation account to verify that the package has been imported correctly.

## Use a package in a runbook

With a package imported, you can use it in a runbook. The following example uses the [Azure Automation utility package](https://github.com/azureautomation/azure_automation_utility). This package makes it easier to use Python with Azure Automation. To use the package, follow the instructions in the GitHub repository and add it to the runbook. For example, you can use `from azure_automation_utility import get_automation_runas_credential` to import the function for retrieving the Run As account.

```python
import azure.mgmt.resource
import automationassets
from azure_automation_utility import get_automation_runas_credential

# Authenticate to Azure using the Azure Automation RunAs service principal
runas_connection = automationassets.get_automation_connection("AzureRunAsConnection")
azure_credential = get_automation_runas_credential()

# Intialize the resource management client with the RunAs credential and subscription
resource_client = azure.mgmt.resource.ResourceManagementClient(
    azure_credential,
    str(runas_connection["SubscriptionId"]))

# Get list of resource groups and print them out
groups = resource_client.resource_groups.list()
for group in groups:
    print group.name
```

> [!NOTE]
> The Python `automationassets` package is not available on pypi.org, so it's not available for import onto a Windows machine.

## Develop and test runbooks offline

To develop and test your Python 2 runbooks offline, you can use the [Azure Automation Python emulated assets](https://github.com/azureautomation/python_emulated_assets) module on GitHub. This module allows you to reference your shared resources such as credentials, variables, connections, and certificates.

## Next steps

To prepare a Python runbook, see [Create a Python runbook](./learn/automation-tutorial-runbook-textual-python-3.md).