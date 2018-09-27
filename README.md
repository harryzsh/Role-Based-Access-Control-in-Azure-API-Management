# How to use Role-Based Access Control in Azure API Management

Recently I have recieved a request from one of APIM customer. He claims that the APIM with RBAC documentation (https://docs.microsoft.com/en-us/azure/api-management/api-management-role-based-access-control)isn't suffcient enough for him to cover in his environment. After helping them with their request, I have decided to write an article about it, which may help APIM users in the future.

Let's start with this customer scenario. Basically Customer wishes to create custom role in APIM, which will users in it to have a contributor privilege to a specific API and only has read permission to this particular API from APIM within Azure portal, which means no other API, product, subscriptions ect... can be read with this role.

Unfortunately, this isn't covered by our documentation. Before we continue on this topic, I do expect you having read the  APIM with RBAC documentation (https://docs.microsoft.com/en-us/azure/api-management/api-management-role-based-access-control) already.

Let me use my environment to walk you through the example code.
* Resource Group : harryAPIMRG
* APIM : harryApim
* API : echo-ap, harryCustomApi

Let's first create a custom role named "APIM Service ReadOnly". We then assign it to at resource group level and add a user into this custom role.  

```sh
$role = Get-AzureRmRoleDefinition "API Management Service Reader Role"
$role.Id = $null
$role.AssignableScopes.Clear()
$role.AssignableScopes.Add('/subscriptions/abe6e3ee-6236-4c01-94f6-316cca169e88/resourceGroups/harryAPIMRG’)
$role.name = “APIM Service ReadOnly”
$role.actions.remove(“Microsoft.ApiManagement/service/*/read)
New-AzureRmRoleDefinition -Role $role
New-AzureRmRoleAssignment -ObjectId 2befd43f-aa17-4eda-b45a-20cc5dd57d40 -RoleDefinitionName 'APIM ReadOnly' -Scope '/subscriptions/abe6e3ee-6236-4c01-94f6-316cca169e88/resourceGroups/harryAPIMRG'
```

You may have already noticed that we have removed "Microsoft.ApiManagement/service/*/read" action within the "APIM Service ReadOnly" role. The reason why we did that is because we only want to allow users from this custom role has read permission to the APIM service only but not anything else within it like product, APIs, Subscriptions ect... If you log into azure portal with the user from the custom role, you are now able to see harryAPIMGR resource group and harryApim APIM service. 


Right! secondly we can simply follow the exact same code from Microsoft documentation to create a custom role and assign it to the specified API level. We use echo-api at this example

```sh
$role2 = Get-AzureRmRoleDefinition "API Management Service Reader Role"
$role2.Id = $null
$role2.AssignableScopes.Clear()
$role2.AssignableScopes.Add('/subscriptions/abe6e3ee-6236-4c01-94f6-316cca169e88/resourceGroups/harryAPIMRG/providers/Microsoft.ApiManagement/service/harryApim')
$role2.name = “Echo API Management” 
$role2.Actions.Add('Microsoft.ApiManagement/service/apis/write')
$role2.Actions.Add('Microsoft.ApiManagement/service/apis/*/write')
New-AzureRmRoleDefinition -Role $role2
New-AzureRmRoleAssignment -ObjectId 2befd43f-aa17-4eda-b45a-20cc5dd57d40 -RoleDefinitionName 'Echo API Management' -Scope '/subscriptions/abe6e3ee-6236-4c01-94f6-316cca169e88/resourceGroups/harryAPIMRG/providers/Microsoft.ApiManagement/service/harryApim/apis/echo-api'
```

Now log into Aure portal again, you should be able to see resource group, APIM service and echo-api within it. However, you still don't have any read permission to other things like product, susbcriptions, Named values etc...

Please note that there is also another reason why we need to create a custom role at resource group level. It's because When a user from the API custom role uses ARM template or HTTP REST API to deploy an AP, they need a custom role at resource group level. Otherwsie, you will see resource group not found or don't have permission to resource group exceptions.