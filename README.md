# Azure Container Group Sample

__Deploy a Container Registry__

<a href="https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2Fdanielscholl%2Fazure-container-groups%2Fmaster%2Fregistry.json" target="_blank">
    <img src="http://azuredeploy.net/deploybutton.png"/>
</a>


__Build and publish the Images to a private registry__

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


__Validate and test__

```powershell
$ContainerGroup="mycontainergroup"

# Grab the IP Address
$IP=$(az container show --resource-group $ResourceGroup --name $ContainerGroup --query ipAddress.ip -otsv)
start http://$IP

# Monitor the side car logs
az container logs --resource-group $ResourceGroup --name $ContainerGroup --container-name aci-sidecar
```

```bash
# Manually Deploy --  Change the deploy.json to
az group deployment create --resource-group ${ResourceGroup} --template-file deploy.json
az container create --resource-group ${ResourceGroup} --name myContainerGroup --f deploy.yaml

```