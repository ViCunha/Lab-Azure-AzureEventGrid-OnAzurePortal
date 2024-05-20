
### Overview
---
In this exercise you learn how to:

- Enable an Event Grid resource provider

- Create a custom topic

- Create a message endpoint

- Subscribe to a custom topic

- Send an event to a custom topic
   
### Key Aspects
---
- Azure Portal

- Azure Could Shell

- Azure CLI

- Azure Event Grid

### Environment
---
Microsoft Azure Portal
- Valid Subscription

### Actions
---
Prepare your environment
  
  - Sign in to the Azure portal

  - Open the Azure Cloud Shell using the Bash

Create the Azure Event Grid by using Azure CLI

  - Set environment variables
```
DEFAULT_CURRENTDATETIME="20240520162100"
DEFAULT_LOCATION="eastus2"
DEFAULT_RESOURCEGROUP_NAME="myresourcegroup${DEFAULT_CURRENTDATETIME}"
MYTOPICNAME="az204-egtopic-${DEFAULT_CURRENTDATETIME}"
MYSITENAME="az204-egsite-${DEFAULT_CURRENTDATETIME}"
MYSITEURL="https://${MYSITENAME}.azurewebsites.net"
ENDPOINT="${MYSITEURL}/api/updates"
SUBID=$(az account show --subscription "" | jq -r '.id')
AZUREEVENTGRID_SUB01="az204ViewerSub"
```

  - Create a resource group
```
az group create --name ${DEFAULT_RESOURCEGROUP_NAME} --location ${DEFAULT_LOCATION}
```

  - Register Azure Event Grid as an information provider
```
az provider show --namespace Microsoft.EventGrid --query "registrationState"
az provider register --namespace Microsoft.EventGrid
az provider show --namespace Microsoft.EventGrid --query "registrationState"
```

Create a custom topic, message and subscription

  - Create a custom topic
```
az eventgrid topic create --name ${MYTOPICNAME} --location ${DEFAULT_LOCATION} --resource-group ${DEFAULT_RESOURCEGROUP_NAME}
```

  - Create a message endpoint
```
az deployment group create --resource-group ${DEFAULT_RESOURCEGROUP_NAME} --template-uri "https://raw.githubusercontent.com/Azure-Samples/azure-event-grid-viewer/main/azuredeploy.json" --parameters siteName=${MYSITENAME} hostingPlanName=viewerhost
echo "Your web app URL: ${MYSITEURL}"
```

  - Subscribe to a custom topic
```
az eventgrid event-subscription create \
    --source-resource-id "/subscriptions/${SUBID}/resourceGroups/${DEFAULT_RESOURCEGROUP_NAME}/providers/Microsoft.EventGrid/topics/${MYTOPICNAME}" \
    --name ${AZUREEVENTGRID_SUB01} \
    --endpoint ${ENDPOINT}
```

Send an event to your custom topic

  - Retrieve URL and key for the custom topic
```
TOPICENDPOINT=$(az eventgrid topic show --name ${MYTOPICNAME} -g ${DEFAULT_RESOURCEGROUP_NAME} --query "endpoint" --output tsv)
KEY=$(az eventgrid topic key list --name ${MYTOPICNAME} -g ${DEFAULT_RESOURCEGROUP_NAME} --query "key1" --output tsv)
```
  
  - Create event data to send
```
EVENT='[ {"id": "'"$RANDOM"'", "eventType": "recordInserted", "subject": "myapp/vehicles/motorcycles", "eventTime": "'`date +%Y-%m-%dT%H:%M:%S%z`'", "data":{ "make": "Contoso", "model": "Northwind"},"dataVersion": "1.0"} ]'
```

  - Use curl to send the event to the topic
```
curl -X POST -H "aeg-sas-key: $KEY" -d "$EVENT" $TOPICENDPOINT
```

Clean up resources

  - Delete the Resource Groups and its content
```
az group delete --name ${DEFAULT_RESOURCEGROUP} --no-wait
```


### Media
---
![image](https://github.com/ViCunha/Lab-Azure-AzureEventGrid-OnAzurePortal/assets/65992033/f2b8b678-58f5-4d73-9c61-1ee238f45876)


### References
---
- [Exercise - Route custom events to web endpoint by using Azure CLI](https://learn.microsoft.com/en-us/training/modules/azure-event-grid/8-event-grid-custom-events)
