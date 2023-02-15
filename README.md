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
Created a secret in the repo named AZURE_CREDENTIALS as directed in the docs for the GH Actions here: https://github.com/marketplace/actions/azure-cli-action and created the workflow as described in the example.
