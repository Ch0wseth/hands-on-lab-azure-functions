---
published: true
type: workshop
title: Product Hands-on Lab - Azure Functions deep dive
short_title: Azure Functions
description: This workshop will cover the integration of Azure Functions Flex Consumption service with other Azure service. You will use it to build a complete real world scenario.
level: beginner # Required. Can be 'beginner', 'intermediate' or 'advanced'
navigation_numbering: false
authors: # Required. You can add as many authors as needed
  - Iheb Khemissi
  - Julien Strebler
  - Damien Aicheh
contacts: # Required. Must match the number of authors
  - "@ikhemissi"
  - "@justrebl"
  - "@damienaicheh"
duration_minutes: 360
tags: azure, azure functions, azure durable functions, flex, open ai, apim, monitoring, cosmos db, csu, codespace, devcontainer
navigation_levels: 3
---

# Product Hands-on Lab - Azure Functions

Welcome to this Azure Functions Workshop. You'll be experimenting with Azure Functions service in multiple labs to achieve a real world scenario. We will use the Azure Functions Flex consumption plan for all of these labs which contains the latest features of Azure Functions. Don't worry, even if the challenges will increase in difficulty, this is a step by step lab, you will be guided through the whole process.

During this workshop you will have the instructions to complete each steps. It is recommended to search for the answers in provided resources and links before looking at the solutions placed under the '📚 Toggle solution' panel.

<div class="task" data-title="Task">

> - You will find the instructions and expected configurations for each Lab step in these yellow **TASK** boxes.
> - Inputs and parameters to select will be defined, all the rest can remain as default as it has no impact on the scenario.
>
> - Log into your Azure subscription locally using Azure CLI and on the [Azure Portal][az-portal] using your own credentials.
> - Instructions and solutions will be given for the Azure CLI, but you can also use the Azure Portal if you prefer.

</div>


## Scenario

The goal of the full lab is to upload an audio file to Azure and save the transcripts back inside a Cosmos DB database and enrich these transcriptions with a summary using Azure OpenAI. The scenario is as follows:

