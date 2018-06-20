# Azure Container Group Sample

## Getting Started

1. __Create a Resource Group__

```bash
az group create --location southcentralus --name aci-demo
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