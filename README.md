# Azure Container Group Sample

## Deploy a Container Registry

<a href="https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2Fdanielscholl%2Fazure-container-groups%2Fmaster%2Fregistry.json" target="_blank">
    <img src="http://azuredeploy.net/deploybutton.png"/>
</a>

```powershell
# Clone the repository
git clone https://github.com/danielscholl/azure-container-groups.git
cd azure-container-groups

# Test Locally
docker-compose up

# Deploy the Azure Container Registry
$ResourceGroup="aci-demo"
az group create --name $ResourceGroup --location eastus
az group deployment create --resource-group ${ResourceGroup} --template-file registry.json
```

## Build and Push Images

```powershell
# Build the Appliation Docker Images
docker-compose build

# Login to the Container Registry
$ResourceGroup="aci-demo"
az acr login --name $(az acr list -g ${ResourceGroup} --query [].name -otsv)

# Tag the Images and push to the Container Registry
$REGISTRY=$(az acr list -g ${ResourceGroup} --query [].loginServer -otsv)
docker tag danielscholl/aci-helloworld $REGISTRY/aci-helloworld
docker tag danielscholl/aci-sidecar $REGISTRY/aci-sidecar
docker push $REGISTRY/aci-helloworld
docker push $REGISTRY/aci-sidecar

```

## Deploy a container group

<a href="https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2Fdanielscholl%2Fazure-container-groups%2Fmaster%2Fdeploy.json" target="_blank">
    <img src="http://azuredeploy.net/deploybutton.png"/>
</a>

```powershell
# Retrieve Registry Information
$REGISTRY=$(az acr list -g ${ResourceGroup} --query [].loginServer -otsv)
$REGISTRY_USER=$(az acr credential show -g ${ResourceGroup} `
    -n $(az acr list -g ${ResourceGroup} --query [].name -otsv) --query username -otsv)
$REGISTRY_KEY=$(az acr credential show -g ${ResourceGroup} `
    -n $(az acr list -g ${ResourceGroup} --query [].name -otsv) --query passwords[0].value -otsv)

# Deploy the Azure Container Group
az group deployment create --resource-group ${ResourceGroup} --template-file deploy.json `
    --registry $REGISTRY --registryUser $REGISTRY_USER --registryKey $REGISTRY_KEY

@"
{
  "$schema": "http://schema.management.azure.com/schemas/2015-01-01/deploymentParameters.json#",
  "contentVersion": "1.0.0.0",
  "parameters": {
    "registry": {
      "value": "${REGISTRY}"
    },
    "registryUser": {
      "value": "${REGISTRY_USER}"
    },
    "registryKey": {
      "value": "${REGISTRY_KEY}"
    }
  }
}
"@ | Out-File "deploy.parameters.json"

az group deployment create --resource-group ${ResourceGroup} --template-file deploy.json `
    --parameters deploy.parameters.json


```

## Validate and test

```powershell
# Retrieve the IP Address of the Container Group
$IP=$(az container show --resource-group $ResourceGroup --name aci-demo --query ipAddress.ip -otsv)

# Open a browser
start http://$IP

# Monitor the side car logs
az container logs --resource-group $ResourceGroup --name aci-demo --container-name aci-sidecar
```
