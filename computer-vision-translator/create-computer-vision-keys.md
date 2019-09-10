# Creating Azure resources

[Prerequisite: Exploring app.py](./explore-app-py.md)

With our Flask application created, it's time to turn our attention to incorporating Computer Vision. Computer Vision is a part of [Cognitive Services](https://azure.microsoft.com/services/cognitive-services), a suite of AI tools built by Microsoft.

We will explore how to create keys our keys.

## Overview of Computer Vision

Azure Cognitive Services is a set of more than 20 services and APIs for infusing intelligence backed by machine learning and neural networks into the applications that you write. One member of the Cognitive Services family is the [Computer Vision API](https://azure.microsoft.com/services/cognitive-services/computer-vision/), which can analyze images uploaded to it and:

- Identify objects in the images.
- Generate captions for the images (for example, "A woman riding a bicycle").
- Use Optical Character Recognition (OCR) to extract text from the images.
- Find faces in the images and identify attributes of those faces, such as age and gender.
- Generate "smart thumbnails" centered around the subjects of the images.
- Recognize famous people and landmarks in the images.

## Azure Command-Line Interface (CLI)

If you aren't familiar with the Azure CLI, you can learn more about it and the numerous commands it supports in [Get started with Azure CLI](https://docs.microsoft.com/en-us/cli/azure/get-started-with-azure-cli?view=azure-cli-latest). Most operations in Azure can be performed by using either the CLI or the [Azure Portal](https://portal.azure.com). Power users tend to prefer the CLI, in part because CLI commands can be used in scripts to automate repetitive tasks.

### Install the Azure (CLI)

The [Azure CLI](https://docs.microsoft.com/cli/azure/get-started-with-azure-cli?view=azure-cli-latest) is a command-line environment for creating and managing Azure resources. Versions are available for Windows, macOS, and Linux. In subsequent unites you'll use the Azure CLI to create various Azure resources. Here you will install the Azure CLI and login to Azure.

If you haven't already installed Azure CLI, visit the [Azure CLI web page](https://docs.microsoft.com/en-us/cli/azure/install-azure-cli?view=azure-cli-latest) and follow the installation instructions. You can determine if it's already installed by running `az -v` in a console window, which will display the verson number.

### Login to Azure using the Azure CLI

The process will be completed using a browser window (the command will automatically open it); the browser can be closed once login is complete.

``` bash
az login
```

### Choose subscription (optional)

You may have multiple subscriptions for Azure associated with your account. We will be using the default subscription. If you aren't sure which one the Azure CLI is using, or you need to change it, you can use the following commands.

``` bash
# List all subscriptions
az account list

# Change the default (if needed)
az account set -s <SUBSCRIPTION_ID>
```

## Create a Cognitive Services Key

In order to use any Cognitive Service, you must first obtain an API key. This key travels in each request you place in an HTTP header named `Ocp-Apim-Subscription-Key`. It is Azure's way of authenticating the caller and determining which Azure subscription to bill calls to. Most Azure Cognitive Service APIs have free tiers for which no billing is performed, but if you plan to place thousands of calls a day to a Cognitive Services API, you will be billed for it through your Azure subscription.

You can create a separate key for each service, or create an [All-in-One](https://portal.azure.com/#create/Microsoft.CognitiveServicesAllInOne) key. The All-in-One key supports every Cognitive Service, with the exception of QnA Maker, Speech Services and Custom Vision (not to be mistaken with Computer Vision, which is what we are using). The All-in-One key is only available as a paid service at this time. If you wish to use the free tier, you will need to create a key for each service.

> **Note:** For purposes of this demo, we will be creating a key for each service so you can use the Free tier on Azure. However, if you wish to [create an All-in-One key](https://portal.azure.com/#create/Microsoft.CognitiveServicesAllInOne), you are free to do so.

You can obtain a Computer Vision API key using the [Azure CLI](https://docs.microsoft.com/cli/azure/get-started-with-azure-cli?view=azure-cli-latest) or the [Azure Portal](https://portal.azure.com/). In this exercise, you will obtain an API key and a corresponding URL for placing calls to the Computer Vision API with that key by using the Azure CLI.

### Create a Resource Group

Resource groups are a key component of [resource management](https://docs.microsoft.com/en-us/azure/azure-resource-manager/resource-group-overview) in Azure. They act as containers for other Azure resources and serve to group those resources together so that you can view billing information for them as a group, apply security rules as a group, and even delete them as a group. Every Azure resource that you create must be part of a resource group.

Open a Command Prompt window or terminal, and use the following command to create a resource group named **contoso-travel** in Azure’s North Central US region to hold all of the Azure resources you create in this module:

``` bash
az group create  --name contoso-travel-rg --location northcentralus
```

> **NOTE:** When choosing locations, it's important to ensure all related services are created in the same location. Making calls across various Azure locations is the leading cause of poor performing applications on Azure. For our purposes, we're going to locate everything in **northcentralus**.

### Create a key for Computer Vision API

Now use the following command to subscribe to the Computer Vision API and place the resulting resource named **computer-vision** in the resource group created previously. Make sure you **log the endpoint url** provided by the output.

``` bash
az cognitiveservices account create --resource-group contoso-travel-rg --name computer-vision --location northcentralus --kind ComputerVision --sku F0 --yes
```

> **NOTE:** The --sku F0 parameter subscribes to the free tier of the Computer Vision API that allows up to 20 calls per minute and a maximum of 5,000 calls per month. This is fine for development, but in production, you would want to subscribe to one of the paid tiers that supports more traffic. For a summation of pricing tiers for the Computer Vision API, see [Cognitive Services Pricing—Computer Vision API](https://azure.microsoft.com/pricing/details/cognitive-services/computer-vision/).

### Obtain the newly created key for Computer Vision API

The key is not automatically displayed upon creation. To retrieve it, use the following command:

``` bash
az cognitiveservices account keys list --resource-group contoso-travel-rg --name computer-vision --query key1 --output tsv
```

> **NOTE:** The output from the command is a string containing numbers and letters. This is your Computer Vision API key. Copy the key into a text file and save it so that you can easily retrieve it later. You will need it later in this course.

## Summary and next steps

We've explored how to create keys for Computer Vision Service and Translator Text. Next, we'll see how we [can call Computer Vision](./computer-vision.md) from our application.
