# Azure Container Group Sample

__Deploy a Container Registry__

<a href="https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2Fdanielscholl%2Fazure-container-groups%2Fmaster%2Fregistry.json" target="_blank">
    <img src="http://azuredeploy.net/deploybutton.png"/>
</a>


__Build and publish the Images to a private registry__

```bash
docker-compose build
$ResourceGroup="aci-demo"
az acr login --name $(az acr list -g ${ResourceGroup} --query [].name -otsv)

REGISTRY=$(az acr list -g ${ResourceGroup} --query [].loginServer -otsv)
docker tag aci-helloworld $REGISTRY/aci-helloworld
docker tag aci-sidecar $REGISTRY/aci-sidecar
docker push $REGISTRY/aci-helloworld
docker push $REGISTRY/aci-sidecar
```


__Deploy a container group__

```bash

az group deployment create --resource-group ${ResourceGroup} --template-file deploy.json
az container create --resource-group ${ResourceGroup} --name myContainerGroup --f deploy.yaml

```