## MemesFinderBot

### CI\CD statuses
Infrastructure: [![Build Status](https://dev.azure.com/VostokEngineering/MemesFinder/_apis/build/status/ArtemKiyashko.memesfinder-infrastructure?branchName=master)](https://dev.azure.com/VostokEngineering/MemesFinder/_build/latest?definitionId=9&branchName=master)

Gateway: [![Build Status](https://dev.azure.com/VostokEngineering/MemesFinder/_apis/build/status/ArtemKiyashko.memesfinder-gateway?branchName=master)](https://dev.azure.com/VostokEngineering/MemesFinder/_build/latest?definitionId=8&branchName=master)

Orchestrator: in-progress

Greeter: [![Build Status](https://dev.azure.com/VostokEngineering/MemesFinder/_apis/build/status/memesfinder-greeter?branchName=master)](https://dev.azure.com/VostokEngineering/MemesFinder/_build/latest?definitionId=13&branchName=master)

DecisionMaker: [![Build Status](https://dev.azure.com/VostokEngineering/MemesFinder/_apis/build/status/memesfinder-decisionmaker?branchName=main)](https://dev.azure.com/VostokEngineering/MemesFinder/_build/latest?definitionId=12&branchName=main)

TextProcessor: [![Build Status](https://dev.azure.com/VostokEngineering/MemesFinder/_apis/build/status/ArtemKiyashko.memesfinder-gateway?branchName=master)](https://dev.azure.com/VostokEngineering/MemesFinder/_build/latest?definitionId=8&branchName=master)

ProcessMeme: [![Build Status](https://dev.azure.com/VostokEngineering/MemesFinder/_apis/build/status/ArtemKiyashko.memesfinder-processmeme?branchName=master)](https://dev.azure.com/VostokEngineering/MemesFinder/_build/latest?definitionId=11&branchName=master)

### Description

This telegram bot is basically listening for the group messages, randomly choosing one of and replying with the picture, related to the one of the key phrase from the original message. Optionally this bot can reply to dedicated request if trained AI model (Azure Cognitive Services/LUIS/etc) recognise specific pattern in income message.

As a key phrase extractor and request entity extractor we are using [Azure Cognitive Services](https://azure.microsoft.com/en-us/products/cognitive-services)

For picture search we are utilizing [Google Custom Search](https://developers.google.com/custom-search/v1/introduction) and it`s [.NET Client Library](https://developers.google.com/api-client-library/dotnet/apis/customsearch/v1)

Communication with Telegram through [Telegram.Bot](https://github.com/TelegramBots/Telegram.Bot) library for .NET

This project using only SaaS model of Azure resources (nothing for IaaS/PaaS or On-Premise)

## Services repositories

[Gateway](https://github.com/ArtemKiyashko/memesfinder-gateway) - receiving Telegram HTTP updates with messages from chat. 

[Orchestrator](https://github.com/ArtemKiyashko/memesfinder-messageorchestrator) - decides whenever it's a general message and should be processed through the DecisionMaker or dedicated meme request and should response immediately.

[DecisionMaker](https://github.com/ArtemKiyashko/memesfinder-decisionmaker) - this service takes the responsibility of taking the decision for processing particular message. If positive dicision taken - just forwarding message to ServiceBus topic.

[TextProcessor](https://github.com/ArtemKiyashko/memesfinder-textprocessor) - finding key phrases in original message via Azure Congnitive Services.

[ProcessMeme](https://github.com/ArtemKiyashko/memesfinder-processmeme) - finding proper picture based on the key phrase received from `TextProcessor`. If picture found - replying to original Telegram message with that picture.

[Greeter](https://github.com/ArtemKiyashko/memesfinder-greeter) - sending welcome message to new chat memebrs

## Architecture

Stateless microservices based on Azure Function Apps with consumption service plan (can be changed in `parameters.json` file).

Azure Service Bus as event bus and message broker for communication between services.

### Data flow:

```mermaid
flowchart TD;

A(TG HTTP IN);
B(FA: Gateway);
C(SB: allmessages);
G(FA: DecisionMaker);
H(SB: textmessages);
D(FA: TextProcessor);
E(SB: keywordmessages);
F(FA: ProcessMeme);
I(FA: Orchestrator);
J(FA: Greeter);
L(SB: generalmessages);
Z(TG HTTP OUT);

A -- Update --> B
B -- Update --> C
C -- Update --> I
I -- MessageWithKeyword --> E
J -- TextMessage --> Z 
I -- Update --> L
L -- Update --> G
G -- Update --> H
H -- Update --> D
D -- MessageWithKeyword --> E
E -- MessageWithKeyword --> F
F -- PhotoMessage --> Z
C -- Update --> J
```

Legend:
 - `FA` - Azure Function App
 - `SB` - ServiceBus topic
 - `Update` - incoming Telegram message from chat
 - `MessageWithKeyword` - composite model including original Telegram message model and key phrase

### Resources to be created by this ARM template

![azure_resources](img/testgroup.png)