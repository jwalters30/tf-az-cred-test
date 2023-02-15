# tf-az-cred-test
Used for testing credentials in Azure AD with Terraform

## Getting Started

Created a new repo (this one), and created a set of credentials for it using the examples at: https://learn.microsoft.com/en-us/cli/azure/create-an-azure-service-principal-azure-cli
```
PS C:\Users\JamesWalters> az ad sp create-for-rbac --name jwTenantOwner --role "Owner" --scope "/"
Creating 'Owner' role assignment under scope '/'
The output includes credentials that you must protect. Be sure that you do not include these credentials in your code or check the credentials into your source control. For more information, see https://aka.ms/azadsp-cli
{
  "appId": "0d51675c-<snip>-f452afa23f70",
  "displayName": "jwTenantOwner",
  "password": "<nope-not-doing-that>",
  "tenant": "08c40d9c-<snip>-34bc8b2bc7c6"
}
```
Created a secret in the repo named AZURE_CREDENTIALS as directed in the docs for the GH Actions here: https://github.com/marketplace/actions/azure-cli-action and created the workflow as described in the example.  NOTE: This is failing to work in GitHub Actions at the moment, more on that later.

While trying to troubleshoot that mess, I tried logging in on the Azure CLI with the SP and trying to list the other SPs:
```
PS C:\Users\JamesWalters> az login --service-principal -u 0d51675c-<snip>-f452afa23f70 -p <nope-not-doing-that> --tenant 08c40d9c-<snip>-34bc8b2bc7c6
[
  {
    "cloudName": "AzureCloud",
    "homeTenantId": "08c40d9c-<snip>-34bc8b2bc7c6",
    "id": "d00942f6-<snip>-aa271ffaa70f",
    "isDefault": true,
    "managedByTenants": [],
    "name": "Azure subscription 1",
    "state": "Enabled",
    "tenantId": "08c40d9c-<snip>-34bc8b2bc7c6",
    "user": {
      "name": "0d51675c-<snip>-f452afa23f70",
      "type": "servicePrincipal"
    }
  }
]
PS C:\Users\JamesWalters> az ad sp list >sp-list2.txt
ERROR: Insufficient privileges to complete the operation.
```
It failed in the same way that I was having trouble with in the other tenant, so at least that is repeatable.  Adding the "Cloud Application Administrator" or "Application Administrator" roles fails (despite using the same syntax that was used for the "Owner" role):
```
PS C:\Users\JamesWalters> az role assignment create --assignee 0d51675c-be7d-44a1-a804-f452afa23f70 --role "Application Administrator" --scope "/"
Role 'Application Administrator' doesn't exist.
PS C:\Users\JamesWalters> az role assignment create --assignee 0d51675c-be7d-44a1-a804-f452afa23f70 --role "Cloud Application Administrator" --scope "/"
Role 'Cloud Application Administrator' doesn't exist.
```

When I list the role assignments that I have added to the SP, it seems like it should have the rights to do a fair number of things:
```
PS C:\Users\JamesWalters> az role assignment list --assignee 0d51675c-be7d-44a1-a804-f452afa23f70 --all --include-inherited
[
  {
    "canDelegate": null,
    "condition": null,
    "conditionVersion": null,
    "description": null,
    "id": "/providers/Microsoft.Authorization/roleAssignments/f4558cc3-66c0-4c12-bd89-a2a71289e671",
    "name": "f4558cc3-66c0-4c12-bd89-a2a71289e671",
    "principalId": "86b07746-<snip>-7510d77802a6",
    "principalName": "0d51675c-<snip>-f452afa23f70",
    "principalType": "ServicePrincipal",
    "roleDefinitionId": "/subscriptions/d00942f6-<snip>-aa271ffaa70f/providers/Microsoft.Authorization/roleDefinitions/8e3af657-a8ff-443c-a75c-2fe8c4bcb635",
    "roleDefinitionName": "Owner",
    "scope": "/",
    "type": "Microsoft.Authorization/roleAssignments"
  },
  {
    "canDelegate": null,
    "condition": null,
    "conditionVersion": null,
    "description": null,
    "id": "/providers/Microsoft.Authorization/roleAssignments/65e5cea3-aca9-4006-a3c6-e13e883904e2",
    "name": "65e5cea3-aca9-4006-a3c6-e13e883904e2",
    "principalId": "86b07746-<snip>-7510d77802a6",
    "principalName": "0d51675c-<snip>-f452afa23f70",
    "principalType": "ServicePrincipal",
    "roleDefinitionId": "/subscriptions/d00942f6-<snip>-aa271ffaa70f/providers/Microsoft.Authorization/roleDefinitions/350f8d15-c687-4448-8ae1-157740a3936d",
    "roleDefinitionName": "Hierarchy Settings Administrator",
    "scope": "/",
    "type": "Microsoft.Authorization/roleAssignments"
  },
  {
    "canDelegate": null,
    "condition": null,
    "conditionVersion": null,
    "description": null,
    "id": "/providers/Microsoft.Authorization/roleAssignments/9353527f-514b-4df9-acdb-0e8a1cc6394d",
    "name": "9353527f-514b-4df9-acdb-0e8a1cc6394d",
    "principalId": "86b07746-<snip>-7510d77802a6",
    "principalName": "0d51675c-<snip>-f452afa23f70",
    "principalType": "ServicePrincipal",
    "roleDefinitionId": "/subscriptions/d00942f6-<snip>-aa271ffaa70f/providers/Microsoft.Authorization/roleDefinitions/18d7d88d-d35e-4fb5-a5c3-7773c20a72d9",
    "roleDefinitionName": "User Access Administrator",
    "scope": "/",
    "type": "Microsoft.Authorization/roleAssignments"
  },
  {
    "canDelegate": null,
    "condition": null,
    "conditionVersion": null,
    "description": null,
    "id": "/providers/Microsoft.Authorization/roleAssignments/c59d2572-9c43-4add-b6b5-9d49a1ebca2a",
    "name": "c59d2572-9c43-4add-b6b5-9d49a1ebca2a",
    "principalId": "86b07746-<snip>-7510d77802a6",
    "principalName": "0d51675c-<snip>-f452afa23f70",
    "principalType": "ServicePrincipal",
    "roleDefinitionId": "/subscriptions/d00942f6-<snip>-aa271ffaa70f/providers/Microsoft.Authorization/roleDefinitions/acdd72a7-3385-48ef-bd42-f606fba81ae7",
    "roleDefinitionName": "Reader",
    "scope": "/",
    "type": "Microsoft.Authorization/roleAssignments"
  }
]
```

### The Workaround

The thing that ended up working, was to assign the "Cloud Application Administrator" role in the Azure Portal.

![Screenshot_20230215_105241](https://user-images.githubusercontent.com/4760734/219106640-383cb281-755d-404a-a3d0-2cdcb2213049.png)

Note that there is no evidence of that role being assigned in the Azure CLI.

