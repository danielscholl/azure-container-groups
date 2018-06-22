# Azure Container Group Sample

## Deploy a Container Registry

<a href="https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2Fdanielscholl%2Fazure-container-groups%2Fmaster%2Fregistry.json" target="_blank">
    <img src="http://azuredeploy.net/deploybutton.png"/>
</a>

```powershell
$ResourceGroup="aci-demo"
az group create --name $ResourceGroup --Location eastus
az group deployment create --resource-group ${ResourceGroup} --template-file registry.json
```

## Build and Push Images

```powershell
docker-compose build
$ResourceGroup="aci-demo"
az acr login --name $(az acr list -g ${ResourceGroup} --query [].name -otsv)

$REGISTRY=$(az acr list -g ${ResourceGroup} --query [].loginServer -otsv)
docker tag danielscholl/aci-helloworld $REGISTRY/aci-helloworld
docker tag danielscholl/aci-sidecar $REGISTRY/aci-sidecar
docker push $REGISTRY/aci-helloworld
docker push $REGISTRY/aci-sidecar

```

__Deploy a container group__

<a href="https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2Fdanielscholl%2Fazure-container-groups%2Fmaster%2Fdeploy.json" target="_blank">
    <img src="http://azuredeploy.net/deploybutton.png"/>
</a>

```powershell
# Registry Credentials
$REGISTRY=$(az acr list -g ${ResourceGroup} --query [].loginServer -otsv)
$REGISTRY_USER=$(az acr credential show -g ${ResourceGroup} `
    -n $(az acr list -g ${ResourceGroup} --query [].name -otsv) --query username -otsv)
$REGISTRY_KEY=$(az acr credential show -g ${ResourceGroup} `
    -n $(az acr list -g ${ResourceGroup} --query [].name -otsv) --query passwords[0].value -otsv)


az group deployment create --resource-group ${ResourceGroup} --template-file registry.json `
    -registry $REGISTRY -registryUser $REGISTRY_USER -registryKey $REGISTRY_KEY
```

__Validate and test__

```powershell
# Grab the IP Address
$IP=$(az container show --resource-group $ResourceGroup --name aci-demo --query ipAddress.ip -otsv)
start http://$IP

# Monitor the side car logs
az container logs --resource-group $ResourceGroup --name aci-demo --container-name aci-sidecar
```
