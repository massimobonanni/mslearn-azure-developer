---
lab:
    topic: Secure solutions in Azure
    title: "Retrieve secrets from Azure Key Vault"
    description: "Learn how to store secrets in Azure Key Vault, and retrieve them in an application."
---

# Retrieve secrets from Azure Key Vault

In this exercise, you  

Tasks performed in this exercise:

* Create Azure Key Vault resources
* Store a secret in a key vault using Azure CLI
* Create a .NET console app to retrieve the secret
* Clean up resources

This exercise takes approximately **30** minutes to complete.

## Before you start

To complete the exercise, you need:

* An Azure subscription. If you don't already have one, you can [sign up for one](https://azure.microsoft.com/).

## Create Azure Key Vault resources and add a secret

In this section of the exercise you create the needed resources in Azure with the Azure CLI.

1. In your browser navigate to the Azure portal [https://portal.azure.com](https://portal.azure.com); signing in with your Azure credentials if prompted.

1. Use the **[\>_]** button to the right of the search bar at the top of the page to create a new cloud shell in the Azure portal, selecting a ***Bash*** environment. The cloud shell provides a command line interface in a pane at the bottom of the Azure portal.

    > **Note**: If you have previously created a cloud shell that uses a *PowerShell* environment, switch it to ***Bash***.

1. In the cloud shell toolbar, in the **Settings** menu, select **Go to Classic version** (this is required to use the code editor).

1. Create a resource group for the resources needed for this exercise. If you already have a resource group you want to use, proceed to the next step. Replace **myResourceGroup** with a name you want to use for the resource group. You can replace **eastus** with a region near you if needed.

    ```
    az group create --name myResourceGroup --location eastus
    ```

1. Many of the commands require unique names and use the same parameters. Creating some variables will reduce the changes needed to the commands that create resources. Run the following commands to create the needed variables. Replace **myResourceGroup** with the name you you're using for this exercise. If you changed the location in the previous step, make the same change in the **location** variable.

    ```
    resourceGroup=myResourceGroup
    location=eastus
    keyVaultName=mykeyvaultname$RANDOM
    ```

1. Run the following command to create an Azure Key Vault resource. This can take a few minutes to run.

    ```
    az keyvault create --name $keyVaultName \
        --resource-group $resourceGroup --location $location
    ```

### Assign a role to your Microsoft Entra user name

To create and retrieve a secret, assign your Microsoft Entra user to the **Key Vault Secrets Officer** role. This gives your user account permission to set, delete, and list secrets. In a typical scenario you may want to separate the create/read actions by assigning the **Key Vault Secrets Officer** to one group, and **Key Vault Secrets User** (can get and list secrets) to another.

1. Run the following command to retrieve the **userPrincipalName** from your account. This represents who the role will be assigned to.

    ```
    userPrincipal=$(az rest --method GET --url https://graph.microsoft.com/v1.0/me \
        --headers 'Content-Type=application/json' \
        --query userPrincipalName --output tsv)
    ```

1. Run the following command to retrieve the resource ID of the Service Bus namespace. The resource ID sets the scope for the role assignment to a specific namespace.

    ```
    resourceID=$(az keyvault show --resource-group $resourceGroup \
        --name $keyVaultName --query id --output tsv)
    ```

1. Run the following command to create and assign the **Key Vault Secrets Officer** role.

    ```
    az role assignment create --assignee $userPrincipal \
        --role "Key Vault Secrets Officer" \
        --scope $resourceID
    ```

Next, add a secret to the key vault you created.

### Add and retrieve a secret with Azure CLI

1. Run the following command to create a secret. 

    ```
    az keyvault secret set --vault-name $keyVaultName \
        --name "MySecret" --value "My secret value"
    ```

1. Run the following command to retrieve the secret to verify it was set.

    ```
    az keyvault secret show --name "MySecret" --vault-name $keyVaultName
    ```

    This command returns some JSON. The last line contains the password in plain text. 

    ```json
    "value": "My secret value"
    ```

## Create a .NET console app to send and receive messages

Now that the needed resources are deployed to Azure the next step is to set up the console application. The following steps are performed in the cloud shell.

1. Run the following commands to create a directory to contain the project and change into the project directory.

    ```
    mkdir keyvault
    cd keyvault
    ```

1. Create the .NET console application.

    ```
    dotnet new console --framework net8.0
    ```

1. Run the following commands to add the **Azure.Identity** and **package 2** packages to the project.

    ```
    dotnet add package Azure.Storage.Queues
    dotnet add package Azure.Identity
    ```

### Add the starter code for the project

1. Run the following command in the cloud shell to begin editing the application.

    ```
    code Program.cs
    ```

1. Replace any existing contents with the following code. Be sure to review the comments in the code, and replace **<YOUR-STORAGE-ACCT-NAME>** with the storage account name you recorded earlier.

```csharp
using Azure;
using Azure.Identity;
using Azure.Storage.Queues;
using Azure.Storage.Queues.Models;
using System;
using System.Threading.Tasks;

// Create a unique name for the queue
// TODO: Replace the <YOUR-STORAGE-ACCT-NAME> placeholder 
string queueName = "myqueue-" + Guid.NewGuid().ToString();
string storageAccountName = "<YOUR-STORAGE-ACCT-NAME>";

// ADD CODE TO CREATE A QUEUE CLIENT AND CREATE A QUEUE



// ADD CODE TO SEND AND LIST MESSAGES



// ADD CODE TO UPDATE A MESSAGE AND LIST MESSAGES



// ADD CODE TO DELETE MESSAGES AND THE QUEUE


```

1. Press **ctrl+s** to save your changes.

### Add code to create a queue client and create a queue

Now it's time to add code to create the queue storage client and create a queue.

1. Locate the **// ADD CODE TO CREATE A QUEUE CLIENT AND CREATE A QUEUE** comment and add the following code directly after the comment. Be sure to review the code and comments.


