# Azure Container Group Sample

## Getting Started

1. __Create an Azure Container Registry__

```bash
# Create Resource Group
ResourceGroup="aci-demo"
Location="southcentralus"
Registry="acidemo"

az group create --name ${ResourceGroup} \
    --location ${Location}
az acr create --name ${Registry} \
    --resource-group ${ResourceGroup} \
    --location ${Location} \
    --sku Standard 
```

1. __Deploy Container Registry to Resource Group__

```bash
az group deployment create --name iac-registry --template-file registry.json --resource-group aci-demo
```

1.  __Login to Container Registry__

```bash
$REGISTRY=(az acr list -g aci-demo --query [].name -otsv)
az acr login --name $REGISTRY
```