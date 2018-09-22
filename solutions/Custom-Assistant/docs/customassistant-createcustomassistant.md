# Virtual Assistant Creation

## Overview

> [!NOTE]
> This topics applies to v4 version of the SDK.

The Virtual Assistant Solution provides everything you need to get started with building your own Assistant. Base Assistant capabilities are provided within the solution including language models for you to build upon along with Conversational Skill support enabling you to plug-in additional capabilities through configuration. See the [Overview](./readme.md) for more information.

The Virtual Assistant solution is under ongoing development within an open-source GitHub repo enabling you to participate with our ongoing work.

The Virtual Assistant Solution is available for .NET, targeting **V4** versions of the SDK.

### Prerequisites
- Install the Azure Bot Service command line (CLI) tools. It's important to do this even if you have earlier versions as the Virtual Assistant makes use of new deployment capabilities.

```shell
npm install -g botdispatch chatdown ludown luis-apis luisgen msbot qnamaker  
```
- Install the Azure Command Line Tools (CLI) from [here](https://docs.microsoft.com/en-us/cli/azure/install-azure-cli-windows?view=azure-cli-latest)

- Install the Az Extension for Bot Service
```shell
az extension add -n botservice
```

### Clone the Repo

The first step is to clone the [Microsoft Conversational AI GitHub Repo](https://github.com/Microsoft/AI). You'll find the Virtual Assistant solution within the `solutions\Custom-Assistant` folder.

Once the Solution has been cloned you will see the following folder structure.

    | - Assistant
        | - LinkedAccounts.Web
        | - Skills
            | - CalendarSkill
            | - DemoSkill
            | - EmailSkill
            | - PointofInterestSkill
            | - ToDoSkill
            | - Test
        | - TestHarnesses
            | - Assistant-ConsoleDirectLineSample
            | - Assistant-WebTest
        | - Microsoft.Bot.Solutions
    | - CustomAssistant.sln
    | - Skills.sln

### Build the Solution

Once cloned the next step it to build the CustomAssistant and Skills solutions within Visual Studio. The Virtual Assistant and Skills solutions are separated for clarity with the Skills solution pushing binaries into a shared output folder enabling easy referencing from the Virtual Assistant.

> Build the Skills project before the Virtual Assistant project.

### Deployment

The Virtual Assistant require the following dependencies for end to end operation.
- Azure Web App
- Azure Storage Account (Transcripts)
- Azure Application Insights (Telemetry)
- Azure CosmosDb (State)
- Azure Cognitive Services - Language Understanding
- Azure Cognitive Services - QnAMaker (including Azure Search, Azure Web App)
- Azure Cognitive Services - Content Moderator (optional manual step)

> Review the pricing and terms for the services and adjust to suit your scenario

If you have multiple Azure subscriptions and want to ensure the deployment selects the correct one, run the following commands before continuing.

```shell
az login
az account list
az account set --subscription "YOUR_SUBSCRIPTION_NAME"
```

Your Virtual Assistant project has a deployment recipe enabling the `msbot clone services` command to automate deployment of all the above services into your Azure subscription and ensure the .bot file in your project is updated with all of the services including keys enabling seamless operation of your Virtual Assistant.

To deploy your Virtual Assistant including all dependencies - e.g. CosmosDb, Application Insights, etc. run the following command from a command prompt within your project folder. Ensure you update the authoring key from the previous step and choose the Azure datacenter location you wish to use.

```shell
msbot clone services --name "MyCustomAssistantName" --luisAuthoringKey "YOUR_AUTHORING_KEY" --folder "DeploymentScripts\msbotClone" --location "westus"
```

The msbot tool will outline the deployment plan including location and SKU. Ensure you review before proceeding.

![Deployment Confirmation](./media/customassistant-deploymentplan.png)

>After deployment is complete, it's **imperative** that you make a note of the .bot file secret provided as this will be required for later steps.

- Update your `appsettings.json` file with the newly created .bot file name and .bot file secret.
- Run the following command and retrieve the InstrumentationKey for your Application Insights instance and update InstrumentationKey in your `appsettings.json` file.

`msbot list --bot YOURBOTFILE.bot --secret YOUR_BOT_SECRET`

        {
          "botFilePath": ".\\YOURBOTFILE.bot",
          "botFileSecret": "YOUR_BOT_SECRET",
          "ApplicationInsights": {
            "InstrumentationKey": "YOUR_INSTRUMENTATION_KEY"
          }
        }

## Skill Configuration

The Virtual Assistant Solution is fully integrated with all available skills out of the box but you need to provide updates to the Skill configuration to reflect the LUIS models required for each skill.

> An automated script leveraging the configuration held in your .bot file is planned for a future release which will automate this step.

For each of the Skills (Calendar, Email, ToDo and PoIntOfInterest) run the following command line to retrieve the configuration information. You can make use of `msbot list` instead if you prefer.

> Note the BotName and SKillName are case sensitive, please ensure you use the correct capitalisation for your Bot name and each Skill.

```shell
msbot get YourBotName_Calendar --bot YOURBOTNAME.bot --secret YOUR_BOT_SECRET
```

Then in the corresponding entry in `appSettings.config` update the `LuisAppId`, `LuisSubscriptionKey` and `LuisEndPoint`.
```
"Skills": [
    {
      "Name": "Calendar",
      "DispatcherModelName": "l_Calendar",
      "Description": "The Calendar Skill adds Calendar related capabilities to your Bot.",
      "Assembly": "CalendarSkill.CalendarSkill, CalendarSkill, Version=1.0.0.0, Culture=neutral",
      "AuthConnectionName": "",
      "Parameters": [
        "IPA.Timezone"
      ],
      "Configuration": {
        "LuisAppId": "",
        "LuisSubscriptionKey": "",
        "LuisEndpoint": ""
      }
    },
```

## Skill Authentication

If you wish to make use of the Calendar, Email and Task Skills.
The final step is to 

## Testing
Once deployment is complete, run your bot project within your development envrironment and open the [Bot Framework Emulator](https://github.com/Microsoft/BotFramework-Emulator). Within the Emulator, choose Open Bot from the File menu and navigate to the .bot file in your directory which was created in the previous step.

You should see an Introduction Adaptive card and the example on-boarding will start. 

See the [Testing](./customassistant-testing.md) section for information on how to test your Virtual Assistant.

> Note that the Deployment will deploy your Virtual Assistant but will not configure Skills. These are an optional step which is documented [here](./customassistant-addingskills.md).