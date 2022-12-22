## MemesFinderBot

### CI\CD tatuses
Infrastructure: [![Build Status](https://dev.azure.com/VostokEngineering/MemesFinder/_apis/build/status/ArtemKiyashko.memesfinder-infrastructure?branchName=master)](https://dev.azure.com/VostokEngineering/MemesFinder/_build/latest?definitionId=9&branchName=master)

Gateway: [![Build Status](https://dev.azure.com/VostokEngineering/MemesFinder/_apis/build/status/ArtemKiyashko.memesfinder-gateway?branchName=master)](https://dev.azure.com/VostokEngineering/MemesFinder/_build/latest?definitionId=8&branchName=master)

TextProcessor: [![Build Status](https://dev.azure.com/VostokEngineering/MemesFinder/_apis/build/status/ArtemKiyashko.memesfinder-gateway?branchName=master)](https://dev.azure.com/VostokEngineering/MemesFinder/_build/latest?definitionId=8&branchName=master)

ProcessMeme: [![Build Status](https://dev.azure.com/VostokEngineering/MemesFinder/_apis/build/status/ArtemKiyashko.memesfinder-processmeme?branchName=master)](https://dev.azure.com/VostokEngineering/MemesFinder/_build/latest?definitionId=11&branchName=master)

### Description

This telegram bot is basically listening for the group messages, randomly chosing one of and replying with the picture, related to the one of the key phrase from the original message.

As a key phrase extractor we are using [Azure Text Analytics](https://azure.microsoft.com/en-us/products/cognitive-services/text-analytics/#overview) service (part of Cognitive Services)

For picture search we are utilizing [Google Custom Search](https://developers.google.com/custom-search/v1/introduction) and it`s [.NET Client Library](https://developers.google.com/api-client-library/dotnet/apis/customsearch/v1)

Communication with Telegram through [Telegra.Bot](https://github.com/TelegramBots/Telegram.Bot) library for .NET

This project using only SaaS model of Azure resources (nothing for IaaS/PaaS or On-Premise)

## Services repositories

[Gateway](https://github.com/ArtemKiyashko/memesfinder-gateway) - receiving Telegram HTTP updates with messages from chat. This service takes the responsibility of taking the decision for processing particular message. If positive dicision taken - just forwarding message to ServiceBus topic

[TextProcessor](https://github.com/ArtemKiyashko/memesfinder-textprocessor) - finding key phrases in original message via Azure Congnitive Services

[ProcessMeme](https://github.com/ArtemKiyashko/memesfinder-processmeme) - finding proper picture based on the key phrase received from `TextProcessor`. If picture found - replying to original Telegram message with that picture.

## Architecture

Stateless microservices based on Azure Function Apps with consumption service plan (can be changed in `parameters.json` file).

Azure Service Bus as event bus and message broker for communication between services.

### Data flow:

```mermaid
flowchart LR;

A(TG HTTP IN);
B(FA: Gateway);
C(SB: alltgmessages)
D(FA: TextProcessor)
E(SB: keywordmessages)
F(FA: ProcessMeme)
Z(TG HTTP OUT)

A -- Update --> B
B -- Update --> C
C -- Update --> D
D -- UpdateWithKeyword --> E
E -- UpdateWithKeyword --> F
F -- PhotoMessage --> Z
```

Legend:
 - `FA` - Azure Function App
 - `SB` - ServiceBus topic
 - `Update` - incoming Telegram message from chat
 - `UpdateWithKeyword` - composite model including original Telegram message model and key phrase

### Resources to be created by this ARM template

![azure_resources](img/rsgweprivatememesfinder.png)