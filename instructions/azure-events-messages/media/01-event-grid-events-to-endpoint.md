---
lab:
    topic: Azure events and messaging
    title: 'Route events to a custom endpoint with Event Grid'
    description: 'Learn how to use Event Grid to route events to a custom endpoint.'
---

# Route events to a custom endpoint with Event Grid

In this exercise, you send events using the **Azure.Messaging.EventGrid** SDK to an Event Grid topic which are consumed and displayed on a message endpoint. 

Tasks performed in this exercise:

* Enable an Event Grid resource provider
* Create a topic in Event Grid
* Create a message endpoint
* Subscribe to the topic
* Send an event to a custom topic

This exercise takes approximately **30** minutes to complete.

## Before you start

To complete the exercise you need:

* An Azure subscription. If you don't already have one, you can sign up for one [https://azure.microsoft.com/](https://azure.microsoft.com/).

## Create resources in Azure using the Azure CLI

In this section of exercise you create the needed resources in Azure. 

1. In your browser navigate to the Azure portal [https://portal.azure.com](https://portal.azure.com); signing in with your Azure credentials if prompted.

1. Use the **[\>_]** button to the right of the search bar at the top of the page to create a new cloud shell in the Azure portal, selecting a ***Bash*** environment. The cloud shell provides a command line interface in a pane at the bottom of the Azure portal.

    > **Note**: If you have previously created a cloud shell that uses a *PowerShell* environment, switch it to ***Bash***.

1. In the cloud shell toolbar, in the **Settings** menu, select **Go to Classic version** (this is required to use the code editor).


1. Create a resource group for the resources needed for this exercise. If you already have a resource group you want to use, proceed to the next step. Replace **myResourceGroup** with a name you want to use for the resource group. You can replace **eastus** with a region near you if needed.

```bash

``


1. Many of the commands require unique names and use the same parameters. Creating some variables will reduce the changes needed to the commands that create resources. Run the following commands to create the needed variables. Rep

```bash
let rNum=$RANDOM
resourceGroup=myResourceGroup
location=eastus
topicName="mytopic-evgtopic-${rNum}"
siteName="az204-evgsite-${rNum}"
siteURL="https://${siteName}.azurewebsites.net"
```
