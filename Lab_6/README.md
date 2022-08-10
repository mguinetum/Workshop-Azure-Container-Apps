# Lab_6: Pipeline CD
<img width='800' src='../images/Lab_6/Lab_6_00.png'/><br>
## Objectif:
L'objectif de ce Lab 6 c'est de ...<br>

## Prération de l'environnement
Affectation des variables:<br>
```
RESOURCE_GROUP="RG-Lab6"
LOCATION="eastus2"
ACR_NAME="acrlab6"
LOG_ANALYTICS_NAME="pierrc-workspace-lab-6"
CONTAINERAPPS_ENVIRONMENT="environment-lab-6"
APLLICATION="hello"


```
Installation de l'environnement:<br>
```
az extension add --name containerapp --upgrade
az provider register --namespace Microsoft.App
az provider register --namespace Microsoft.OperationalInsights

az group create \
  --name ${RESOURCE_GROUP} \
  --location ${LOCATION}

az acr create \
  --resource-group ${RESOURCE_GROUP} \
  --name ${ACR_NAME} \
  --sku Basic \
  --admin-enabled true

az monitor log-analytics workspace create \
  --resource-group ${RESOURCE_GROUP} \
  --workspace-name ${LOG_ANALYTICS_NAME} \
  --location ${LOCATION}
LOG_ANALYTICS_WORKSPACE_CLIENT_ID=`az monitor log-analytics workspace show --query customerId -g ${RESOURCE_GROUP} -n ${LOG_ANALYTICS_NAME} --out tsv`
LOG_ANALYTICS_WORKSPACE_PRIMARY_KEY=`az monitor log-analytics workspace get-shared-keys --query primarySharedKey -g ${RESOURCE_GROUP} -n ${LOG_ANALYTICS_NAME} --out tsv`

az containerapp env create \
  --name ${CONTAINERAPPS_ENVIRONMENT} \
  --resource-group ${RESOURCE_GROUP} \
  --location ${LOCATION} \
  --logs-workspace-id $LOG_ANALYTICS_WORKSPACE_CLIENT_ID \
  --logs-workspace-key $LOG_ANALYTICS_WORKSPACE_PRIMARY_KEY
```

## "Build & Push" l'application

```
cd ./Lab_6/App
az acr build -t ${ACR_NAME}.azurecr.io/${APLLICATION}:1.0.0 -r ${ACR_NAME} .
```

## Déploiement de l'application
```
az containerapp create \
  --name ${APLLICATION} \
  --resource-group ${RESOURCE_GROUP} \
  --environment ${CONTAINERAPPS_ENVIRONMENT} \
  --image ${ACR_NAME}.azurecr.io/${APLLICATION}:1.0.0 \
  --target-port 3000 \
  --ingress external \
  --registry-server ${ACR_NAME}.azurecr.io \
  --query configuration.ingress.fqdn
```
A la fin de l'installation:<br>
```
Container app created. Access your app at https://hello.kinddune-d6b77287.eastus2.azurecontainerapps.io/
```
Test de l'application:<br>
Faire un "curl" sur l'URL ex: `curl https://hello.kinddune-d6b77287.eastus2.azurecontainerapps.io` 