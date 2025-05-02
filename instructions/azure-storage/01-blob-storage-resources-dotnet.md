---
lab:
    topic: Azure Storage
    title: 'Create Blob storage resources with the .NET client library'
    description: 'Learn how to use the Azure Storage .NET client library to create containers, upload and list blobs, and delete containers.'
---

# Create Blob storage resources with the .NET client library

In this exercise, you create a console app that performs the following actions in Azure Blob storage:

* Create a container
* Upload blobs to a container
* List the blobs in a container
* Download blobs
* Delete a container

Tasks performed in this exercise:

* Prepare the Azure resources
* Create the console app
* Run the console app and verify results

This exercise should take approximately **30** minutes to complete.

## Before you start

Before you begin, make sure you have the following requirements in place:

* An Azure subscription. If you don't already have one, you can [sign up for one](https://azure.microsoft.com/).

* [Visual Studio Code](https://code.visualstudio.com/) on one of the [supported platforms](https://code.visualstudio.com/docs/supporting/requirements#_platforms).

* [.NET 8](https://dotnet.microsoft.com/en-us/download/dotnet/8.0) is the target framework.

* [C# Dev Kit](https://marketplace.visualstudio.com/items?itemName=ms-dotnettools.csdevkit) for Visual Studio Code.

## Create an Azure Storage account

In this section of exercise you create a resource group and Azure Storage account. You also record the endpoint, and access key for the account.

1. In your browser navigate to the Azure portal [https://portal.azure.com](https://portal.azure.com); signing in with your Azure credentials if prompted.

1. Use the **[\>_]** button to the right of the search bar at the top of the page to create a new cloud shell in the Azure portal, selecting a ***Bash*** environment. The cloud shell provides a command line interface in a pane at the bottom of the Azure portal.

    > **Note**: If you have previously created a cloud shell that uses a *PowerShell* environment, switch it to ***Bash***.

1. In the cloud shell toolbar, in the **Settings** menu, select **Go to Classic version** (this is required to use the code editor).

1. Create a resource group for the resources needed for this exercise. Replace **\<myResourceGroup>** with a name you want to use for the resource group. You can replace **useast** with a region near you if needed. 

    ```
    az group create --location useast --name <myResourceGroup>
    ```

1. Run the following commands to create the Azure Storage account. The first command creates a variable with a unique name for your storage account. Run the second command and replace **\<myResourceGroup>** with the group you chose earlier. Replace **\<myLocation>** with the location you used earlier.

    >**Note:** Storage account names must be between 3 and 24 characters in length and may contain numbers and lowercase letters only. Your storage account name must be unique within Azure. No two storage accounts can have the same name. *This command takes a few minutes to complete.*

    ```bash
    myStorageAcct=storageExercise$RANDOM
    ```

    ```
    az storage account create -g <myResourceGroup> -n $myStorageAcct -l <myLocation> --sku Standard_LRS
    ```

1.  Run the following command to retrieve the connection string for the Azure Storage account. Record the connection string from the command results, it's needed later in the exercise. Replace **\<myResourceGroup>** with the group you chose earlier.

    ```bash
    az storage account show-connection-string -n $myStorageAcct -g <myResourceGroup>
    ```

## Create the console application

NNow that the needed resources are deployed to Azure the next step is to set up the console application. The following steps are performed in your local environment.

1. Create a folder named **blobstorage** for the project.

1. Start **Visual Studio Code** and select the **File | Open Folder...** option in the menu bar to open the folder you created.

1. In the menu bar select the **Terminal | New Terminal** to open a terminal is Visual Studio Code. 

1. Run the following command in the terminal to create the .NET console app.

    ```dotnetcli
    dotnet new console --framework net8.0
    ```

1. Run the following commands to add the **Azure.Storage.Blobs** package to the project.

    ```dotnetcli
    dotnet add package Azure.Storage.Blobs
    ```

1. Run the following command to create a **data** folder in your project. 

    ```bash
    mkdir data
    ```

Now it's time to replace the template code in the **Program.cs** file.

### Add the starting code for the project

1. Open the *Program.cs* file and replace any existing code with the following code.  Be sure to replace the placeholder value for **storageConnectionString** following the directions in the code comments.

    The code provides the overall structure of the app, and some necessary elements. Review the comments in the code to get an understanding of how it works. To complete the application, you add code in specified areas later in the exercise. 

    ```csharp
    using Azure.Storage.Blobs;
    using Azure.Storage.Blobs.Models;
    
    Console.WriteLine("Azure Blob Storage exercise\n");
    
    // Run the examples asynchronously, wait for the results before proceeding
    ProcessAsync().GetAwaiter().GetResult();
    
    Console.WriteLine("Press enter to exit the sample application.");
    Console.ReadLine();
    
    static async Task ProcessAsync()
    {
        // Replace CONNECTION_STRING with the connection string you saved earlier
        string storageConnectionString = "CONNECTION_STRING";
        
        // Create a client that can authenticate with a connection string
        BlobServiceClient blobServiceClient = new BlobServiceClient(storageConnectionString);
    
        // COPY EXAMPLE CODE BELOW HERE
    
    }
    ```

### Add code to complete the application 

In this section of the exercise you add code in specified areas of the projects to create the full application. First, add code to create a container.

Creating a container includes creating an instance of the **BlobServiceClient** class, and then calling the **CreateBlobContainerAsync** method to create the container in your storage account. A GUID value is appended to the container name to ensure that it's unique. The **CreateBlobContainerAsync** method fails if the container already exists.

1. Add the following code after the **// COPY EXAMPLE CODE BELOW HERE** comment. 

    ```csharp
    //Create a unique name for the container
    string containerName = "wtblob" + Guid.NewGuid().ToString();
    
    // Create the container and return a container client object
    BlobContainerClient containerClient = await blobServiceClient.CreateBlobContainerAsync(containerName);
    Console.WriteLine("A container named '" + containerName + "' has been created. " +
        "\nTake a minute and verify in the portal." + 
        "\nNext a file will be created and uploaded to the container.");
    Console.WriteLine("Press 'Enter' to continue.");
    Console.ReadLine();
    ```

    Next, you add code to upload a generated file to the container.

1. Add the following code after previous code example. The code gets a reference to a **BlobClient** object by calling the **GetBlobClient** method on the container created in the previous section. It then uploads a generated local file using the **UploadAsync** method. This method creates the blob if it doesn't already exist, and overwrites it if it does.

    ```csharp
    // Create a local file in the ./data/ directory for uploading and downloading
    string localPath = "./data/";
    string fileName = "wtfile" + Guid.NewGuid().ToString() + ".txt";
    string localFilePath = Path.Combine(localPath, fileName);
    
    // Write text to the file
    await File.WriteAllTextAsync(localFilePath, "Hello, World!");
    
    // Get a reference to the blob
    BlobClient blobClient = containerClient.GetBlobClient(fileName);
    
    Console.WriteLine("Uploading to Blob storage as blob:\n\t {0}\n", blobClient.Uri);
    
    // Open the file and upload its data
    using (FileStream uploadFileStream = File.OpenRead(localFilePath))
        {
    await blobClient.UploadAsync(uploadFileStream);
    uploadFileStream.Close();
        }
    
    Console.WriteLine("\nThe file was uploaded. We'll verify by listing" + 
            " the blobs next.");
    Console.WriteLine("Press 'Enter' to continue.");
    Console.ReadLine();
    ```

    Next, you add code to list the blobs in the container.

1. Add the following code after previous code example. You list the blobs in the container by using the **GetBlobsAsync** method. In this case, only one blob was added to the container, so the listing operation returns just that one blob. 

    ```csharp
    // List blobs in the container
    Console.WriteLine("Listing blobs...");
    await foreach (BlobItem blobItem in containerClient.GetBlobsAsync())
    {
        Console.WriteLine("\t" + blobItem.Name);
    }
    
    Console.WriteLine("\nYou can also verify by looking inside the " + 
            "container in the portal." +
            "\nNext the blob will be downloaded with an altered file name.");
    Console.WriteLine("Press 'Enter' to continue.");
    Console.ReadLine();
    ```

    Next, you add code to download a blob.

1. Add the following code after the previous code example. The code uses the **DownloadAsync** method to download the blob created previously to your local file system. The example code adds a suffix of "DOWNLOADED" to the blob name so that you can see both files in local file system.

    ```csharp
    // Download the blob to a local file
    // Append the string "DOWNLOADED" before the .txt extension 
    string downloadFilePath = localFilePath.Replace(".txt", "DOWNLOADED.txt");
    
    Console.WriteLine("\nDownloading blob to\n\t{0}\n", downloadFilePath);
    
    // Download the blob's contents and save it to a file
    BlobDownloadInfo download = await blobClient.DownloadAsync();
    
    using (FileStream downloadFileStream = File.OpenWrite(downloadFilePath))
    {
        await download.Content.CopyToAsync(downloadFileStream);
    }
    Console.WriteLine("\nLocate the local file in the data directory created earlier to verify it was downloaded.");
    Console.WriteLine("The next step is to delete the container and local files.");
    Console.WriteLine("Press 'Enter' to continue.");
    Console.ReadLine();
    ```

    Next, you add code to delete a container.

1. Add the following code after the previous code example. The code cleans up the resources the app created by deleting the entire container using **DeleteAsync**. It also deletes the local files created by the app.

    ```csharp
    // Delete the container and clean up local files created
    Console.WriteLine("\n\nDeleting blob container...");
    await containerClient.DeleteAsync();
    
    Console.WriteLine("Deleting the local source and downloaded files...");
    File.Delete(localFilePath);
    File.Delete(downloadFilePath);
    
    Console.WriteLine("Finished cleaning up.");
    ```

## Run the code

Now that the app is complete it's time to build and run it. Run the following commands in the Visual Studio Code terminal. 

```bash
dotnet build
dotnet run
```

There are many prompts in the app to allow you to take the time to see what's happening in the portal after each step. Just open the Azure Portal and navigate to the storage account you created earlier. Then select **Data storage | Containers** in the resource pane.

## Clean up resources

Now that you finished the exercise, you should delete the cloud resources you created to avoid unnecessary resource usage.

1. Navigate to the resource group you created and view the contents of the resources used in this exercise.
1. On the toolbar, select **Delete resource group**.
1. Enter the resource group name and confirm that you want to delete it.

> **CAUTION:** Deleting a resource group deletes all resources contained within it. If you chose an existing resource group for this exercise, any existing resources outside the scope of this exercise will also be deleted.
