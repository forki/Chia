# Chia - HelperFunctions for reporting

[![Build Status](https://dev.azure.com/DanpowerEnergyData/Chia/_apis/build/status/DanpowerGruppe.Chia?branchName=master)](https://dev.azure.com/DanpowerEnergyData/Chia/_build/latest?definitionId=4&branchName=master)

[![nuget - Chia](https://img.shields.io/nuget/v/Chia.svg?colorB=green)](https://www.nuget.org/packages/Chia/)

[![nuget - ChiaClient](https://img.shields.io/nuget/v/Chia.Client.svg?colorB=green)](https://www.nuget.org/packages/Chia.Client/)

Chia contains HelperFunctions for reporting. Chia contains some Azure Storage functions, logging features and some excel utils.

## FileWriter

Initialize your FileWriter instant with `initFileWriter`:

```fs
open Chia.Domain.Logging
open Chia.Domain.Config
open Chia.FileWriter
let devStatus = Development
let fileWriterInfo = initFileWriter devStatus "ProjectName" Local
let fileWriterInfoAzure = initFileWriter devStatus "PIvy" Azure
```

You can now use an `logOk` and `logError` like this:

```fs
logOk fileWriterInfo ("Something went ok!")
```

```fs
try
    let trySomething = unsafe ()
with
| exn ->
    let msg = sprintf  "Your error message: %s" exn.Message
    logError exn fileWriterInfo msg  //or use fileWriterInfoAzure for logging to Azure
    failwith msg
```

## CreateTable

Helper to create a Azure table:

First connect to your storage account:
```fs
open Chia.CreateTable
let connected =
    let connection = AzureConnection StorageAccount.storageConnString
    connection.Connect()
```
Now you can create your Azure table like this:

```fs
let azureTable = getTable "TableName" fileWriterInfoAzure connected
```

## CreateBlob

Helper to create a Azure blobs:

First create your blob container:

```fs
open Chia.CreateBlob
let containerInfo = {   StorageConnString = StorageConnString = StorageAccount.storageConnString
                        ContainerName = "ContainerName"}
let myContainer = getContainer containerInfo
```

Now you can get a list of all you blobs in the container like this:

```fs
let blobItems = getBlobs myContainer
```

## GetTableEntry

Helper to query a Azure tables:

With the GetTableEntry module you can easily query Azure Tables.

Get just on single tableValue like this:
```fs
let value = getValue (request.PartKey, request.RowKey) azureTable
```

Get all values in a table by using a table mapper.
First you have to define you mapper:

```fs
type Mapper = {
    PartKey : string
    RowKey : Ids.SortableRowKey
}

let mapper (entity : DynamicTableEntity) : Mapper =
    { PartyKey = entity.PartitionKey
      RowKey = SortableRowKey entity.RowKey

let values = getValues mapper azureTable
```

You can also get all values by one rowKey like this:

```fs
let valuesByRowKey = getValuesByRowKey rowKey mapper azureTable
```

If you want to create more complex queries you can just parse in a TableQuery Filter.

First define your filter:

```fs
let filter partKey = TableQuery.GenerateFilterCondition("PartitionKey", QueryComparisons.Equal, partKey)
let filteredValues partKey = getValuesWithFilter (filter partKey) azureTable
```
