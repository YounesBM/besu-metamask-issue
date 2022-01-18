# 1. Changes to the base repository (https://github.com/ConsenSys/quorum-kubernetes)

## prod/helm/charts/goquorum-genesis/templates/genesis-job.yaml
We modified the genesis.json file to put the same accounts as in the playground ("alloc" property).
The private keys of these accounts can be retrieved in this file:
https://github.com/ConsenSys/quorum-kubernetes/blob/master/playground/kubectl/quorum-besu/ibft2/configmap/besu-genesis-configmap.yaml

## Provider "Azure"
In the prod/helm/charts folder, for all the "values.yaml" files, the value of the "provider" field has been changed to "azure".

# 2. Deployment

## Azure Deployment
Create an Azure resource group.
In the created resource group, deploy using the azure/arm/azuredeploy.json file.

## Adapt files in the prod/helm/values folder
In the prod/helm/values folder, for each file, adapt the values of the following properties:


    azure:
	    # the script/bootstrap.sh uses the name 'quorum-pod-identity' so only change this if you altered the name
	    identityName: quorum-pod-identity
	    # the clientId of the user assigned managed identity created in the template
	    identityClientId: azure-clientId
	    # the name of the keyvault
	    keyvaultName: azure-keyvault
	    # the tenant ID of the key vault
	    tenantId: azure-tenantId
	    # the subscription ID to use - this needs to be set explicitly when using multi tenancy
	    subscriptionId: azure-subscriptionId

## Deploy the charts
In the prod/helm folder, deploy the charts in the "besu" namespace.

    helm install monitoring ./charts/quorum-monitoring --namespace besu
    helm install genesis ./charts/besu-genesis --namespace besu --values ./values/genesis-besu.yml
    
    helm install bootnode-1 ./charts/besu-node --namespace besu --values ./values/bootnode.yml
    helm install bootnode-2 ./charts/besu-node --namespace besu --values ./values/bootnode.yml
    
    helm install validator-1 ./charts/besu-node --namespace besu --values ./values/validator.yml
    helm install validator-2 ./charts/besu-node --namespace besu --values ./values/validator.yml
    helm install validator-3 ./charts/besu-node --namespace besu --values ./values/validator.yml
    helm install validator-4 ./charts/besu-node --namespace besu --values ./values/validator.yml
    
    helm install member-1 ./charts/besu-node --namespace besu --values ./values/txnode.yml
    
    helm repo add ingress-nginx https://kubernetes.github.io/ingress-nginx
    helm repo update
    helm install besu-ingress ingress-nginx/ingress-nginx \
        --namespace besu \
        --set controller.replicaCount=1 \
        --set controller.nodeSelector."beta\.kubernetes\.io/os"=linux \
        --set defaultBackend.nodeSelector."beta\.kubernetes\.io/os"=linux \
        --set controller.admissionWebhooks.patch.nodeSelector."beta\.kubernetes\.io/os"=linux \
        --set controller.service.externalTrafficPolicy=Local

    # login to the cluster
    kubectl apply -f ../../ingress/ingress-rules-besu.yml


# 3. Metamask Issue & Logs

## Metamask

![Metamask Connection](https://github.com/YounesBM/logs-quorum/blob/main/Metamask%20network.JPG?raw=true)

## Logs member-1

    kubectl logs besu-node-member-1-0 -c member-1-besu --namespace besu

![Member-1 logs](https://github.com/YounesBM/logs-quorum/blob/main/Metamask.JPG?raw=true)