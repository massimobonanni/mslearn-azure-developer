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

This exercise takes approximately **30** minutes to complete.

## Before you start

Before you begin, make sure you have the following requirements in place:

* An Azure subscription. If you don't already have one, you can [sign up for one](https://azure.microsoft.com/).
* [Visual Studio Code](https://code.visualstudio.com/) on one of the [supported platforms](https://code.visualstudio.com/docs/supporting/requirements#_platforms). 
* [C# extension](https://marketplace.visualstudio.com/items?itemName=ms-dotnettools.csharp) for Visual Studio Code.
* [.NET 8](https://dotnet.microsoft.com/en-us/download/dotnet/8.0) is the target framework.
* [Azure CLI](https://learn.microsoft.com/cli/azure/install-azure-cli) installed locally.

## Create an Azure Storage account

In this section of exercise you create a resource group and Azure Storage account. You also record the endpoint, and access key for the account.

1. In your browser navigate to the Azure portal [https://portal.azure.com](https://portal.azure.com); signing in with your Azure credentials if prompted.

1. Use the **[\>_]** button to the right of the search bar at the top of the page to create a new cloud shell in the Azure portal, selecting a ***Bash*** environment. The cloud shell provides a command line interface in a pane at the bottom of the Azure portal.

    > **Note**: If you have previously created a cloud shell that uses a *PowerShell* environment, switch it to ***Bash***.

1. Create a resource group for the resources needed for this exercise. Replace **myResourceGroup** with a name you want to use for the resource group. You can replace **eastus2** with a region near you if needed. If you already have a resource group you want to use, proceed to the next step.

    ```azurecli
    az group create --location eastus2 --name myResourceGroup
    ```

1. Run the following commands to create the Azure Storage account. The first command creates a variable with a unique name for your storage account. Run the second command and replace **myResourceGroup** with the group you chose earlier. Replace **myLocation** with the location you used earlier.

    >**Note:** Storage account names must be between 3 and 24 characters in length and may contain numbers and lowercase letters only. Your storage account name must be unique within Azure. No two storage accounts can have the same name. *This command takes a few minutes to complete.*

    ```bash
    myStorageAcct=storageexercise$RANDOM
    ```

    ```bash
    az storage account create -g myResourceGroup -n $myStorageAcct -l myLocation --sku Standard_LRS
    ```

1. Run the following command to retrieve the blob endpoint for the Azure Storage account. Replace **myResourceGroup** with the group you chose earlier. Record the endpoint from the command results, it's needed later in the exercise. 

    ```bash
    az storage account show -n $myStorageAcct -g myResourceGroup  --query primaryEndpoints | jq '.blob'
    ```

Now that the needed resources are deployed to Azure the next step is to set up the console application. The rest of the exercise is performed in your local environment.

## Download files for the project

In this section you download the starter files for the project. You add code to the starter files to complete the application.

1. Paste the link below into a web browser and save the file. 

    `https://raw.githubusercontent.com/MicrosoftLearning/mslearn-azure-developer/main/allfiles/downloads/dotnet/azure-storage-dotnet.zip`

1. Launch **File Explorer** and navigate to the location the file was saved.

1. Unzip the file into it's own folder.

## Configure the application

1. Start **Visual Studio Code** and select the **File > Open Folder...** option in the menu bar. Open the folder *01-blob-storage-resources-dotnet* located inside the unzipped file.

1. In the menu bar select the **Terminal > New Terminal** to open a terminal is Visual Studio Code. 

1. Run the following commands in the terminal to add the required packages in the application.

    ```bash
    dotnet add package Azure.Storage.Blobs
    dotnet add package dotenv.net
    dotnet add package Azure.Identity
    ```

1. Run the following command to create a **data** folder in your project. 

    ```bash
    mkdir data
    ```

1. Open the *.env* configuration file and replace **YOUR_BLOB_STORAGE_URL** with the endpoint value you saved earlier. Save your changes.

Now it's time to complete the code for the project.

## Add code to complete the project

Open the *Program.cs* file and review the comments to get an understanding of the overall flow of the application. Throughout the rest of the exercise you add code in specified areas to create the full application. 

1. Find the **// CREATE A BLOB STORAGE CLIENT** comment, then add the following code directly beneath the comment. The **BlobServiceClient** acts as the primary entry point for managing containers and blobs in a storage account. The client uses the *DefaultAzureCredential* for authentication.

    ```csharp
    // Create a client that authenticates with DefaultAzureCredential
    BlobServiceClient blobServiceClient = 
        new BlobServiceClient(new Uri(envVars["BLOB_STORAGE_URL"]), new DefaultAzureCredential());
    ```

1. Find the **// CREATE A CONTAINER** comment, then add the following code directly beneath the comment. Creating a container includes creating an instance of the **BlobServiceClient** class, and then calling the **CreateBlobContainerAsync** method to create the container in your storage account. A GUID value is appended to the container name to ensure that it's unique. The **CreateBlobContainerAsync** method fails if the container already exists.

    ```csharp
    //Create a unique name for the container
    string containerName = "wtblob" + Guid.NewGuid().ToString();
    
    // Create the container and return a container client object
    Console.WriteLine("Creating container: " + containerName);
    BlobContainerClient containerClient = 
        await blobServiceClient.CreateBlobContainerAsync(containerName);
    
    // Check if the container was created successfully
    if (containerClient != null)
    {
        Console.WriteLine("Container created successfully, press 'Enter' to continue.");
        Console.ReadLine();
    }
    else
    {
        Console.WriteLine("Failed to create the container, exiting program.");
        return;
    }
    ```
    

1. Find the **// CREATE A LOCAL FILE FOR UPLOAD TO BLOB STORAGE** comment, then add the following code directly beneath the comment. This creates a file in the data directory that is uploaded to the container.

    ```csharp
    // Create a local file in the ./data/ directory for uploading and downloading
    Console.WriteLine("Creating a local file for upload to Blob storage...");
    string localPath = "./data/";
    string fileName = "wtfile" + Guid.NewGuid().ToString() + ".txt";
    string localFilePath = Path.Combine(localPath, fileName);

    // Write text to the file
    await File.WriteAllTextAsync(localFilePath, "Hello, World!");
    Console.WriteLine("Local file created, press 'Enter' to continue.");
    Console.ReadLine();
    ```

1. Find the **// UPLOAD THE FILE TO BLOB STORAGE** comment, then add the following code directly beneath the comment. The code gets a reference to a **BlobClient** object by calling the **GetBlobClient** method on the container created in the previous section. It then uploads a generated local file using the **UploadAsync** method. This method creates the blob if it doesn't already exist, and overwrites it if it does.

    ```csharp
    // Get a reference to the blob and upload the file
    BlobClient blobClient = containerClient.GetBlobClient(fileName);
    
    Console.WriteLine("Uploading to Blob storage as blob:\n\t {0}", blobClient.Uri);
    
    // Open the file and upload its data
    using (FileStream uploadFileStream = File.OpenRead(localFilePath))
    {
        await blobClient.UploadAsync(uploadFileStream);
        uploadFileStream.Close();
    }
    
    // Verify if the file was uploaded successfully
    bool blobExists = await blobClient.ExistsAsync();
    if (blobExists)
    {
        Console.WriteLine("File uploaded successfully, press 'Enter' to continue.");
        Console.ReadLine();
    }
    else
    {
        Console.WriteLine("File upload failed, exiting program..");
        return;
    }
    ```

1. Find the **// LIST THE CONTAINER'S BLOBS** comment, then add the following code directly beneath the comment. You list the blobs in the container by using the **GetBlobsAsync** method. In this case, only one blob was added to the container, so the listing operation returns just that one blob. 

    ```csharp
    // List blobs in the container
    Console.WriteLine("Listing blobs in container...");
    await foreach (BlobItem blobItem in containerClient.GetBlobsAsync())
    {
        Console.WriteLine("\t" + blobItem.Name);
    }
    
    Console.WriteLine("Press 'Enter' to continue.");
    Console.ReadLine();
    ```

1. Find the **// DOWNLOAD THE BLOB TO A LOCAL FILE** comment, then add the following code directly beneath the comment. The code uses the **DownloadAsync** method to download the blob created previously to your local file system. The example code adds a suffix of "DOWNLOADED" to the blob name so that you can see both files in local file system. 

    ```csharp
    // Add the string "DOWNLOADED" before the .txt extension so it doesn't 
    // overwrite the original file
    
    string downloadFilePath = localFilePath.Replace(".txt", "DOWNLOADED.txt");
    
    Console.WriteLine("Downloading blob to: {0}", downloadFilePath);
    
    // Download the blob's contents and save it to a file
    BlobDownloadInfo download = await blobClient.DownloadAsync();
    
    using (FileStream downloadFileStream = File.OpenWrite(downloadFilePath))
    {
        await download.Content.CopyToAsync(downloadFileStream);
    }
    
    Console.WriteLine("Locate the local file in the 'data' directory created earlier to verify it was downloaded.");
    Console.WriteLine("Press 'Enter' to continue.");
    Console.ReadLine();
    ```

1. Find the **// DELETE THE BLOB AND CONTAINER** comment, then add the following code directly beneath the comment. The code cleans up the resources the app created by deleting the entire container using **DeleteAsync**. It also deletes the local files created by the app. 

    ```csharp
    // Delete the container and the local files
    Console.WriteLine("Delete container and local files. Press 'Enter' to continue.");
    Console.ReadLine();
    
    await containerClient.DeleteAsync();
    Console.WriteLine("Container deleted successfully.");
    
    Console.WriteLine("Deleting the local source and downloaded files...");
    File.Delete(localFilePath);
    File.Delete(downloadFilePath);
    
    Console.WriteLine("Finished cleaning up.");
    ```

## Run the application

Now that the app is complete it's time to build and run it. Because you're using the **DefaultAzureCredential** for authentication you need to login to Azure from the Visual Studio Code terminal.

Follow the steps below to login to Azure:

1. Run the `az login` command in the Visual Studio Code terminal to start the login process.

1. Enter your credentials in the pop-up window that appears.

1. A list of Azure subscriptions will appear in the Visual Studio Code terminal. Select the subscription you used earlier to create the resources. 

Next, run the following command to start the application:

    ```bash
    dotnet run
    ```

There are many prompts in the app to allow you to take the time to see what's happening in the portal after each step. Just open the Azure Portal and navigate to the storage account you created earlier. Then select **Data storage > Containers** in the resource pane.

## Clean up resources

Now that you finished the exercise, you should delete the cloud resources you created to avoid unnecessary resource usage.

1. Navigate to the resource group you created and view the contents of the resources used in this exercise.
1. On the toolbar, select **Delete resource group**.
1. Enter the resource group name and confirm that you want to delete it.

> **CAUTION:** Deleting a resource group deletes all resources contained within it. If you chose an existing resource group for this exercise, any existing resources outside the scope of this exercise will also be deleted.
