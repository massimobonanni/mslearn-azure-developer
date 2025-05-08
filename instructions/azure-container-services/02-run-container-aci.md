---
lab:
    topic: Azure container services
    title: 'Deploy a container to Azure Container Instances with the Azure CLI'
    description: 'Learn how to use Azure CLI commands to deploy a container to Azure Container Instances.'
---

# Build and run a container image with Azure Container Registry Tasks

In this exercise, you create a console app that creates a container, database, and an item in Azure Cosmos DB.

Tasks performed in this exercise:

* Create a resource group for the container
* Create a container
* Verify the container is running

This exercise takes approximately **20** minutes to complete.

## Before you start

Before you begin, make sure you have the following requirements in place:

* An Azure subscription. If you don't already have one, you can [sign up for one](https://azure.microsoft.com/).

## Create a resource group

1. In your browser navigate to the Azure portal [https://portal.azure.com](https://portal.azure.com); signing in with your Azure credentials if prompted.

1. Use the **[\>_]** button to the right of the search bar at the top of the page to create a new cloud shell in the Azure portal, selecting a ***Bash*** environment. The cloud shell provides a command line interface in a pane at the bottom of the Azure portal.

    > **Note**: If you have previously created a cloud shell that uses a *PowerShell* environment, switch it to ***Bash***.

1. Create a resource group for the resources needed for this exercise. Replace **\<myResourceGroup>** with a name you want to use for the resource group. You can replace **useast** with a region near you if needed. 

    ```
    az group create --location useast --name <myResourceGroup>
    ```

## Create a container

You create a container by providing a name, a Docker image, and an Azure resource group to the **az container create** command. You expose the container to the Internet by specifying a DNS name label.

1. Run the following command to create a DNS name used to expose your container to the Internet. Your DNS name must be unique, run this command from Cloud Shell to create a variable that holds a unique name.

    ```bash
    DNS_NAME_LABEL=aci-example-$RANDOM
    ```

1. Run the following command to create a container instance. Replace **\<myResourceGroup>** and **\<myLocation>** with the values you used earlier. It takes a few minutes for the operation to complete.

    ```bash
    az container create --resource-group <myResourceGroup> 
        --name mycontainer 
        --image mcr.microsoft.com/azuredocs/aci-helloworld 
        --ports 80 
        --dns-name-label $DNS_NAME_LABEL --location <myLocation> 
    ```

    In the previous command, **$DNS_NAME_LABEL** specifies your DNS name. The image name, **mcr.microsoft.com/azuredocs/aci-helloworld**, refers to a Docker image that runs a basic Node.js web application.

Move to the next section after the **az container create** command is finished.

## Verify the container is running

You can check the containers build status with the **az container show** command. 

1. Run the following command to check the provisioning status of the container you created. Replace **\<myResourceGroup>** with the value you used earlier.

    ```bash
    az container show --resource-group <myResourceGroup> 
        --name mycontainer 
        --query "{FQDN:ipAddress.fqdn,ProvisioningState:provisioningState}" 
        --out table 
    ```

    You see your container's fully qualified domain name (FQDN) and its provisioning state. Here's an example.

    ```
    FQDN                                    ProvisioningState
    --------------------------------------  -------------------
    aci-wt.eastus.azurecontainer.io         Succeeded
    ```

    > **Note:** If your container is in the **Creating** state, wait a few moments and run the command again until you see the **Succeeded** state.

1. From a browser, navigate to your container's FQDN to see it running. The image isn't running You may get a warning that the site isn't safe.

## Clean up resources

Now that you finished the exercise, you should delete the cloud resources you created to avoid unnecessary resource usage.

1. Navigate to the resource group you created and view the contents of the resources used in this exercise.
1. On the toolbar, select **Delete resource group**.
1. Enter the resource group name and confirm that you want to delete it.

> **CAUTION:** Deleting a resource group deletes all resources contained within it. If you chose an existing resource group for this exercise, any existing resources outside the scope of this exercise will also be deleted.
