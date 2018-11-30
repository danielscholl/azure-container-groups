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
$ResourceGroup="demo-aci"
az group create --name $ResourceGroup --location westus
az group deployment create --resource-group ${ResourceGroup} --template-file registry.json
```

## Build and Push Images

```powershell
# Build the Appliation Docker Images
docker-compose build

# Login to the Container Registry
$ResourceGroup="demo-aci"
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

# Create Parameters via PowerShell
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

# Create Parameters via bash
cat > deploy.parameters.json <<EOF
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
EOF

# Deploy on Public Network
az group deployment create --resource-group ${ResourceGroup} --template-file deploy.json `
    --parameters deploy.parameters.json

# Deploy on Private VNET
az group deployment create --resource-group ${ResourceGroup} --template-file deploy_on_network.json `
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


## Delete
> Note: ACI Network integration is in preview and you have to remove it manually.

```powershell
# Replace <my-resource-group> with the name of your resource group


# Get network profile ID
NETWORK_PROFILE_ID=$(az network profile list --resource-group $ResourceGroup --query [0].id --output tsv)

# Delete the network profile
az network profile delete --id $NETWORK_PROFILE_ID -y

# Get the service association link (SAL) ID
SAL_ID=$(az network vnet subnet show --resource-group $ResourceGroup --vnet-name aci-vnet --name aci-subnet --query id --output tsv)/providers/Microsoft.ContainerInstance/serviceAssociationLinks/default

# Delete the default SAL ID for the subnet
az resource delete --ids $SAL_ID --api-version 2018-08-01

# Delete the subnet delegation to Azure Container Instances
az network vnet subnet update --resource-group $ResourceGroup --vnet-name aci-vnet --name aci-subnet --remove delegations 0

# Delete the subnet
az network vnet subnet delete --resource-group $ResourceGroup --vnet-name aci-vnet --name aci-subnet

# Delete virtual network
az network vnet delete --resource-group $ResourceGroup --name aci-vnet
```