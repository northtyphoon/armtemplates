# Create a resource group
```
az group create \
  -n mytaskrunrg \
  -l westus
```

# Deploy a registry and a task run which builds/pushes to the registry
```
registry=$(az group deployment create \
  -g mytaskrunrg \
  --template-file azuredeploy.json \
  --parameters azuredeploy.parameters.json \
  --query 'properties.outputs.registry.value' \
  -o tsv)
```

# List the image tag
```
az acr repository list -n $registry -o tsv \
  | xargs -I% az acr repository show-tags -n $registry --repository % --detail -o table
```

# Crate a user assigned identity
identity=$(az identity create \
  -g mytaskrunrg \
  -n mytaskrunidentity \
  --query 'id' \
  -o tsv)

# Deploy a task run which is associated with the user assigned identity and builds/pushes an image to the registry
```
registry=$(az group deployment create \
  -g mytaskrunrg \
  --template-file azuredeploy.json \
  --parameters azuredeploy.parameters.json \
  --parameters userAssignedIdentity=$identity \
  --parameters taskRunName=mytaskrunwithidentity \
  --query 'properties.outputs.registry.value' \
  -o tsv)
```

# List the image tag
```
az acr repository list -n $registry -o tsv \
  | xargs -I% az acr repository show-tags -n $registry --repository % --detail -o table
```