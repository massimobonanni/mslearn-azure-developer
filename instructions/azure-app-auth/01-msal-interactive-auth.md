---
lab:
    topic: Azure authentication and authorization
    title: 'Implement interactive authentication with MSAL.NET'
    description: 'Learn how to implement interactive authentication using the MSAL.NET SDK and acquire a token.'
---

# Implement interactive authentication with MSAL.NET

In this exercise, you register an application in Microsoft Entra ID, then create a .NET console application that uses MSAL.NET to perform interactive authentication and acquire an access token for Microsoft Graph. You learn how to configure authentication scopes, handle user consent, and see how tokens are cached for subsequent runs. 

Tasks performed in this exercise:

* Register an application with the Microsoft identity platform
* Create a .NET console app that implements the  **PublicClientApplicationBuilder** class to configure authentication.
* Acquire a token interactively using the **user.read** Microsoft Graph permission.

This exercise takes approximately **15** minutes to complete.

## Register a new application

1. In your browser navigate to the Azure portal [https://portal.azure.com](https://portal.azure.com); signing in with your Azure credentials if prompted.

1. Use the **[\>_]** button to the right of the search bar at the top of the page to create a new cloud shell in the Azure portal, selecting a ***Bash*** environment. The cloud shell provides a command line interface in a pane at the bottom of the Azure portal.

    > **Note**: If you have previously created a cloud shell that uses a *PowerShell* environment, switch it to ***Bash***.

1. In the cloud shell toolbar, in the **Settings** menu, select **Go to Classic version** (this is required to use the code editor).

1. In the portal, search for and select **App registrations**. 

1. Select **+ New registration**, and when the **Register an application** page appears, enter your application's registration information:

    | Field | Value |
    |--|--|
    | **Name** | Enter `myMsalApplication`  |
    | **Supported account types** | Select **Accounts in this organizational directory only** |
    | **Redirect URI (optional)** | Select **Public client/native (mobile & desktop)** and enter `http://localhost` in the box to the right. |

1. Select **Register**. Microsoft Entra ID assigns a unique application (client) ID to your app, and you're taken to your application's **Overview** page. 

1. In the **Essentials** section of the **Overview** page record the **Application (client) ID** and the **Directory (tenant) ID**. The information is needed for the application.

    ![Screenshot showing the location of the fields to copy.](./media/01-app-directory-id-location.png)
 
## Create a .NET console app to send and receive messages

Now that the needed resources are deployed to Azure the next step is to set up the console application. The following steps are performed in the cloud shell.

1. Run the following commands to create a directory to contain the project and change into the project directory.

    ```
    mkdir authapp
    cd authapp
    ```

1. Create the .NET console application.

    ```
    dotnet new console
    ```

1. Run the following commands to add the **Microsoft.Identity.Client** and **dotenv.net** packages to the project.

    ```
    dotnet add package Microsoft.Identity.Client
    dotnet add package dotenv.net
    ```

### Configure the console application

In this section you create, and edit, a **.env** file to hold the secrets you recorded earlier. 

1. Run the following command to create the **.env** file, and then open it in the code editor.

    ```
    touch .env
    code .env
    ```

1. Add the following code to the **.env** file. Replace **YOUR_CLIENT_ID**, and **YOUR_TENANT_ID** with the values you recorded earlier.

    ```
    CLIENT_ID="YOUR_CLIENT_ID"
    TENANT_ID="YOUR_TENANT_ID"
    ```

1. Press **ctrl+s** to save the file, then **ctrl+q** to exit the editor.

### Add the starter code for the project

1. Run the following command in the cloud shell to begin editing the application.

    ```
    code Program.cs
    ```

1. Replace any existing contents with the following code. Be sure to review the comments in the code.

    ```csharp
    using Microsoft.Identity.Client;
    using dotenv.net;
    
    // Load environment variables from .env file
    DotEnv.Load();
    var envVars = DotEnv.Read();
    
    // Retrieve Azure AD Application ID and tenant ID from environment variables
    string _clientId = envVars["CLIENT_ID"];
    string _tenantId = envVars["TENANT_ID"];
    
    // ADD CODE TO DEFINE SCOPES AND CREATE CLIENT 
    
    
    
    // ADD CODE TO ACQUIRE AN ACCESS TOKEN
    
    
    ```

1. Press **ctrl+s** to save your changes.

### Add code to complete the application

1. Locate the **// ADD CODE TO DEFINE SCOPES AND CREATE CLIENT** comment and add the following code directly after the comment. Be sure to review the comments in the code.

    ```csharp
    // Define the scopes required for authentication
    string[] _scopes = { "User.Read" };
    
    // Build the MSAL public client application with authority and redirect URI
    var app = PublicClientApplicationBuilder.Create(_clientId)
        .WithAuthority(AzureCloudInstance.AzurePublic, _tenantId)
        .WithDefaultRedirectUri()
        .Build();
    ```

1. Locate the **// ADD CODE TO ACQUIRE AN ACCESS TOKEN** comment and add the following code directly after the comment. Be sure to review the comments in the code.

    ```csharp
    // Attempt to acquire an access token silently or interactively
    AuthenticationResult result;
    try
    {
        // Try to acquire token silently from cache for the first available account
        var accounts = await app.GetAccountsAsync();
        result = await app.AcquireTokenSilent(_scopes, accounts.FirstOrDefault())
                    .ExecuteAsync();
    }
    catch (MsalUiRequiredException)
    {
        // If silent token acquisition fails, prompt the user interactively
        result = await app.AcquireTokenInteractive(_scopes)
                    .ExecuteAsync();
    }
    
    // Output the acquired access token to the console
    Console.WriteLine($"Access Token:\n{result.AccessToken}");
    ```

1. Press **ctrl+s** to save the file, then **ctrl+q** to exit the editor.

## Run the application

Now that the app is complete it's time to run it. 

1. Start the application by running the following command:

    ```
    dotnet run
    ```

1. The app opens the default browser prompting you to select the account you want to authenticate with. If there are multiple accounts listed select the one associated with the tenant used in the app.

1. If this is the first time you've authenticated to the registered app you receive a **Permissions requested** notification asking you to approve the app to sign you in and read your profile, and maintain access to data you have given it access to. Select **Accept**.

    ![Screenshot showing the permissions requested notification](./media/01-granting-permission.png)

1. You should see the results similar to the example below in the console.

    ```
    Access Token:
    eyJ0eXAiOiJKV1QiLCJub25jZSI6IlZF.........
    ```

1. Start the application a second time and notice you no longer receive the **Permissions requested** notification. The permission you granted earlier was cached.

## Clean up resources

Now that you finished the exercise, you should delete the app registration you created earlier.

1. In the Azure portal, navigate to the app registration you created.
1. On the toolbar, select **Delete**.
1. Confirm the deletion.