# Scale

## Pre-req

### Create Cluster

./create_cluster.sh myCluster westeurope

### Connect ACR from demo 1

az aks update -g fasttrack-demo -n fasttrack-demo --attach-acr /subscriptions/c484c80e-0a6f-4470-86de-697ecee16984/resourceGroups/ContainerRegistry/providers/Microsoft.ContainerRegistry/registries/mikhegn

### Enable VK

k apply -f tillerrbac.yaml
helm init --service-account tiller
az aks install-connector -g cntourdemo -n cntourdemo --connector-name virtual-kubelet --os-type Both

### Enable VN

az network vnet subnet create -g myResourceGroup --vnet-name myVnet --name myVirtualNodeSubnet --address-prefixes 10.241.0.0/16 --delegations Microsoft.ContainerInstance/containerGroups
az aks enable-addons -g myResourceGroup -n myAKSCluster --addons virtual-node --subnet-name myVirtualNodeSubnet

## Demo starts here

## HPA

k apply -f hpa.yaml
k get hpa -o wide -w
watch k top pod
k exec helloparis-app-db585c8b9-wjmn2 dd if=/dev/zero of=/dev/null

## Cluster autoscaler

k describe nodes | grep -A 4 -E "Name: | Resource"
k apply -f ca.yaml
k get deployment aci-helloworld-ca
k describe nodes | grep -A 4 -E "Name: | Resource"
k get pods
k describe pod/aci-helloworld-ca-
k describe configmap/cluster-autoscaler-status -n kube-system

## Virtual Nodes scaling

k apply -f vn.yaml
k get pods

### Clean-up

k delete deployment/aci-helloworld
k delete deployment/aci-helloworld-ca
k delete deployment/aci-helloworld-hpa
k delete hpa/aci-helloworld-hpa
