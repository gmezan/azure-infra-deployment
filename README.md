# Deploying Infrastructure with Terraform and GitHub Actions

## Azure set up
1. In your Azure Subscription, create a Resource Group

    ```sh
    az group create --name <Resource-Group-Name> \
        --location <Region>
    ```

1. Create a storage account in the resource group

    ```sh
    az storage account create \
        --name <Storage-Account-Name> \
        --resource-group <Resource-Group-Name> \
        --location <Region> \
        --sku <Sku>
    ```

1. Create a `tfstate` container in the storage account

    ```sh
    az storage container create \
        --name tfstate \
        --account-name <Storage-Account-Name> \
        --auth-mode login
    ```

    This container will be used to store the _tfstate_.

1. Create Managed Identity to login to Azure from Actions

    ```sh
    az identity create --name <id-gh-actions> \
        --resource-group <Resource-Group-Name> \
        --location <Region>
    ```

1. Assign _Contributor_ Role to the managed identity for the subscription

    ```sh
    az role assignment create --assignee <ManagedIdentityServicePrincipal> \
        --role Contributor \
        --scope <Subscription-Id>
    ```

1. Create the following federated credentials

    ```sh
    az identity federated-credential create --identity-name <id-gh-actions> \
                                        --name onPullRequest \
                                        --resource-group <Resource-Group-Name> \
                                        --audiences api://AzureADTokenExchange \
                                        --issuer https://token.actions.githubusercontent.com \
                                        --subject repo:<owner>/<repo>:pull_request

    az identity federated-credential create --identity-name <id-gh-actions> \
                                        --name onBranch \
                                        --resource-group <Resource-Group-Name> \
                                        --audiences api://AzureADTokenExchange \
                                        --issuer https://token.actions.githubusercontent.com \
                                        --subject repo:<owner>/<repo>:ref:refs/heads/main-azure

    az identity federated-credential create --identity-name <id-gh-actions> \
                                        --name onEnvironment \
                                        --resource-group <Resource-Group-Name> \
                                        --audiences api://AzureADTokenExchange \
                                        --issuer https://token.actions.githubusercontent.com \
                                        --subject repo:<owner>/<repo>:environment:azure
    ```

1. Configure credentials in GH. Go to the correspondent GH repo. In _Settings_ > _Secrets and variables_ > _Actions_, create the following secrets with the values corresponding to the created managed identity.
   - `AZURE_CLIENT_ID`
   - `AZURE_SUBSCRIPTION_ID`
   - `AZURE_TENANT_ID`

1. Go to the correspondent repo. In _Settings_ > _Environments_, create the `azure` environment.


Taking inspiration from https://github.com/Azure-Samples/terraform-github-actions/



## AWS set up

1. Create an IAM Access Key in the AWS console.

1. Configure credentials in GH. Go to the correspondent GH repo. In _Settings_ > _Secrets and variables_ > _Actions_, create the following secrets with the values corresponding to the created access key.
   - `AWS_ACCESS_KEY_ID`
   - `AWS_SECRET_ACCESS_KEY`

1. Create a PR to `main-aws` and deploy the infra
![image](https://github.com/user-attachments/assets/22f12b53-f7d4-4dea-9df7-956ee73515a0)


Taking inspiration from:
- https://spacelift.io/blog/github-actions-terraform
- https://aws.amazon.com/blogs/modernizing-with-aws/automate-microsoft-web-application-deployments-with-github-actions-and-terraform/
