---
lab:
    topic: Azure authentication and authorization
    title: 'Implement interactive authentication with MSAL.NET'
    description: 'Learn implement interactive authentication using the MSAL.NET SDK and acquire a token.'
---

# Implement interactive authentication with MSAL.NET

In this exercise, you register a new application in Microsoft Entra ID (Azure AD), then create a .NET console application that uses the **Microsoft.Identity.Client** namespace to perform interactive authentication. 

Tasks performed in this exercise:

* Register an application with the Microsoft identity platform
* Create a .NET console app that implements the  **PublicClientApplicationBuilder** class to configure authentication.
* Acquire a token interactively using the **user.read** Microsoft Graph permission.

This exercise takes approximately **15** minutes to complete.

## Before you start

To complete the exercise you need:

* An Azure subscription. If you don't already have one, you can [sign up for one](https://azure.microsoft.com/).

## Register a new application

1. In your browser navigate to the Azure portal [https://portal.azure.com](https://portal.azure.com); signing in with your Azure credentials if prompted.

1. Use the **[\>_]** button to the right of the search bar at the top of the page to create a new cloud shell in the Azure portal, selecting a ***Bash*** environment. The cloud shell provides a command line interface in a pane at the bottom of the Azure portal.

    > **Note**: If you have previously created a cloud shell that uses a *PowerShell* environment, switch it to ***Bash***.

1. 