![Hand's On Lab Architecture](assets/architecture-overview.png)

1. You will use Azure Load Testing to be able to simulate the traffic on your system
1. All requests will go through APIM (API Management). This includes the requests for fetching transcriptions and for uploading audio files.
1. The first Azure Function (standard function) will be mainly responsible for uploading the audio file to the Storage Account.
1. Whenever a blob is uploaded to the Storage Account, a `BlobCreated` event will be emitted to Event Grid
1. The Event Grid System Topic will push the event (in real time) to trigger the Azure Durable Function
1. The Azure Durable Function will start processing the audio file
1. The Azure Durable Function will use the Speech To Text service for audio transcription. It will use the Monitor pattern to check every few seconds if the transcription is done.
1. The Azure Durable Function will retrieve the transcription from the Speech to Text service
1. The Azure Durable Function will use Azure OpenAI to generate a summary of the audio file from the transcription
1. The Azure Durable Function will then store the transcription and its summary in Cosmos DB

You will also learn:
- How to use managed identity to secure the access to Azure services.
- How to monitor and observe Azure Functions

## Programming language

You will have to create few functions in this workshop to address our overall scenario. You can choose the programming language you are the most comfortable with among the ones [supported by Azure Functions][az-func-languages]. We will provide solutions in .NET 8 (isolated) for the moment, but other languages might be added in the future.

With everything ready let's start the lab 🚀

## Pre-requisites

Before starting this lab, be sure to set your Azure environment :

- An Azure Subscription with the **Owner** role to create and manage the labs' resources entirely and deploy the infrastructure as code and managed identities.
- The permission to register resource providers on your Azure Subscription (if not done yet). You can find the command lines to run in the **Sign in to Azure** section later: `Microsoft.CognitiveServices`, `Microsoft.DocumentDB`, `Microsoft.ApiManagement`, `Microsoft.Web`, `Microsoft.LoadTestService`, `Microsoft.KeyVault`, `Microsoft.EventGrid`.
- You will also need a GitHub Account (Free, Team or Enterprise) to clone the workshop and potentially work on it using Github Codespaces.

To retrieve the lab content :

- Create a [fork][repo-fork] of the repository from the **main** branch to help you keep track of your changes

3 development options are available:
  - 🥇 **Preferred method** : Pre-configured GitHub Codespace 
  - 🥈 Local Devcontainer
  - 🥉 Local Dev Environment with all the prerequisites detailed below

<div class="tip" data-title="Tips">

> - To focus on the main purpose of the lab, we encourage the usage of codespace or DevContainer as they abstract the dev environment configuration, and avoid potential local dependencies conflict.
> 
> - You could decide to run everything without relying on a DevContainer : To do so, make sure you install all the prerequisites detailed below.

</div>

### 🥇 : Pre-configured GitHub Codespace

To use a Github Codespace, you will need :
- [A GitHub Account][github-account]

Github Codespace offers the ability to run a complete dev environment (Visual Studio Code, Extensions, Tools, Secure port forwarding etc.) on a dedicated virtual machine. 
The configuration for the environment is defined in the `.devcontainer` folder, making sure everyone gets to develop and practice on identical environments : No more conflict on dependencies or missing tools ! 

Every Github account (even the free ones) grants access to 120 vCPU hours per month, _**for free**_. A 2 vCPU dedicated environment is enough for the purpose of the lab, meaning you could run such environment for 60 hours a month at no cost!

To get your codespace ready for the labs, here are a few steps to execute : 
- After you forked the repo, click on `<> Code`, `Codespaces` tab and then click on the `+` button:

![codespace-new](./assets/codespace-new.png)

- You can also provision a beefier configuration by defining creation options and select the **Machine Type** you like: 

![codespace-configure](./assets/codespace-configure.png)

### 🥈 : Using a local Devcontainer

This repo comes with a Devcontainer configuration that will let you open a fully configured dev environment from your local Visual Studio Code, while still being completely isolated from the rest of your local machine configuration : No more dependency conflict.
Here are the required tools to do so : 

- [Git client][git-client] 
- [Docker Desktop][docker-desktop] running
- [Visual Studio Code][vs-code] installed

Start by cloning the Hands-on-lab-Functions repo you just forked on your local Machine and open the local folder in Visual Studio Code.
Once you have cloned the repository locally, make sure Docker Desktop is up and running and open the cloned repository in Visual Studio Code.  

You will be prompted to open the project in a Dev Container. Click on `Reopen in Container`. 

If you are not prompted by Visual Studio Code, you can open the command palette (`Ctrl + Shift + P`) and search for `Reopen in Container` and select it: 

![devcontainer-reopen](./assets/devcontainer-reopen.png)

### 🥉 : Using your own local environment

The following tools and access will be necessary to run the lab in good conditions on a local environment :  

- [Git client][git-client] 
- [Visual Studio Code][vs-code] installed (you will use Dev Containers)
- [Azure CLI][az-cli-install] installed on your machine
- [Azure Functions Core Tools][az-func-core-tools] installed, this will be useful for creating the scaffold of your Azure Functions using command line.
- If you are using VS Code, you can also install the [Azure Function extension][azure-function-vs-code-extension]
- [.Net 8][download-dotnet] if you want to run all the Azure Functions solutions.
- [Terraform][download-terraform] to deploy the infrastructure as code.

Once you have set up your local environment, you can clone the hands-on-lab-azure-functions repo you just forked on your machine, and open the local folder in Visual Studio Code and head to the next step. 

## Visual Studio Code Setup

### 👉 Load the Workspace

Once your environment is ready, you will have to enter the Visual Studio Workspace to get all the tools ready.
To do so, click the **burger menu** in the top left corner (visible only with codespace), **File** and then **Open Workspace from File...** 

![codespace-workspace](./assets/codespace-workspace.png)

- Select `.vscode/hands-on-lab-azure-functions.code-workspace` :

![codespace-workspace-select](./assets/codespace-workspace-select.png)

- You are now ready to go! For the rest of the lab, in case you lose the terminal, you can press `Ctrl + J` or open a new one here : 

![codespace-terminal-new](./assets/codespace-terminal-new.png)

Let's begin!

### 🔑 Sign in to Azure

<div class="task" data-title="Task">

> - Log into your Azure subscription in your environment using Azure CLI and on the [Azure Portal][az-portal] using your credentials.
> - Instructions and solutions will be given for the Azure CLI, but you can also use the Azure Portal if you prefer.

</div>

<details>

<summary>📚 Toggle solution</summary>

```bash
# Login to Azure : 
# --tenant : Optional | In case your Azure account has access to multiple tenants
# The tenant id can be easily find by searching for "Tenant properties" in the search bar of the Azure Portal.

# Option 1 : Local Environment or Dev Container
az login --tenant <your-tenant-id or domain.com>
# Option 2 : Github Codespace : you might need to specify --use-device-code parameter to ease the az cli authentication process
az login --use-device-code --tenant <your-tenant-id or domain.com>

# Display your account details
az account show
# Select your Azure subscription
az account set --subscription <subscription-id>

# Register the following Azure providers if they are not already

# Azure Cognitive Services
az provider register --namespace 'Microsoft.CognitiveServices'
# Azure CosmosDb
az provider register --namespace 'Microsoft.DocumentDB'
# Azure API Management
az provider register --namespace 'Microsoft.ApiManagement'
# Azure Functions
az provider register --namespace 'Microsoft.Web'
# Azure Load Test Service
az provider register --namespace 'Microsoft.LoadTestService'
# Azure Key Vault
az provider register --namespace 'Microsoft.KeyVault'
# Azure Event Grid
az provider register --namespace 'Microsoft.EventGrid'
```

</details>

### Deploy the infrastructure

You must deploy the infrastructure before starting the lab. 

First, you need to initialize the Terraform infrastructure by running the following command:

```bash
cd terraform && terraform init
```

Since `azurerm` Terraform provider version 4 you need to specify the Subscription ID to be able to deploy, so you must retrieve it and expose it using the environment variable `ARM_SUBSCRIPTION_ID` like this:

```bash
export ARM_SUBSCRIPTION_ID=$(az account show --query id -o tsv)
```
 
Then run the following command to deploy the infrastructure if you have the **Owner** role on the subscription:

```bash
terraform plan -out plan.out
```

Finally, apply the deployment:

```bash
terraform apply plan.out
```

The deployment should take around 5 minutes to complete, you can continue reading the next part in parallel.

[az-cli-install]: https://learn.microsoft.com/en-us/cli/azure/install-azure-cli
[az-func-core-tools]: https://learn.microsoft.com/en-us/azure/azure-functions/functions-run-local?tabs=v4%2Clinux%2Ccsharp%2Cportal%2Cbash#install-the-azure-functions-core-tools
[az-func-languages]: https://learn.microsoft.com/en-us/azure/azure-functions/functions-versions#languages
[azure-function-vs-code-extension]: https://marketplace.visualstudio.com/items?itemName=ms-azuretools.vscode-azurefunctions
[az-portal]: https://portal.azure.com
[docker-desktop]: https://www.docker.com/products/docker-desktop/
[download-dotnet]: https://dotnet.microsoft.com/en-us/download/dotnet/8.0
[github-account]: https://github.com/join
[git-client]: https://git-scm.com/downloads
[repo-fork]: https://github.com/microsoft/hands-on-lab-azure-functions/fork
[vs-code]: https://code.visualstudio.com/
[download-terraform]: https://developer.hashicorp.com/terraform/install

---

# Lab 1 : Upload an audio file

For this first lab, you will focus on the following scope :

![Hand's On Lab Architecture Lab 1](assets/azure-functions-lab1.png)

The Azure Storage Account will be used to store the audios files inside the `audios` container.

To check that everything was created as expected, open the [Azure Portal][az-portal] and select the storage account which is **not** starting with `stfstd` or `stfdrbl` those two are for the two Azure Functions.

In the third one, you should retrieve your `audios` container:

![Storage account access keys](assets/storage-account-show-container.png)

## A bit of theory

### Azure Functions

Azure Functions is a `compute-on-demand` solution, offering a common function programming model for various languages. To use this serverless solution, no need to worry about deploying and maintaining infrastructures, Azure provides with the necessary up-to-date compute resources needed to keep your applications running. Focus on your code and let Azure Functions handle the rest.

Azure Functions are event-driven : They must be triggered by an event coming from a variety of sources. This model is based on a set of `triggers` and `bindings` which let you avoid hard-coding access to other services. Your function receives data (for example, the content of a queue message) in function parameters. You send data (for example, to create a queue message) by using the return value of the function :

- `Binding` to a function is a way of connecting another resource to the function in a declarative way; bindings can be used to fetch data (input bindings), write data (output bindings), or both. Azure services such as Azure Storage blobs and queues, Service Bus queues, Event Hubs, and Cosmos DB provide data to the function as parameters.
- `Triggers` are a specific kind of binding that causes a function to run. A trigger defines how a function is invoked, and a function must have exactly one trigger. Triggers have associated data, which is often provided as a parameter payload to the function.

In the same `Function App` you will be able to add multiple `functions`, each with its own set of triggers and bindings. These triggers and bindings can benefit from existing `expressions`, which are parameter conventions easing the overall development experience. For example, you can use an expression to use the execution timestamp, or generate a unique `GUID` name for a file uploaded to a storage account.

Azure Functions run and benefit from the App Service platform, offering features like: deployment slots, continuous deployment, HTTPS support, hybrid connections and others. Apart from the `Flex Consumption` (Serverless) model we're most interested in this Lab, Azure Functions can also be deployed a dedicated `Consumption`, `App Service Plan` or in a hybrid model called `Premium Plan`.

### Managed identities

Security is our first concern at Microsoft. To avoid any credential management issues, the best practice is to use managed identities on Azure. They offer several key benefits:

- **Enhanced Security**: Managed identities eliminate the need to store credentials in your code, reducing the risk of accidental leaks or breaches.
- **Simplified Credential Management**: Azure automatically handles the lifecycle of these identities, so you don’t need to manually manage secrets, passwords, or keys.
- **Seamless Integration**: Managed identities can authenticate to any Azure service that supports Microsoft Entra ID authentication, making it easier to connect and secure your applications.
- **Cost Efficiency**: There are no additional charges for using managed identities, making it a cost-effective solution for securing your Azure resources.

In this workshop, you will only be using Managed Identities to secure service-to-service communications.

## Creating the Function App

At this stage in our scenario, the goal is to upload an audio file into the Storage Account inside the `audios` container. To achieve this, an Azure Function will be used as an API to upload the audio file with a unique `GUID` name to your Storage Account.

<div class="task" data-title="Tasks">

> - Create an `Azure Function` with a POST `HTTP Trigger` and a `Blob Output Binding` to upload the file to the Storage Account. The Blob Output Binding will use a `binding expression` to generate a unique `GUID` name for the file.
>
> - Use the `func` CLI tool and .NET 8 using the isolated mode to create this Azure Function
> - Set the `Connection` parameter to `AudioUploadStorage` to use the Managed Identity to access the Storage Account.

</div>

<div class="tip" data-title="Tips">

> - [Azure Functions][azure-function]<br> 
> - [Azure Function Core Tools][azure-function-core-tools]<br> 
> - [Basics of Azure Functions][azure-function-basics]<br> 
> - [HTTP Triggered Azure Function][azure-function-http]<br>
> - [Blob Output Binding][azure-function-blob-output]<br> 
> - [Azure Functions Binding Expressions][azure-function-bindings-expression]

</div>

<details>
<summary>📚 Toggle solution</summary>

You can refer to the solutions in the workshop's Github Repository, under `./src/solutions/Lab1FuncStd`.

### Preparation

You will create a function using the [Azure Function Core Tools][azure-function-core-tools]:

```bash
# At the root of your repository, create a folder for your function app and navigate to it
mkdir FuncStd
cd FuncStd

# Create the new function app as a .NET 8 Isolated project
# No need to specify a name, the folder name will be used by default
func init --worker-runtime dotnet-isolated --target-framework net8.0

# Create a new function endpoint with an HTTP trigger to which you'll be able to send the audio file
func new --name AudioUpload --template 'HTTP Trigger'

# Add a new Nuget package dependency to the Blob storage SDK
dotnet add package Microsoft.Azure.Functions.Worker.Extensions.Storage.Blobs --version 6.7.0
```

### .NET 8 implementation

In this version of the implementation, you will be using the [.NET 8 Isolated][in-process-vs-isolated] runtime.

Now that you have a skeleton for our `AudioUpload` function in the `AudioUpload.cs` file, you will need to update it to meet the following goals:

- It should read the uploaded file from the body of the POST request
- It should store the file as a blob inside the blob Storage Account
- It should respond to user with a status code 200

To upload the file, you will rely on the blob output binding [`BlobOutput`][blob-output] of the Azure Function, which will take care of the logic of connecting to the Storage Account and uploading the function with minimal line of code in our side.

To do this, let's start by adding a `AudioUploadOutput` class to the `AudioUpload.cs` file.
For simplicity reasons you will be reusing the existing file to add the class, but keep in mind that you can also opt for adding it in its own dedicated file.

```csharp
public class AudioUploadOutput
{
    [BlobOutput("%STORAGE_ACCOUNT_CONTAINER%/{rand-guid}.wav", Connection = "AudioUploadStorage")]
    public byte[] Blob { get; set; }

    public required IActionResult HttpResponse { get; set; }
}
```

This class will handle uploading the blob and returning the HTTP response:

- The blob will be stored in the container identified by `STORAGE_ACCOUNT_CONTAINER` which is an environment variable.
- The blob will be named `{rand-guid}.wav` which resolves to a UUID followed by `.wav`.
- `AudioUploadStorage` is the name of the prefix in the App setting which will be used to connect to the blob storage account using a managed identity. Behind the scenes the Azure Function will fetch the app setting called `AudioUploadStorage__serviceUri` and use it to connect to the Storage Account where audio files are uploaded. 

In fact if you open the Azure Function App resource started with `func-std` in the [Azure Portal][az-portal] and go to the `Environment variables` panel. You should see in App Settings the `STORAGE_ACCOUNT_CONTAINER` set to `audios` and another entry called `AudioUploadStorage__serviceUri` used to locate the storage account.

As part of the workshop, the `Storage Blob Data Owner` was already assigned to your Azure Function's managed identity. This role together with the `AudioUploadStorage__serviceUri` environment variable will allow the function to access the Storage account securely without needing to use a connection string.

Next, you will need to update the class `AudioUpload` to add the logic for reading the file from the request, and then use `AudioUploadOutput` to perform the blob upload and returning the response.

Update the code of the `Run` method in the `AudioUpload` class as follows:

```csharp
[Function(nameof(AudioUpload))]
public AudioUploadOutput Run(
    [HttpTrigger(AuthorizationLevel.Anonymous, "post")] HttpRequest req
)
{
    _logger.LogInformation("Processing a new audio file upload request");

    // Get the first file in the form
    byte[]? audioFileData = null;
    var file = req.Form.Files[0];

    using (var memstream = new MemoryStream())
    {
        file.OpenReadStream().CopyTo(memstream);
        audioFileData = memstream.ToArray();
    }

    // Store the file as a blob and return a success response
    return new AudioUploadOutput()
    {
        Blob = audioFileData,
        HttpResponse = new OkObjectResult("Uploaded!")
    };
}
```

</details>

## Testing locally

### Run the function locally

Add the following environment variables to your `local.settings.json` file:

```json
{
  "IsEncrypted": false,
  "Values": {
    "AzureWebJobsStorage": "UseDevelopmentStorage=true",
    "FUNCTIONS_WORKER_RUNTIME": "dotnet-isolated",
    "STORAGE_ACCOUNT_CONTAINER": "audios",
    "AudioUploadStorage": "UseDevelopmentStorage=true"
  }
}
```

To test your function locally, you will need to start the extension `Azurite` to emulate the Azure Storage Account. Just run `Ctrl` + `Shift` + `P` and search for `Azurite: Start`:

![Start Azurite](assets/function-azurite.png)

Then you can use the Azure Function Core Tools to run the function locally:

```bash
func start
```

if you have an error such as:

> Can't determine Project to build. Expected 1 .csproj or .fsproj but found 2

Just remove the bin and obj folder and run the command again, the issue is currently being corrected.

```bash
rm -rf bin/ && rm -rf obj/ && func start
```

<div class="tip" data-title="Tips">

> - If you are using Github Codespaces for testing, and you encounter authentication issues (e.g. 401) or infinite loop, then first make sure the visibility of the port `7071` is either set to "Public" or that you are passing a valid Github token in the `X-Github-Token` header.
> - Please refer to this [port forwarding guide on Github Codespaces](https://docs.github.com/en/codespaces/developing-in-a-codespace/forwarding-ports-in-your-codespace#using-command-line-tools-and-rest-clients-to-access-ports) for more details.
> - If you are using a DevContainer, look at the forwarded address in the port tab of VS Code.

</div>

### Upload an audio file

Upload an audio file to Azurite's blob storage using the function running locally.

To do that you can use one of the sample audio files provided in the workshop:

- [Microsoft AI](assets/audios/MicrosoftAI.wav)
- [Azure Functions](assets/audios/AzureFunctions.wav)

Next, run the following command to upload the audio file. You can also use Postman or another HTTP client if you have previously opted for using a dev container or a local dev environment.

```sh
curl -v -F audio=@docs/assets/audios/MicrosoftAI.wav http://localhost:7071/api/AudioUpload
```

### Check blob creation

Finally, make sure that the audio file was saved in Azurite as a blob with the name `[GUID].wav`.

We will use the [Azure Storage extension][azure-storage-extension] to list available blobs in the `audios` container in Azurite (Local Emulator):

![Start Azurite](assets/azurite-explorer.png)

You can repeat the same test commands to ensure new files get saved in Azurite whenever you upload a file using the function running locally.

## Deployment to Azure

### Option 1 : Deploy your function with VS Code

- Open the Azure extension in VS Code left panel
- Make sure you're signed in to your Azure account
- Open the Function App panel
- Right-click on your function app and select `Deploy to Function App...`
- Select the Function starting with `func-std-`

![Deploy to Function App](assets/function-app-deploy.png)

### Option 2 : Deploy your function with the Azure Function Core Tools

Deploy your function using the VS Code extension or via command line:

```bash
# Inside the FuncStd folder run the following command:
func azure functionapp publish func-std-<your-instance-suffix-name>
```

## Test the Function App deployed in Azure

Let's give the new function a try using [Postman][postman]. Go to the Azure Function and select `Functions` then `AudioUpload` and select the `Get Function Url` with the `default (function key)`.
The Azure Function url is protected by a code to ensure a basic security layer. 

![Azure Function url credentials](assets/func-url-credentials.png)

Use this url with Postman to upload the audio file.

You can use the provided sample audio files to test the function:

- [Microsoft AI](assets/audios/MicrosoftAI.wav)
- [Azure Functions](assets/audios/AzureFunctions.wav)

Create a POST request and in the row where you set the key to `audio` for instance then, make sure to select the `file` option in the hidden dropdown menu to be able to select a file in the value field:

![Postman](assets/func-postman.png)

Go back to the Storage Account and check the `audios` container. You should see the files that you uploaded with your `AudioUpload` Azure Function!

</details>

## Save your changes

Don't forget to commit your changes to your forked repository to keep track of your progress. **You will need the code for the next labs.**

You can commit directly on the `main` branch of the repository:

Open a terminal and run the following commands:

```bash
# Add, commit and push your changes
git add .
git commit -m "Lab 1 - Azure Function to upload audio file"
git push
```

You are now ready for the next labs!

## Lab 1 : Summary

By now you should have a solution which stores uploaded audio files within a blob storage using a first Azure Function. Audio files are stored inside an `audios` container.

The first Azure Function API created in the Lab offers a first security layer to the solution as it requires a key to be called, as well as makes sure all the files are stores with a uniquely generated name (GUID). We will go further in the next labs.

[az-portal]: https://portal.azure.com
[azure-function]: https://learn.microsoft.com/en-us/cli/azure/functionapp?view=azure-cli-latest
[azure-function-core-tools]: https://learn.microsoft.com/en-us/azure/azure-functions/functions-run-local?tabs=v4%2Cwindows%2Ccsharp%2Cportal%2Cbash
[azure-function-basics]: https://learn.microsoft.com/en-us/azure/azure-functions/supported-languages
[azure-function-http]: https://learn.microsoft.com/en-us/azure/azure-functions/functions-bindings-http-webhook-trigger?pivots=programming-language-python&tabs=python-v2%2Cin-process%2Cfunctionsv2
[azure-function-blob-output]: https://learn.microsoft.com/en-us/azure/azure-functions/functions-bindings-storage-blob-output?pivots=programming-language-python&tabs=python-v2%2Cin-process
[azure-function-bindings-expression]: https://learn.microsoft.com/en-us/azure/azure-functions/functions-bindings-expressions-patterns
[in-process-vs-isolated]: https://learn.microsoft.com/en-us/azure/azure-functions/dotnet-isolated-in-process-differences
[blob-output]: https://learn.microsoft.com/en-us/azure/azure-functions/functions-bindings-storage-blob-output?tabs=python-v2%2Cin-process&pivots=programming-language-csharp
[azure-managed-identity]: https://learn.microsoft.com/en-us/azure/azure-functions/functions-bindings-storage-blob-output?tabs=python-v2%2Cisolated-process%2Cnodejs-v4&pivots=programming-language-csharp#identity-based-connections
[azure-storage-extension]: https://marketplace.visualstudio.com/items?itemName=ms-azuretools.vscode-azurestorage#:~:text=Installation.%20Download%20and%20install%20the%20Azure%20Storage%20extension%20for%20Visual
[postman]: https://www.postman.com/

---

# Lab 2 : Process the audio file with an Azure Durable Function

On this lab, you will focus on the following scope :

![Hand's On Lab Architecture Lab](assets/azure-functions-lab3.png)

Processing the audio file involves the following actions:
- Detecting file uploads
- Creating a transcript of the file
- Saving the transcript to Azure Cosmos DB
- Generating a summary with Azure OpenAI

To ensure the execution of all these steps and to orchestrate all of this process, you will need to create a Durable Function.
Durable Function is an extension of Azure Functions that lets you write stateful functions in a serverless environment. The extension manages state, checkpoints, and restarts for you.

## Detect a file upload event 

### Listen to the audio upload

Now, you have the audio file uploaded in the storage account, you will need to detect this event to trigger the next steps of the scenario.

<div class="task" data-title="Tasks">

> - Create a new `Durable Function` with a `Blob Trigger` to detect the file upload event based on Event Grid and start the processing of the audio file.
>
> - Use the `func` CLI tool and .NET 8 using the isolated mode to create this Durable Function.
> - Use the `Audio.cs` file below to instantiate an `AudioFile` object when the Azure Function is triggered.
> - Create an `AudioTranscriptionOrchestration.cs` file which will be used to create the orchestration of the entire Azure Function.
> - Generate a URI with a SAS token to access the blob storage.
> - Add an Event Grid subscription to detect the upload of audios in real time.

</div>

<div class="tip" data-title="Tips">

> - [Azure Functions][azure-function]<br>
> - [Azure Functions Binding Expressions][azure-function-bindings-expression]<br>
> - [Azure Function Blob Triggered][azure-function-blob-trigger]<br>
> - [Azure Function Blob Trigger with Event Grid][blob-trigger-event-grid]<br>

</div>

The `Audio.cs` file will be used to create an `AudioFile` object and also an `AudioTranscription` object when the transcription is done, this will be used to store the data in Cosmos DB in the next step.

<details>
<summary>📄 Audio.cs</summary>

```csharp
using System.Text.Json.Serialization;

namespace FuncDurable
{
    public abstract class Audio
    {
        [JsonPropertyName("id")]
        public string Id { get; set; }
        
        // Blob path uri
        [JsonPropertyName("path")]
        public string Path { get; set; }
    }

    public class AudioFile : Audio
    {
        [JsonPropertyName("urlWithSasToken")]
        public string UrlWithSasToken { get; set; }

        [JsonPropertyName("jobUri")]
        public string? JobUri { get; set; }
    }

    public class AudioTranscription : Audio
    {
        [JsonPropertyName("result")]
        public string Result { get; set; }

        [JsonPropertyName("status")]
        public string Status { get; set; }

        [JsonPropertyName("completion")]
        public string? Completion { get; set; }
    }
}
```
</details>


<details>
<summary>📚 Toggle solution</summary>

```bash
# Create a folder for your function app and navigate to it
mkdir FuncDurable
cd FuncDurable

# Create the new function app as a .NET 8 Isolated project
# No need to specify a name, the folder name will be used by default
func init --worker-runtime dotnetIsolated --target-framework net8.0

# Add the Nuget package for Storage Account to use for Functions
dotnet add package Microsoft.Azure.Functions.Worker.Extensions.Storage.Blobs --version 6.7.0

# Add the Nuget package to use Durable Functions
dotnet add package Microsoft.Azure.Functions.Worker.Extensions.DurableTask --version 1.4.0
```

Add the `Audio.cs` file with the content provided above. Then create a new file called `AudioTranscriptionOrchestration.cs` to create the orchestration of the entire Azure Function.

First, let's create the Blob Trigger function:

```csharp
using Microsoft.Azure.Functions.Worker;
using Microsoft.DurableTask;
using Microsoft.DurableTask.Client;
using Microsoft.Extensions.Logging;
using Azure.Identity;
using Azure.Storage.Blobs;
using Azure.Storage.Sas;

namespace FuncDurable
{
    public static class AudioTranscriptionOrchestration
    {
        [Function(nameof(AudioBlobUploadStart))]
        public static async Task AudioBlobUploadStart(
                [BlobTrigger("%STORAGE_ACCOUNT_CONTAINER%/{name}", Source = BlobTriggerSource.EventGrid, Connection = "STORAGE_ACCOUNT_EVENT_GRID")] Stream stream, string name,
                [DurableClient] DurableTaskClient client,
                FunctionContext executionContext)
        {
            ILogger logger = executionContext.GetLogger(nameof(AudioBlobUploadStart));

            logger.LogInformation($"Processing audio file {name}");

            var storageAccountNameUrl = Environment.GetEnvironmentVariable("STORAGE_ACCOUNT_URL") ?? throw new ArgumentNullException("STORAGE_ACCOUNT_URL");

            // Create a new Blob service client with Azure AD credentials.
            var blobServiceClient = new BlobServiceClient(new Uri(storageAccountNameUrl), new DefaultAzureCredential());
            var blobContainerClient = blobServiceClient.GetBlobContainerClient(Environment.GetEnvironmentVariable("STORAGE_ACCOUNT_CONTAINER"));
            var blobClient = blobContainerClient.GetBlobClient(name);

            var userDelegationKey = blobServiceClient.GetUserDelegationKey(DateTimeOffset.UtcNow,
                                                                            DateTimeOffset.UtcNow.AddMinutes(10));
            var sasBuilder = new BlobSasBuilder()
            {
                BlobContainerName = blobClient.BlobContainerName,
                BlobName = blobClient.Name,
                Resource = "b", // b for blob, c for container
                StartsOn = DateTimeOffset.UtcNow,
                ExpiresOn = DateTimeOffset.UtcNow.AddHours(2),
            };

            // Only read permission is necessary
            sasBuilder.SetPermissions(BlobSasPermissions.Read); 
            
            // Add the SAS token to the container URI.
            var blobUriBuilder = new BlobUriBuilder(blobClient.Uri)
            {
                Sas = sasBuilder.ToSasQueryParameters(userDelegationKey, blobServiceClient.AccountName)
            };

            var audioFile = new AudioFile
            {
                Id = Guid.NewGuid().ToString(),
                Path = blobClient.Uri.ToString(),
                UrlWithSasToken = blobUriBuilder.ToUri().ToString()
            };

            logger.LogInformation($"Processing audio file {audioFile.Id}");
        }
    }
}
```

As you can see you are using the `BlobTrigger` attribute to detect the file upload event. This attribute will trigger the function when a new blob is uploaded to the `audios` container. 

To detect when a new audio is uploaded in the Storage Account we use the Event Grid service by enabling it with the option: `BlobTriggerSource.EventGrid`.

Then we generate a SAS token to access the blob.

Let's deploy your function using the VS Code extension or by command line to the function stating with `func-drbl`.

### Event Subscription

Next step is to create an Event Grid Subscription to listen to the event of uploading in your `audios` container.

Open the resource group and find the `Event Grid System Topic` resource already created for you. Select the **Event Subscriptions** tab and create a new Event Subscription.

Give it a name and select only `Blob Created` in the `Filter to Event Types` drop down. In the `Endpoint type` section select **Web Hook**.

Then to configure the webhook you need to follow this pattern:

```bash
https://<FUNCTION_APP_NAME>.azurewebsites.net/runtime/webhooks/blobs?functionName=Host.Functions.<YOUR_FUNCTION_NAME>&code=<BLOB_EXTENSION_KEY>
```

- The `<FUNCTION_APP_NAME>` is the one starting with `func-drbl`
- The `<YOUR_FUNCTION_NAME>` is `AudioBlobUploadStart` in your case
- The `<BLOB_EXTENSION_KEY>`	is generated after you deploy your Azure Function code, it's in the **App keys** section of your Azure Durable Function (starting with `func-drbl`).

![Create Event Subscription](assets/create-event-subscription.png)

Finally, you just want to process `.wav` files, so you need to enable this in the **Filters** tab.

![Event filtering](assets/event-filtering.png)

Finally, you can click **Create**.

Try to upload a file to the audios container and you should see:

The Event Subscription starting:

![Event Grid Topic](assets/event-grid-subscription-topic.png)

And if you click on the `AudioBlobUploadStart` function and select the **Invocations** tab, you will see the Azure Durable Function starting base on this event:

![Durable Function Audio detected](assets/durable-function-audio-detected.png)

</details>

## Consume Speech to Text APIs

The Azure Cognitive Services are cloud-based AI services that give the ability to developers to quickly build intelligent apps thanks to these pre-trained models. They are available through client library SDKs in popular development languages and REST APIs.

Cognitive Services can be categorized into five main areas:

- Decision: Content Moderator provides monitoring for possible offensive, undesirable, and risky content. Anomaly Detector allows you to monitor and detect abnormalities in your time series data.
- Language: Azure Language service provides several Natural Language Processing (NLP) features to understand and analyze text.
- Speech: Speech service includes various capabilities like speech to text, text to speech, speech translation, and many more.
- Vision: The Computer Vision service provides you with access to advanced cognitive algorithms for processing images and returning information.
- Azure OpenAI Service: Powerful language models including the GPT-3, GPT-4, Codex and Embeddings model series for content generation, summarization, semantic search, and natural language to code translation.

You now want to retrieve the transcript out of the audio file uploaded thanks to the speech to text cognitive service.

<div class="task" data-title="Tasks">

> - Because the transcription can be a long process, you will use the monitor pattern of the Azure Durable Functions to call the speech to text batch API and check the status of the transcription until it's done.
>
> - Use the `SpeechToTextService.cs` file and the `Transcription.cs` model provided below to get the transcription.
> - A skeleton of the orchestration part will be provided below.
> - Instantiate an `AudioTranscription` object when the transcription is done, this will be used to store the data in Cosmos DB in the next step.
> - Do not forget to start the orchestration in the `AudioBlobUploadStart` function.

</div>

<div class="tip" data-title="Tips">

> - [What are Cognitive Services][cognitive-services]<br>
> - [Cognitive Service Getting Started][cognitive-service-api]<br> 
> - [Batch endpoint Speech to Text API][speech-to-text-batch-endpoint]<br>
> - [Monitor pattern Durable Function][monitor-pattern-durable-functions]<br>

</div>

This is the definition of the `Transcription.cs` file:

<details>
<summary>📄 Transcription.cs</summary>

```csharp
namespace FuncDurable
{
    public class TranscriptionJobFiles
    {
        public string Files { get; set; }
    }

    public class TranscriptionJob
    {
        public string Self { get; set; }

        public string Status { get; set; }

        public TranscriptionJobFiles Links { get; set; }
    }

    public class TranscriptionResultValueFile
    {
        public string ContentUrl { get; set; }
    }

    public class TranscriptionResultValue
    {
        public string Kind { get; set; }
        public TranscriptionResultValueFile Links { get; set; }
    }

    public class TranscriptionResult
    {
        public TranscriptionResultValue[] Values { get; set; }
    }

    public class Transcription
    {
        public string Display { get; set; }
    }

    public class TranscriptionDetails
    {
        public Transcription[] CombinedRecognizedPhrases { get; set; }
    }
}
```
</details>

Here is the content of the `SpeechToTextService.cs` file:

<details>
<summary>📄 SpeechToTextService.cs</summary>

```csharp
using System.Text;
using System.Text.Json;

namespace FuncDurable
{
    public static class SpeechToTextService
    {
        private static HttpClient httpClient = new()
        {
            BaseAddress = new Uri(Environment.GetEnvironmentVariable("SPEECH_TO_TEXT_ENDPOINT")!),
            DefaultRequestHeaders = { { "Ocp-Apim-Subscription-Key", Environment.GetEnvironmentVariable("SPEECH_TO_TEXT_API_KEY")! } }
        };

        public static async Task<string> CreateBatchTranscription(string audioBlobSasUri, string? id)
        {
            using StringContent jsonContent = new(
                JsonSerializer.Serialize(new
                {
                    contentUrls = new List<string> { audioBlobSasUri },
                    locale = "en-US",
                    displayName = id ?? $"My Transcription {DateTime.UtcNow.ToLongTimeString()}",
                }),
                Encoding.UTF8,
                "application/json"
            );

            HttpResponseMessage httpResponse = await httpClient.PostAsync("/speechtotext/v3.1/transcriptions", jsonContent);
            var serializedJob = await httpResponse.Content.ReadAsStringAsync();

            var options = new JsonSerializerOptions
            {
                PropertyNamingPolicy = JsonNamingPolicy.CamelCase
            };

            var job = JsonSerializer.Deserialize<TranscriptionJob>(serializedJob, options);

            if (job == null) {
                throw new Exception("Batch transcription creation failure");
            }

            return job.Self;
        }

        private static async Task<TranscriptionJob?> GetBatchTranscriptionJob(string jobUrl)
        {
            HttpResponseMessage httpResponse = await httpClient.GetAsync(jobUrl);
            var serializedJob = await httpResponse.Content.ReadAsStringAsync();

            var options = new JsonSerializerOptions
            {
                PropertyNamingPolicy = JsonNamingPolicy.CamelCase
            };

            return JsonSerializer.Deserialize<TranscriptionJob>(serializedJob, options);
        }
        
        public static async Task<string> CheckBatchTranscriptionStatus(string jobUrl)
        {
            var job = await GetBatchTranscriptionJob(jobUrl);

            return job?.Status ?? "Unknown";
        }

        public static async Task<string> GetTranscription(string jobUrl)
        {
            var job = await GetBatchTranscriptionJob(jobUrl);

            // https://learn.microsoft.com/en-us/rest/api/speechtotext/transcriptions/get?view=rest-speechtotext-v3.2-preview.2&tabs=HTTP#status
            if (job?.Status == "Failed") {
                return "";
            }

            if (job?.Status != "Succeeded") {
                throw new Exception("Batch transcription not done yet");
            }

            var files = job?.Links.Files;

            HttpResponseMessage resultsHttpResponse = await httpClient.GetAsync(files);
            var serializedJobResults = await resultsHttpResponse.Content.ReadAsStringAsync();

            var options = new JsonSerializerOptions
            {
                PropertyNamingPolicy = JsonNamingPolicy.CamelCase
            };

            var transcriptionResult = JsonSerializer.Deserialize<TranscriptionResult>(serializedJobResults, options);
            var transcriptionFileUrl = transcriptionResult?.Values.Where(value => value.Kind == "Transcription").First().Links.ContentUrl;

            if (transcriptionFileUrl == null)
            {
                throw new Exception("Transcription file url not found");
            }

            HttpResponseMessage transcriptionDetailsHttpResponse = await httpClient.GetAsync(transcriptionFileUrl);
            var serializedTranscriptionDetails = await transcriptionDetailsHttpResponse.Content.ReadAsStringAsync();
            var transcriptionDetails = JsonSerializer.Deserialize<TranscriptionDetails>(serializedTranscriptionDetails, options);
            var transcription = transcriptionDetails?.CombinedRecognizedPhrases.First().Display;

            if (transcription == null)
            {
                throw new Exception("Transcription result not found");
            }

            return transcription;
        }
    }
}
```
</details>

Below is the orchestration part of the `AudioTranscriptionOrchestration.cs` file where you will have to implement the different steps of the orchestration marked by `TODO`. Add it in the class `AudioTranscriptionOrchestration.cs` you created earlier:

<details>
<summary>📄 AudioTranscriptionOrchestration.cs</summary>

```csharp
[Function(nameof(AudioTranscriptionOrchestration))]
public static async Task RunOrchestrator(
    [OrchestrationTrigger] TaskOrchestrationContext context, 
    AudioFile audioFile)
{
    ILogger logger = context.CreateReplaySafeLogger(nameof(AudioTranscriptionOrchestration));
    if (!context.IsReplaying) { logger.LogInformation($"Processing audio file {audioFile.Id}"); }

    // Step1: TODO: Start transcription
    

    DateTime endTime = context.CurrentUtcDateTime.AddMinutes(2);

    while (context.CurrentUtcDateTime < endTime)
    {
        // Step2: TODO: Check if transcription is done
        
        if (!context.IsReplaying) { logger.LogInformation($"Status of the transcription of {audioFile.Id}: {status}"); }

        if (status == "Succeeded" || status == "Failed")
        {
            // Step3: TODO: Get transcription
            

            // Step4: TODO: Enrich the transcription

            if (!context.IsReplaying) { logger.LogInformation($"Enrich transcription of {audioFile.Id} to Cosmos DB"); }

            // Step5: TODO: Save transcription

            if (!context.IsReplaying) { logger.LogInformation($"Save transcription, finishing processing of {audioFile.Id}"); }

            break;
        }
        else
        {
            // Wait for the next checkpoint
            var nextCheckpoint = context.CurrentUtcDateTime.AddSeconds(5);
            if (!context.IsReplaying) { logger.LogInformation($"Next check for {audioFile.Id} at {nextCheckpoint}."); }

            await context.CreateTimer(nextCheckpoint, CancellationToken.None);
        }
    }
}

[Function(nameof(StartTranscription))]
public static async Task<string> StartTranscription([ActivityTrigger] AudioFile audioFile, FunctionContext executionContext)
{
    // TODO: Call the Speech To Text service to create a batch transcription
}


[Function(nameof(CheckTranscriptionStatus))]
public static async Task<string> CheckTranscriptionStatus([ActivityTrigger] AudioFile audioFile, FunctionContext executionContext)
{
    // TODO: Call the Speech To Text service to check the status of the transcription
}


[Function(nameof(GetTranscription))]
public static async Task<string?> GetTranscription([ActivityTrigger] AudioFile audioFile, FunctionContext executionContext)
{
    // TODO: Call the Speech To Text service to get the transcription
}

[Function(nameof(EnrichTranscription))]
public static AudioTranscription EnrichTranscription([ActivityTrigger] AudioTranscription audioTranscription, FunctionContext executionContext)
{
    ILogger logger = executionContext.GetLogger(nameof(EnrichTranscription));
    logger.LogInformation($"Enriching transcription {audioTranscription.Id}");      
    return audioTranscription;
}
```

</details>

<details>
<summary>📚 Toggle solution</summary>

First, you need to start the orchestration of the transcription of the audio file in the `AudioBlobUploadStart` function you did previously by adding this code at the end:

```csharp
string instanceId = await client.ScheduleNewOrchestrationInstanceAsync(nameof(AudioTranscriptionOrchestration), audioFile);

logger.LogInformation("Started orchestration with ID = '{instanceId}'.", instanceId);
```

The `ScheduleNewOrchestrationInstanceAsync` will start the orchestration of the transcription of the audio file.

Then you will need to implement the different steps of the orchestration in the `AudioTranscriptionOrchestration.cs` file.

Let's start with the `StartTranscription` function:

```csharp
ILogger logger = executionContext.GetLogger(nameof(StartTranscription));
logger.LogInformation($"Starting transcription of {audioFile.Id}");

var jobUri = await SpeechToTextService.CreateBatchTranscription(audioFile.UrlWithSasToken, audioFile.Id);

logger.LogInformation($"Job uri for {audioFile.Id}: {jobUri}");

return jobUri;
```

The goal here is to create a batch transcription using the `SpeechToTextService` and retrieve the job URI of the transcription. This job URI will be used to check the status of the transcription and get the transcription itself.

Then you will need to implement the `CheckTranscriptionStatus` function:

```csharp
ILogger logger = executionContext.GetLogger(nameof(CheckTranscriptionStatus));
logger.LogInformation($"Checking the transcription status of {audioFile.Id}");
var status = await SpeechToTextService.CheckBatchTranscriptionStatus(audioFile.JobUri!);
return status;
```

This function will check the status of the transcription using the `SpeechToTextService` and return the status.

Finally, you will need to implement the `GetTranscription` function:

```csharp
ILogger logger = executionContext.GetLogger(nameof(GetTranscription));
var transcription = await SpeechToTextService.GetTranscription(audioFile.JobUri!);
logger.LogInformation($"Transcription of {audioFile.Id}: {transcription}");
return transcription;
```

This function will get the transcription of the audio file using the `SpeechToTextService` and return the transcription.

As you probably noticed, each function use his own logger to log the different steps of the orchestration. This will help you to debug the orchestration if needed.

So far so good, you have all the functions needed to orchestrate the transcription of the audio file. The idea now is to call those functions in the orchestration part of the `AudioTranscriptionOrchestration.cs` file.

Each of those functions (`StartTranscription`, `CheckTranscriptionStatus` and `GetTranscription`) will be called in the orchestration part as an activity.

For the `Step1` you just need to call the `StartTranscription` function:

```csharp
var jobUri = await context.CallActivityAsync<string>(nameof(StartTranscription), audioFile);
audioFile.JobUri = jobUri;
```

For the `Step2` you will need to call the `CheckTranscriptionStatus` function to get the status of the transcription:

```csharp
var status = await context.CallActivityAsync<string>(nameof(CheckTranscriptionStatus), audioFile);
```

For the `Step3` you will need to call the `GetTranscription` function and create the `AudioTranscription` object to store the data in Cosmos DB in the next step:

```csharp
string transcription = await context.CallActivityAsync<string>(nameof(GetTranscription), audioFile);

if (!context.IsReplaying) { logger.LogInformation($"Retrieved transcription of {audioFile.Id}: {transcription}"); }

var audioTranscription = new AudioTranscription
{
    Id = audioFile.Id,
    Path = audioFile.Path,
    Result = transcription,
    Status = status
};
```

The  `SPEECH_TO_TEXT_ENDPOINT` and the `SPEECH_TO_TEXT_API_KEY` environment variables are already set on Azure for you, if you look at the environment variable of your function you will see that for security reason the `SPEECH_TO_TEXT_API_KEY` is referring a Key Vault where the key is.

</details>

## Store data to Cosmos DB

Azure Cosmos DB is a fully managed NoSQL database which offers Geo-redundancy and multi-region write capabilities. It currently supports NoSQL, MongoDB, Cassandra, Gremlin, Table and PostgreSQL APIs and offers a serverless option which is perfect for our use case.

You now have a transcription of your audio file, next step is to store it in a NoSQL database inside Cosmos DB.

<div class="task" data-title="Tasks">

> - Create a new `Activity Function` called `SaveTranscription` to store the transcription of the audio file in Cosmos DB.
> - Use the `CosmosDBOutput` binding to store the data in the Cosmos DB.
> - Store the `AudioTranscription` object in the Cosmos DB container called `audios_transcripts`.
> - Call the activity from the orchestration part.
> - Use manage identity to connect to Cosmos DB. 

</div>

<div class="tip" data-title="Tips">

> - [Serverless Cosmos DB][cosmos-db]<br>
> - [Cosmos DB Output Binding][cosmos-db-output-binding]

</div>

<details>
<summary>📚 Toggle solution</summary>

Because you need to connect to Azure Cosmos DB with the `CosmosDBOutput` binding you need to first add the associated Nuget Package:

```bash
dotnet add package Microsoft.Azure.Functions.Worker.Extensions.CosmosDB --version 4.12.0
```

Then to store the transcription of the audio file in Cosmos DB, you will need to create a new `Activity Function` called `SaveTranscription` in the `AudioTranscriptionOrchestration.cs` file and apply the `CosmosDBOutput` binding to store the data in the Cosmos DB:

```csharp
[Function(nameof(SaveTranscription))]
[CosmosDBOutput("%COSMOS_DB_DATABASE_NAME%",
                    "%COSMOS_DB_CONTAINER_ID%",
                    Connection = "COSMOS_DB",
                    CreateIfNotExists = true)]
public static AudioTranscription SaveTranscription([ActivityTrigger] AudioTranscription audioTranscription, FunctionContext executionContext)
{
    ILogger logger = executionContext.GetLogger(nameof(SaveTranscription));
    logger.LogInformation("Saving the audio transcription...");

    return audioTranscription;
}
```

As you can see, by just defining the binding, the Azure Function will take care of storing the data in the Cosmos DB container, so you just need to return the object you want to store, in this case, the `AudioTranscription` object.

To be able to connect the Azure Function to the Cosmos DB, you have the `COSMOS_DB_DATABASE_NAME`, the `COSMOS_DB_CONTAINER_ID` and the `COSMOS_DB` environment variables. The `COSMOS_DB` will be the connection key that will be concatenated with `__accountEndpoint` to specify the Cosmos DB account endpoint so it will be able to connect using Managed identity.

Those environment variables are already set in the Azure Function App settings (`func-drbl-<your-instance-name>`) when you deployed the infrastructure previously.

Now you just need to call the `SaveTranscription` function in the orchestration part of the `AudioTranscriptionOrchestration.cs` file:

```csharp
// Step5: Save transcription
await context.CallActivityAsync(nameof(SaveTranscription), audioTranscription);

if (!context.IsReplaying) { logger.LogInformation($"Save transcription, finishing processing of {audioFile.Id}"); }
```

</details>

#### Deployment and testing

You can now redeploy your function and upload an audio file to see if the transcription is stored in the Cosmos DB container and check the logs to see the different steps of the orchestration.

Deploy the Azure Durable Function using the same method as before but with the new function starting with `func-drbl-<your-instance-suffix-name>`.

If the deployment succeed you should see the new function in the Azure Function App:

![Azure Function App](assets/durable-function-deployed.png)

You can now validate the entire workflow : delete and upload once again the audio file. You should see the new item created above in your Cosmos DB container:

![Cosmos Db Explorer](assets/cosmos-db-explorer.png)

## Lab 2 : Summary

By now you should have a solution that :

- Invoke the execution of an Azure Durable Function responsible for retrieving the audio transcription thanks to a Speech to Text (Cognitive Service) batch processing call.
- Once the transcription is retrieved, the Azure Durable Function store this value in a Cosmos DB database.

[azure-function]: https://learn.microsoft.com/en-us/cli/azure/functionapp?view=azure-cli-latest
[azure-function-bindings-expression]: https://learn.microsoft.com/en-us/azure/azure-functions/functions-bindings-expressions-patterns
[azure-function-blob-trigger]: https://learn.microsoft.com/en-us/azure/azure-functions/functions-bindings-storage-blob-trigger?tabs=python-v2%2Cisolated-process%2Cnodejs-v4%2Cextensionv5&pivots=programming-language-csharp
[speech-to-text-batch-endpoint]: https://learn.microsoft.com/en-us/azure/ai-services/speech-service/batch-transcription-audio-data?tabs=portal
[monitor-pattern-durable-functions]: https://learn.microsoft.com/en-us/azure/azure-functions/durable/durable-functions-monitor?tabs=csharp
[cognitive-services]: https://learn.microsoft.com/en-us/azure/cognitive-services/what-are-cognitive-services
[cosmos-db-output-binding]: https://learn.microsoft.com/en-us/azure/azure-functions/functions-bindings-cosmosdb-v2-output?tabs=python-v2%2Cisolated-process%2Cnodejs-v4%2Cextensionv4&pivots=programming-language-csharp
[cognitive-service-api]: https://learn.microsoft.com/en-us/azure/ai-services/speech-service/rest-speech-to-text-short#regions-and-endpoints
[cosmos-db]: https://learn.microsoft.com/en-us/azure/cosmos-db/scripts/cli/nosql/serverless
[blob-trigger-event-grid]: https://learn.microsoft.com/en-us/azure/azure-functions/functions-event-grid-blob-trigger?pivots=programming-language-csharp

---

# Lab 3 : Monitor your Azure Functions

Let's now focus on monitoring the Azure Functions. Azure Application Insights provides a monitoring and logging solution that allows you to monitor the performance and health of your functions. You can use the Azure portal to monitor your functions, view logs, and troubleshoot issues.

In this lab you will focus on this scope:

![Hand's On Lab Architecture Lab](assets/azure-functions-lab4.png)

## Use Azure Load Testing to simulate the load

Using Azure Load Testing can help identify potential issues (e.g. errors and latency) very early and reduce the impact of these issues on your users.

Azure Load Testing is a cloud-based service provided by Azure that allows developers to simulate high volumes of user traffic on their applications. This service is designed to identify potential performance bottlenecks and ensure that applications can handle high loads, especially during peak times.

Benefits of Azure Load Testing:

- **Scalability**: Azure Load Testing can simulate thousands to millions of virtual users, allowing you to test your application under various load conditions.
- **Ease of Use**: With its intuitive interface and pre-configured test templates, Azure Load Testing makes it easy to set up and run load tests.
- **Detailed Reporting**: Azure Load Testing provides detailed reports and real-time analytics, helping you identify and resolve performance bottlenecks.
- **Cost-Effective**: With Azure Load Testing, you only pay for what you use. This makes it a cost-effective solution for load testing.

Integration with other services:

Azure Load Testing integrates seamlessly with other Azure services. For instance, it can be used in conjunction with Azure Monitor and Application Insights to provide detailed performance metrics and insights. 

## Add a new endpoint to your Azure Function App

Let's add a new endpoint to your first Azure Function App to get the audio transcriptions and use Azure Load Testing to simulate the load on this endpoint.

In the `FuncStd` add a new file called `GetTranscriptions.cs` with the following content:

<details>
<summary>📄 GetTranscriptions.cs</summary>

```csharp
using Microsoft.AspNetCore.Http;
using Microsoft.AspNetCore.Mvc;
using Microsoft.Azure.Functions.Worker;
using Microsoft.Extensions.Logging;

namespace FuncStd
{
    public class GetTranscriptions
    {
        private readonly ILogger _logger;

        public GetTranscriptions(ILoggerFactory loggerFactory)
        {
            _logger = loggerFactory.CreateLogger<GetTranscriptions>();
        }

        [Function(nameof(GetTranscriptions))]
        public IActionResult Run(
            [HttpTrigger(AuthorizationLevel.Function, "get")] HttpRequest req,
            [CosmosDBInput(
                databaseName: "%COSMOS_DB_DATABASE_NAME%",
                containerName: "%COSMOS_DB_CONTAINER_ID%",
                Connection = "COSMOS_DB",
                SqlQuery = "SELECT * FROM c ORDER BY c._ts DESC OFFSET 0 LIMIT 50")
            ] IEnumerable<Transcription> transcriptions
        )
        {
            _logger.LogInformation("C# HTTP trigger function processed a request.");

            // Simulate unexpected bahaviors
            UnexpectedBehaviors.Simulate();

            return  new JsonResult(transcriptions);
        }
    }
}
```

</details>

Then add the `Transcription.cs` model file with the following content:

<details>
<summary>📄 Transcription.cs</summary>

```csharp
namespace FuncStd
{
    public class Transcription
    {
        public string id { get; set; }
        public string path { get; set; }
        public string result { get; set; }
        public string status { get; set; }
        public string? completion { get; set; }
        public int _ts { get; set; }
    }
}
```

</details>

Finally, add the `UnexpectedBehaviors.cs` file with the following content to simulate errors and latency:

<details>
<summary>📄 UnexpectedBehaviors.cs</summary>

```csharp
namespace FuncStd
{
    public static class UnexpectedBehaviors
    {
        private static int _errorRate = 0;
        private static int _latencyInSeconds = 0;

        static UnexpectedBehaviors()
        {
            // Get the error rate from the environment variables
            if (!Int32.TryParse(Environment.GetEnvironmentVariable("ERROR_RATE"), out _errorRate))
            {
                _errorRate = 0;
            }

            // Get the extra injected latency from the environment variables
            if (!Int32.TryParse(Environment.GetEnvironmentVariable("LATENCY_IN_SECONDS"), out _latencyInSeconds))
            {
                _latencyInSeconds = 0;
            }
        }

        public static void Simulate()
        {
            // Simulating latency: sleep for _latencyInSeconds seconds
            if (_latencyInSeconds != 0) {
                Thread.Sleep(_latencyInSeconds * 1000);
            }

            // Simulating errors: throw errors with a probability of _errorRate
            if (_errorRate != 0 && Random.Shared.Next(0, 100) < _errorRate) {
                throw new Exception("Simulated error!");
            }
        }
    }
}
```

</details>

Finally, add the `Microsoft.Azure.Functions.Worker.Extensions.CosmosDB` Nuget package to your project by running the following command:

```sh
dotnet add package Microsoft.Azure.Functions.Worker.Extensions.CosmosDB --version 4.12.0
```

Redeploy the `func-std` Function App to add this new endpoint.

If the deployment succeed you should see the new `GetTranscriptions` function in the Azure Function App:

![New endpoint](assets/monitoring-new-endpoint.png)

If you look at the environment variables of the `func-std` Function App, you will see that the `TranscriptionsDatabase__accountEndpoint` are already set, this is the setup needed to connect to the Cosmos DB. Your Azure Function has also the role of `Cosmos DB Built-in Data Reader` to be able to read the data from the Cosmos DB. Remember, it's the same approach that you used in the Azure Durable Function in the previous lab.

## Create a load test for the new endpoint

<div class="task" data-title="Task">

> - Create a Load test for the `GetTranscriptions` endpoint
> - Limit the duration of the test to **3 minutes**

</div>

<div class="tip" data-title="Tips">

> - Use the direct integration of [Azure Load Testing with Azure Functions][azure-load-testing-setup]

</div>

<details>

<summary>📚 Toggle solution</summary>

1. Locate the Function App in the Azure Portal which start with `func-std`
1. Click on the `Load Testing (Preview)` blade
1. Click on the `Create test` button in the middle of the page
1. In the `Test plan` tab
1. Select the existing Azure Load Testing resource inside your resource group and provide a short name and description of the test:

![Create test](assets/monitoring-create-test.png)

1. Click on `Add request`
1. Make sure to select the `GetTranscriptions` endpoints and uses the right HTTP method (`GET`)
1. Validate the request using the `Add` button

![Add request](assets/monitoring-get-transcriptions-requests.png)


1. Select the tab `Load` and set `Test duration (minutes)` to 3
1. Click on `Review + create` then on `Create`
1. The test will take few seconds to get created and then you should see a popup telling you that the test has started.

</details>

As the test starts, you will see a `Load test results` dashboard with various metrics like the total number of requests, throughput, and error percentage.

<div class="task" data-title="Task">

> - Find out the average response time of the `GetTranscriptions` endpoint

</div>

<details>

<summary>📚 Toggle solution</summary>

1. Locate the `Aggregation` filter in the `Client-side metrics` panel
1. Uncheck existing selection, select `Average`, then click on `Apply`
1. Locate the metric below the graph in `Response time (successful responses)`. That is the average response time.

![Average response time](assets/monitoring-average-time-when-succeeded.png)

As you can see the response time average is 113ms.

</details>

You know have a way to monitor the performance of your Azure Function and identify potential issues before they impact your users.

## Simulate errors

To simulate errors, you can see that in the `GetTranscriptions.cs` file you have a call to the `UnexpectedBehaviors.Simulate()` method which will throw an exception randomly.

If you open the `UnexpectedBehaviors.cs` file you will see that you have 2 environments variables `ERROR_RATE` and `LATENCY_IN_SECONDS` which are used to set the error rate and latency.

Those environment variables are already set in the Azure Function App settings (`func-std-<your-instance-name>`) when you deployed the infrastructure previously.

Let's start by playing with the `ERROR_RATE` environment variable to simulate errors. Go to your Azure Function instance (`func-std-<your-instance-name>`), inside **Settings** > **Environment variables** > **App Settings** and set the `ERROR_RATE` to `50` to simulate 50% of errors.

![Set Error Rate](assets/azure-function-app-error-rate.png)

Go back to the **Load Testing** menu and re-run to the end of the test to see the new result.

<div class="task" data-title="Task">

> - Use Application Insights to find more details about the error triggered by the load test

</div>

<details>

<summary>📚 Toggle solution</summary>

1. Go to your Azure Function App starting with `func-std`
1. Locate the **Application Insights** tab inside **Settings** and on the right click on **View Application Insights data**.
1. You should see a spike of errors on the `Failed requests` panel:

![Failed requests](assets/monitoring-spike-of-requests.png)

Then, click on the `Failures` tab on the left and click on the top response code (`500`) on the right panel. Select the suggested `GetTranscriptions` operation on the right panel

![Failure overview](assets/monitoring-app-insights-failure-overview.png)

You should see the details of the errors including any other service which was involved on the operation.
Select the exception to access its call stack and check the `message` on the right panel and click on `[show more]` to get more details

![Failure details](assets/monitoring-app-insights-detail.png)

You should see a reference to `UnexpectedBehaviors.Simulate()`, in the `UnexpectedBehaviors.cs` file:

![Failure stack](assets/monitoring-app-insights-exception-detail.png)

Based on those metrics you can identify the root cause of the error and fix it.

</details>

## Simulate latency

Now let's go back to the **App Settings** of your `func-std` instance and reset the `ERROR_RATE` environment variable to `0` to disable the errors and simulate latency. Set the `LATENCY_IN_SECONDS` to `3` to simulate a latency of 3 seconds.

Then run the load test again and wait for the end of the test.

<div class="task" data-title="Task">

> - Use Application Insights to find more details about the latency issue that you are simulating.

</div>

<details>

<summary>📚 Toggle solution</summary>

1. Go to your Azure Function App starting with `func-std`
1. Locate the **Application Insights** tab inside **Settings** and on the right click on **View Application Insights data**.
1. You should see an increase in response time on the `Server response time` panel:

![Increase time](assets/monitoring-response-time-increase.png)

1. Use the `Performance` blade or click on the response time chart
1. You should see the duration taken by each operation
1. Zoom into a range where the latency was injected, you should see the `GetTransactions` operation be unnaturally slow (+3 seconds) and you should see a red arrow on the right of the operation together with the percentage of increase in latency. This is the endpoint to investigate.

![Latency details](assets/monitoring-get-transcriptions-performances.png)

</details>

[azure-load-testing-setup]:  https://learn.microsoft.com/en-us/azure/load-testing/how-to-create-load-test-function-app

You can now reset the `LATENCY_IN_SECONDS` to `0` to continue the next lab.

## Lab 3 : Summary

As you can see, Azure Load Testing is a powerful tool that allows you to simulate high volumes of user traffic on your applications and identify potential performance bottlenecks or detect issues in your implementation. By using Azure Load Testing, you can ensure that your applications can handle high loads and provide a seamless user experience.

---

# Lab 4 : Integration with Azure API Management

Let's now integrate the Azure Functions with Azure API Management (APIM) to expose the transcription of the audio file as an API. 

Previously to test your Azure Function you had to get the Function URL with the *default (function key)* to ensure a basic security layer. But in a real-world scenario, you will need to secure your Azure Function and expose it through an API Gateway like Azure API Management.

In fact, with Azure API Management you can expose your Azure Functions as APIs and manage them with policies like authentication, rate limiting, caching, etc. You can manage who can call your Azure Function by providing a subscription key or using OAuth 2.0 authentication.

In this lab you will see how to expose the Azure Function as an API using Azure API Management:

![Hand's On Lab Architecture Lab](assets/azure-functions-lab5.png)

## Define the API in APIM

### Import the Azure Function

Inside your resource group, you should see an APIM instance. Click on it, you will be redirected to the APIM instance overview.

<div class="task" data-title="Tasks">

> - Define the Azure Function (`func-std-`) endpoints as an API in Azure API Management.

</div>

<div class="tip" data-title="Tips">

> - [Import Azure Function Azure API Management][import-azure-function-azure-api-management]<br>

</div>

<details>
<summary>📚 Toggle solution</summary>

First, go to the **APIs** section in the left menu and click on the **+ Add API** button.

Then select the **Function App** option:

![Select Function App](assets/apim-import-function.png)

In the popup menu to create a Function select **Browse**, this will redirect you to the list of all Azure Functions in your Subscription. Click on the **Select** button and pick the Azure Function which is responsible for the transcription of the audio file. It should start by `func-std-`.

![Select Function](assets/apim-select-function-app.png)

Then you will see automatically the list of endpoints, select the **AudioUpload** and the **GetTranscriptions** endpoints and click on the **Select** button.

This will fill all the information needed to create the API in APIM, let's update the API details to have something more meaningful.

- **Display name**: `Audio Transcription API`
- **Name**: `audio-transcription-api`
- **API Url Suffix**: `audios-transcriptions`

![API Details](assets/apim-import-function-detail-form.png)

 You can now click on the **Create** button.

 You should see the new API in the list of APIs in your APIM instance.

</details>

### What happened behind the scenes?

If you navigate to the **Backends** section of your APIM you should see a line pointing to the Azure Function App you just imported:

![APIM Backend](assets/apim-backends-list.png)

This is a declaration of the Azure Function App as a backend in APIM. This will allow you to call the Azure Function from the API Gateway.

Now, inside the **Named values** section of the APIM you should see a line which represent the storage of the Azure Function **Host keys** to authorize the APIM to call the Azure Function:

![APIM Named Values](assets/apim-named-value-key.png)

This key was created automatically by Azure, if you go in your Azure Function, inside **Functions** > **App keys** you will see an access given to the APIM instance:

![Function App Key](assets/apim-azure-function-host-keys.png)

So now the APIM instance can call the Azure Function with this key hidden for the user.

When defining an API on APIM you can protect it using different methods like OAuth 2.0, Subscription keys, etc.

If you go to your definition of the API in APIM, in the **Settings** tab you will see the **Subscription required** option. This option allows you to protect your API with a subscription key which should be passed in the header or in the query string of the request:

![APIM Subscription Key](assets/apim-api-settings.png)

You can specify the **Subscription key header name** and the **Subscription key query parameter name** to define how the subscription key should be passed in the request.

## Call your API

Let's test this in Postman to see how it works. Open the Postman application and copy/paste the URL from your API in APIM. It should be inside the **Test** tab:

![APIM Test URL](assets/apim-test-tab-url.png)

In Postman, create a new request and paste the URL in the URL field. Select the **Body** tab and select **form-data** as the type of the body. Then define `audios` as a key and select your audio file to upload.

Run the request, and you should see a `401` status code because you need to add the subscription key in the header of the request. In fact the APIM give you the possibility to protect your API with a subscription key.

To do so, go back to your APIM instance and select the **Subscriptions** tab and click on the **+ Add Subscription** button to create a new subscription key dedicated to your API:

![APIM Add Subscription](assets/apim-create-api-subscription-key.png)

Then you can copy the subscription key and add it in the header of your request in Postman in the `Ocp-Apim-Subscription-Key` key. Run the request again, and you should see a `200` status code:

![Postman Test](assets/apim-postman-sucess-result.png)


## Lab 4 : Summary

At the end of this lab you should have an Azure Function exposed as an API via Azure API Management. You should be able to call this API with a subscription key to upload an audio file to the storage account.

[import-azure-function-azure-api-management]: https://learn.microsoft.com/en-us/azure/api-management/import-function-app-as-api

---

# Lab 5 : Use Azure Functions with Azure OpenAI

In this lab you will use Azure Functions to call the Azure OpenAI service to analyze the transcription of the audio file and add some information to the Cosmos DB entry.

You will go back to the Azure Durable Function you did in the previous lab and add a connection to Azure OpenAI to be able to summarize the transcription you saved.

So the scope of the lab is this one:

![Hand's On Lab Architecture Lab](assets/azure-functions-lab6.png)

## Setup for Azure OpenAI

### Add the .NET package

Inside your `FuncDurable` let's add the following version of `Microsoft.Azure.Functions.Worker.Extensions.OpenAI`:

```bash
dotnet add package Microsoft.Azure.Functions.Worker.Extensions.OpenAI --version 0.19.0-alpha
```

### Add environment variables

To be able to connect the Azure Function to the Azure OpenAI service, you will need to set the `AZURE_OPENAI_ENDPOINT` and the `CHAT_MODEL_DEPLOYMENT_NAME` environment variable. Those variables were already deployed for you using the infrastructure as code. 

### Add the role to the Azure Function App

The Azure Function App will also need the role of `Cognitive Services OpenAI User` to be able to call the Azure OpenAI service using managed identities.

This role was already assigned to your Azure Function when it was provisioned at the beginning of the lab.

<div class="task" data-title="Tasks">

> - Ensure the Durable function app has the required permissions to consume Azure OpenAI

</div>

<details>
<summary>📚 Toggle solution</summary>

You can confirm this by going to your Azure OpenAI service and in the **Access control (IAM)** section click on the **Role assignment** table and locate the `Cognitive Services OpenAI User` role (you can directly filter assignments by role).

You should see that the Durable function's Function App has the role `Cognitive Services OpenAI User`.

</details>


## Enrich the transcription with Azure OpenAI

<div class="task" data-title="Tasks">

> - Update the Activity function `EnrichTranscription` inside the `AudioTranscriptionOrchestration.cs` to call Azure OpenAI via `TextCompletionInput`
> - Use the result to update the `Completion` field of the transcription.

</div>

<details>
<summary>📚 Toggle solution</summary>

First, you need to add the `TextCompletionInput` binding to the `EnrichTranscription` method:

```csharp
[Function(nameof(EnrichTranscription))]
public static AudioTranscription EnrichTranscription(
    [ActivityTrigger] AudioTranscription audioTranscription, FunctionContext executionContext,
    [TextCompletionInput("Summarize {Result}", ChatModel = "%CHAT_MODEL_DEPLOYMENT_NAME%")] TextCompletionResponse response
)
```

Make sure to add the following `using` to be able to use the `TextCompletionInput` attribute:

```csharp
using Microsoft.Azure.Functions.Worker.Extensions.OpenAI.TextCompletion;
```

This will manage for you the authentication to the Azure OpenAI service and send the transcription to the service to get a summary of the transcription.

Then you just have to consume the `Content` property of the response object and update the `Completion` field of the `AudioTranscription` object:

```csharp
audioTranscription.Completion = response.Content;
```

And that's it, you have now enriched the transcription of the audio file with the Azure OpenAI service!

So, to summarize, the function will look like this:

```csharp
[Function(nameof(EnrichTranscription))]
public static AudioTranscription EnrichTranscription(
    [ActivityTrigger] AudioTranscription audioTranscription, FunctionContext executionContext,
    [TextCompletionInput("Summarize {Result}", ChatModel = "%CHAT_MODEL_DEPLOYMENT_NAME%")] TextCompletionResponse response
)
{
    ILogger logger = executionContext.GetLogger(nameof(EnrichTranscription));
    logger.LogInformation($"Enriching transcription {audioTranscription.Id}");
    audioTranscription.Completion = response.Content;
    return audioTranscription;
}
```

Finally, update the step 4 and 5 from the `RunOrchestrator` method in the `AudioTranscriptionOrchestration.cs` to trigger the activity of enrichment of the transcription before saving the result to the Cosmos DB:

```csharp
// Step4: Enrich the transcription
AudioTranscription enrichedTranscription = await context.CallActivityAsync<AudioTranscription>(nameof(EnrichTranscription), audioTranscription);

if (!context.IsReplaying) { logger.LogInformation($"Saving transcription of {audioFile.Id} to Cosmos DB"); }

// Step5: Save transcription
await context.CallActivityAsync(nameof(SaveTranscription), enrichedTranscription);

if (!context.IsReplaying) { logger.LogInformation($"Finished processing of {audioFile.Id}"); }
```

</details>

## Deployment and testing

Deploy the Azure Durable Function using the same method as before in the Azure Function App starting with `func-drbl-<your-instance-suffix-name>`.

You will see a new property in your Cosmos DB item called `completion` with a summary of the audio made by Azure OpenAI:

![Open AI Summary Result](assets/open-ai-summary-result.png)

## Lab 5 : Summary

You saw how easy it is to integrate Azure OpenAI with Azure Function to enrich your items inside Cosmos DB. You have now a full scenario with your Azure Durable Function!

---

# Closing the workshop

Once you're done with this lab you can delete the resource group you created at the beginning.

To do so, click on `delete resource group` in the Azure Portal to delete all the resources and audio content at once. The following Az-Cli command can also be used to delete the resource group :

```bash
# Delete the resource group with all the resources
az group delete --name <resource-group>
```
