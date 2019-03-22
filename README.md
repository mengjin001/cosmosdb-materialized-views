# Near-Real Time Updated Materialized View With Cosmos DB and Azure Functions

WIP: descrive the use case scenario and the solution

Sample data:

    {
        "deviceId": "036",
        "value": 164.91290226807487,
        "timestamp": "2019-03-22T19:46:20.8633068Z"
    }

Sample Materialized Views. This one is for each device and contain aggregated data for the device specified in `id`:

    {
        "id": "030",
        "aggregationSum": 3519.8782286699293,
        "lastValue": 155.41897977488998,
        "type": "device",
        "deviceId": "030",
        "lastUpdate": "2019-03-22T19:50:17Z"
    }

There is also a `global` materialized view, where the last value *for each device* is stored:

    {
        "id": "global",
        "deviceId": "global",
        "type": "global",
        "deviceSummary": {
            "035": 104.3423159533843,
            "016": 129.1018793494915,
            ...
            "023": 177.62450146378228,
            "033": 178.97744880941576
        },
    }

Values are updated in near-real time by using the Change Feed feature provided by Cosmos DB. The sample is using the default feed polling time of 5 seconds, but it can easily changed to a much lower value if you need more "real time" updates.

- https://docs.microsoft.com/en-us/azure/cosmos-db/change-feed-functions 

- https://docs.microsoft.com/en-us/azure/azure-functions/functions-bindings-cosmosdb-v2#trigger---c-attributes 

## Prerequisites

If you don't have an Azure subscription, create a [free account](https://azure.microsoft.com/free/?ref=microsoft.com&utm_source=microsoft.com&utm_medium=docs&utm_campaign=visualstudio) before you begin.

In addition:

* [Visual Studio 2017](https://visualstudio.microsoft.com/downloads/) or  [Visual Studio Code](https://code.visualstudio.com/)
* [.NET Core SDK](https://dotnet.microsoft.com/download)
* [Git](https://www.git-scm.com/downloads)
* [Azure CLI](https://docs.microsoft.com/en-us/cli/azure/install-azure-cli)
* [Azure Storage Explorer](https://azure.microsoft.com/en-us/features/storage-explorer/)

## Getting Started

Make sure you have WSL (Windows System For Linux) installed and have AZ CLI version > 2.0.50. Before running any script also make sure you are authenticated on AZ CLI using

```bash
az login
```

and have selected the Azure Subscription you want to use for the tests:

```bash
az account list --output table
az account set --subscription "<YOUR SUBSCRIPTION NAME>"
```

## Clone the sample project

Clone the repository and open the code-samples directory from your command line tool.

```bash
git clone https://github.com/Azure-Samples/cosmosdb-materialized-views
cd cosmosdb-materialized-views
```

## Create Azure Resources

To create and configure the Azure Resources needed for the project, you just have to run the `deploy.sh` script in the `script` folder.

Script has been tested on Mac OSX and Ubuntu Bash.

    ./script/deploy

The following resources will be created:

- Resource Group
- Azure Storage
- Azure Function (using Consumption Plan)
- Application Insight
- Cosmos DB with 2 Collections (1000 RU/s each)

## Run the Producer application

The producer application will generate sample sensor data as described before. The application takes the device ids to generate as parameter:

    cd sensor-data-producer
    dotnet run 1

The above sample will generate random data for DeviceId 001. If you want to generate more data just specify how many sensor you need to be simulated:

    dotnet run 10

will generate data for 10 sensors, from 001 to 010. If you want to generate more workload and you're planning to distribute the work on different clients (using Azure Container Instances or Azure Kubernetes Service for example), you'll find useful the ability to specify which device id range you want to be generated by each running application. For example

    dotnet run 15-25

will generate data with Device Ids starting from 015 up to 025.

## Check results

Once the producer is stared you can see the result by using Azure Portal or Azure Storage Explorer to look for document create int the `view` collection of the created Cosmos DB database.

You can also take a look at the Application Insight Live Metric Streams to see in real time function processing incoming data from the Change Feed