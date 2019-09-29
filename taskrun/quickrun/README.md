# Create a resource group
```
az group create \
  -n mytaskrunrg \
  -l westus
```

# Deploy a registry and a task run which run a multi-step task
```
registry=$(az group deployment create \
  -g mytaskrunrg \
  --template-file azuredeploy.json \
  --parameters azuredeploy.parameters.json \
  --query 'properties.outputs.registry.value' \
  -o tsv)
```

# Output the run log
```
az acr task list-runs -r $registry --query '[0].runId' -o tsv | xargs -I% az acr task logs -r $registry --run-id %
```