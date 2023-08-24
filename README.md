# Forward logs to Azure Blob Storage from ARO

## Make your storage container

Export the correct values:

```
export LOCATION=germanywestcentral
export RESOURCEGROUP=openshift
export CLUSTER_NAME=my-openshift
export AZR_STORAGE_ACCOUNT_NAME=${CLUSTER_NAME}$(cat /dev/urandom | LC_ALL=C tr -dc 'a-z0-9' | fold -w 5 | head -n 1)
```

Now create the account and the containers:

```
# Create Storage Account
az storage account create \
   --name $AZR_STORAGE_ACCOUNT_NAME \
   --resource-group $RESOURCEGROUP \
   --location $LOCATION \
   --sku Standard_RAGRS \
   --kind StorageV2
# Fetch the Azure storage key
AZR_STORAGE_KEY=$(az storage account keys list -g "${RESOURCEGROUP}" \
   -n "${AZR_STORAGE_ACCOUNT_NAME}" --query "[0].value" -o tsv)
# Create Azure Storage Containers
az storage container create --name "${CLUSTER_NAME}-logs" \
  --account-name "${AZR_STORAGE_ACCOUNT_NAME}" \
  --account-key "${AZR_STORAGE_KEY}"
```

Now edit the configmap.yaml to match your storage container name and access key string:

```
CONNECTION_STRING="DefaultEndpointsProtocol=https;AccountName=$AZR_STORAGE_ACCOUNT_NAME;AccountKey=$AZR_STORAGE_KEY;EndpointSuffix=core.windows.net"
echo $CONNECTION_STRING

vi configmap.yaml
# Edit the container_name and storage_account strings respectively
```

Now apply using kubectl:

```
kubectl apply -k .
```

## Install the OpenShift Cluster Logging Operator and OpenShift Elasticsearch operator from the console

## Create the cluster logging operator and cluster log forwarder:

```
oc create -f cluster-log-forwarder.yaml
oc create -f cluster-logging.yaml
```
