---
lab:
    topic: Secure solutions in Azure
    title: "Retrieve secrets from Azure Key Vault"
    description: "Learn how to implement interactive authentication using the MSAL.NET SDK and acquire a token."
---

# Retrieve secrets from Azure Key Vault

In this exercise, you  

Tasks performed in this exercise:

* Create Azure Key Vault resources
* Store a key-value pair using Azure CLI
* Create a .NET console app to retrieve the secret
* Clean up resources

This exercise takes approximately **30** minutes to complete.

## Before you start

To complete the exercise, you need:

* An Azure subscription. If you don't already have one, you can [sign up for one](https://azure.microsoft.com/).

## Create resources in Azure

1. In your browser navigate to the Azure portal [https://portal.azure.com](https://portal.azure.com); signing in with your Azure credentials if prompted.

1. Use the **[\>_]** button to the right of the search bar at the top of the page to create a new cloud shell in the Azure portal, selecting a ***Bash*** environment. The cloud shell provides a command line interface in a pane at the bottom of the Azure portal.

    > **Note**: If you have previously created a cloud shell that uses a *PowerShell* environment, switch it to ***Bash***.