set aksname bugbash

# Create the Azure AD application
set serverApplicationId (az ad app create --display-name "$aksname" --identifier-uris "https://aksname" --query appId -o tsv)

# Update the application group memebership claims
az ad app update --id $serverApplicationId --set groupMembershipClaims=All

# Create a service principal for the Azure AD application
az ad sp create --id $serverApplicationId

# Get the service principal secret
set serverApplicationSecret (az ad sp credential reset --name $serverApplicationId --credential-description "AKSPassword" --query password -o tsv

az ad app permission add --id $serverApplicationId --api 00000003-0000-0000-c000-000000000000 --api-permissions e1fe6dd8-ba31-4d61-89e7-88639da4683d=Scope 06da0dbc-49e2-44d2-8312-53f166ab848a=Scope 7ab1d382-f21e-4acd-a863-ba3e13f7da61=Role

az ad app permission grant --id $serverApplicationId --api 00000003-0000-0000-c000-000000000000
az ad app permission admin-consent --id  $serverApplicationId

set clientApplicationId (az ad app create --display-name "$aksname"Client" --native-app --reply-urls "https://"$aksname"Client" --query appId -o tsv)

az ad sp create --id $clientApplicationId

oAuthPermissionId=$(az ad app show --id $serverApplicationId --query "oauth2Permissions[0].id" -o tsv)

az ad app permission add --id $clientApplicationId --api $serverApplicationId --api-permissions ${oAuthPermissionId}=Scope

az ad app permission grant --id $clientApplicationId --api $serverApplicationId

az group create --name myResourceGroup --location EastUS

tenantId=$(az account show --query tenantId -o tsv)

az aks create \
    --resource-group myResourceGroup \
    --name $aksname \
    --node-count 1 \
    --generate-ssh-keys \
    --aad-server-app-id $serverApplicationId \
    --aad-server-app-secret $serverApplicationSecret \
    --aad-client-app-id $clientApplicationId \
    --aad-tenant-id $tenantId

    az aks get-credentials --resource-group myResourceGroup --name $aksname --admin

    az ad signed-in-user show --query userPrincipalName -o tsv