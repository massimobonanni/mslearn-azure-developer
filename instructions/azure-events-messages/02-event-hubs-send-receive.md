---
lab:
    topic: Azure events and messaging
    title: 'Send and retrieve events from Azure Event Hubs'
    description: 'Learn how send and retrieve events from Azure Event Hubs with the .NET Azure.Messaging.EventHubs SDK.'
---

# Send and retrieve events from Azure Event Hubs

In this exercise, you create Azure Event Hubs resources and build a .NET console app to send and receive events using the **Azure.Messaging.EventHubs** SDK. You learn how to provision cloud resources, interact with Event Hubs, and clean up your environment when finished.

Tasks performed in this exercise:

* Create a resource group
* Create Azure Event Hubs resources
* Create a .NET console app to send and retrieve events
* Clean up resources

This exercise takes approximately **30** minutes to complete.

## Before you start

To complete the exercise, you need:

* An Azure subscription. If you don't already have one, you can sign up for one [https://azure.microsoft.com/](https://azure.microsoft.com/).

## Create Azure Event Hubs resources

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
    namespaceName=eventhubsns$RANDOM
    ```

### Create an Azure Event Hubs namespace and event hub

An Azure Event Hubs namespace is a logical container for event hub resources within Azure. It provides a unique scoping container where you can create one or more event hubs, which are used to ingest, process, and store large volumes of event data. The following instructions are performed in the cloud shell. 

1. Run the following command to create an Event Hubs namespace.

    ```
    az eventhubs namespace create --name $namespaceName --resource-group $resourceGroup -l $location
    ```

1. Run the following command to create an event hub named **myEventHub** in the Event Hubs namespace. 

    ```
    az eventhubs eventhub create --name myEventHub --resource-group $resourceGroup \
      --namespace-name $namespaceName
    ```

## Send and retrieve events with a .NET console application

Now that the needed resources are deployed to Azure the next step is to set up the console application. The following steps are performed in the cloud shell.

1. Run the following commands to create a directory to contain the project and change into the project directory.

    ```
    mkdir eventhubs
    cd eventhubs
    ```

1. Create the .NET console application.

    ```
    dotnet new console --framework net8.0
    ```

1. Run the following commands to add the **Azure.Messaging.EventHubs** and **dotenv.net** packages to the project.

    ```
    dotnet add package Azure.Messaging.EventHubs
    dotnet add package dotenv.net
    ```

1. Run the following command to retrieve the connection string needed for the console application. Note that the name of the event hub is appended to the primary connection string of the namespace.

    ```
    eventhubConnStr=$(
      az eventhubs namespace authorization-rule keys list \
        --resource-group $resourceGroup \
        --namespace-name $namespaceName \
        --name RootManageSharedAccessKey \
        --query primaryConnectionString \
        --output tsv
    )";EntityPath=myEventHub"
    ```
    
    ```
    # Copy and save the connection string
    echo $eventhubConnStr
    ```

### Configure the console application

In this section you create, and edit, a **.env** file to hold the connection string you retrieved earlier. 

1. Run the following command to create the **.env** file to hold the secrets, and then open it in the code editor.

    ```
    touch .env
    code .env
    ```

1. Add the following code to the **.env** file. Replace **YOUR-CONNECTION-STRING** with the value you recorded earlier.

    ```
    EVENT_HUB_CONNECTION_STRING="YOUR-CONNECTION-STRING"
    EVENT_HUB_NAME="myEventHub"
    ```

1. Press **ctrl+s** to save the file, then **ctrl+q** to exit the editor.

Now it's time to replace the template code in the **Program.cs** file using the editor in the cloud shell.

### Add the starter code for the project

1. Run the following command in the cloud shell to begin editing the application.

    ```
    code Program.cs
    ```

1. Replace any existing contents with the following code. Be sure to review the comments in the code.

    ```csharp
    using Azure.Messaging.EventHubs;
    using Azure.Messaging.EventHubs.Producer;
    using Azure.Messaging.EventHubs.Consumer;
    using System.Text;
    using dotenv.net;
    
    // Load environment variables from .env file and assign to variables
    DotEnv.Load();
    var envVars = DotEnv.Read();
    string connectionString = envVars["EVENT_HUB_CONNECTION_STRING"];
    string eventHubName = envVars["EVENT_HUB_NAME"];
    
    // Check for empty connection string or event hub name
    if (string.IsNullOrWhiteSpace(connectionString) || string.IsNullOrWhiteSpace(eventHubName))
    {
        Console.WriteLine("Error: EVENT_HUB_CONNECTION_STRING and EVENT_HUB_NAME environment variables must be set.");
        return;
    }
    
    // Number of events to be sent to the event hub
    int numOfEvents = 3;
    
    // CREATE A PRODUCER CLIENT AND SEND EVENTS
    
    
    
    // CREATE A CONSUMER CLIENT AND RETRIEVE EVENTS
    
    
    ```

### Add code to complete the application

In this section you add code to create the producer and consumer clients to send and receive events.

1. Locate the **// CREATE A PRODUCER CLIENT AND SEND EVENTS** comment and add the following code directly after the comment. Be sure to review the comments in the code.

    ```csharp
    // Create a producer client to send events to the event hub
    EventHubProducerClient producerClient = new EventHubProducerClient(
        connectionString,
        eventHubName);
    
    // Create a batch of events 
    using EventDataBatch eventBatch = await producerClient.CreateBatchAsync();
    
    
    // Adding a random number to the event body and sending the events. 
    var random = new Random();
    for (int i = 1; i <= numOfEvents; i++)
    {
        int randomNumber = random.Next(1, 101); // 1 to 100 inclusive
        string eventBody = $"Event {randomNumber}";
        if (!eventBatch.TryAdd(new EventData(Encoding.UTF8.GetBytes(eventBody))))
        {
            // if it is too large for the batch
            throw new Exception($"Event {i} is too large for the batch and cannot be sent.");
        }
    }
    try
    {
        // Use the producer client to send the batch of events to the event hub
        await producerClient.SendAsync(eventBatch);
    
        Console.WriteLine($"A batch of {numOfEvents} events has been published.");
        Console.WriteLine("Press Enter to retrieve and print the events...");
        Console.ReadLine();
    }
    finally
    {
        await producerClient.DisposeAsync();
    }
    ```

1. Locate the **// CREATE A CONSUMER CLIENT AND RETRIEVE EVENTS** comment and add the following code directly after the comment. Be sure to review the comments in the code.

    ```csharp
    // Create an EventHubConsumerClient
    await using var consumerClient = new EventHubConsumerClient(
        EventHubConsumerClient.DefaultConsumerGroupName,
        connectionString,
        eventHubName);
    
    Console.WriteLine("Retrieving all events from the hub...");
    
    // Get total number of events in the hub by summing (last - first + 1) for all partitions
    long totalEventCount = 0;
    string[] partitionIds = await consumerClient.GetPartitionIdsAsync();
    foreach (var partitionId in partitionIds)
    {
        PartitionProperties properties = await consumerClient.GetPartitionPropertiesAsync(partitionId);
        if (properties.LastEnqueuedSequenceNumber >= properties.BeginningSequenceNumber)
        {
            totalEventCount += (properties.LastEnqueuedSequenceNumber - properties.BeginningSequenceNumber + 1);
        }
    }
    
    Console.WriteLine($"Total events in the hub: {totalEventCount}");
    
    // Start retrieving events from the event hub and print to the console
    int retrievedCount = 0;
    await foreach (PartitionEvent partitionEvent in consumerClient.ReadEventsAsync(startReadingAtEarliestEvent: true))
    {
        if (partitionEvent.Data != null)
        {
            string body = Encoding.UTF8.GetString(partitionEvent.Data.Body.ToArray());
            Console.WriteLine($"Retrieved event: {body}");
            retrievedCount++;
            if (retrievedCount >= totalEventCount)
            {
                Console.WriteLine("Done retrieving events. Press Enter to exit...");
                Console.ReadLine();
                return;
            }
        }
    }
    ```

1. Press **ctrl+s** to save the file, then **ctrl+q** to exit the editor.

## Run the application

Now that the app is complete it's time to build and run it. Start the application by running the following command:

```
dotnet run
```

After a few seconds you should see output similar to the following example:

```
A batch of 3 events has been published.
Press Enter to retrieve and print the events...

Receiving all events from the hub...
Total events in the hub: 3
Retrieved event: Event 4
Retrieved event: Event 96
Retrieved event: Event 74
Done retrieving events. Press Enter to exit...
```

The application always sends three events to the hub, but it retrieves all events in the hub. If you run the application multiple times an increasing number of events are retrieved. The random numbers used for event creation help you identify different events.

## Clean up resources

Now that you finished the exercise, you should delete the cloud resources you created to avoid unnecessary resource usage.

1. Navigate to the resource group you created and view the contents of the resources used in this exercise.
1. On the toolbar, select **Delete resource group**.
1. Enter the resource group name and confirm that you want to delete it.

> **CAUTION:** Deleting a resource group deletes all resources contained within it. If you chose an existing resource group for this exercise, any existing resources outside the scope of this 
