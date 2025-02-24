# Bootstrapping trust between a HCP Terrafrom/TFE workspace and Vault

This directory contains example code for setting up a Terraform Cloud workspace (TFE workspace) whose runs will be automatically authenticated to Vault using Workload Identity.

The basic building blocks in `vault.tf` will enable the `jwt` auth backend in Vault and create a role that is bound to a particular Terraform workspace.

The building blocks in `tfc-workspace.tf` will create that Terraform workspace and set all the configuration variables needed in order to allow runs to authenticate to Vault.

## How to use

You'll need the Terraform CLI installed, and you'll need to set the following environment variables in your local shell: 
1. `VAULT_TOKEN`: the Vault token that you'll use to bootstrap your trust configuration in Vault. It will need the ability to enable auth backends and create roles and policies. ( Copy the value from "Vault Admin token") 
1. `VAULT_NAMESPACE` (optional): only set this if you're not using the default Vault namespace.
1. `TFE_TOKEN`: a Terraform Cloud user token with permission to create workspaces within your organization. (Run "terraform login <your-tfc-hostname>" in the CLI, this will generate a token, copy and save it.) 


You can also use export to set those variables, before running terrafrom command. For example:
export VAULT_TOKEN = <token_valuexxxxx> 
export TFE_TOKEN = <token_tfexxxx>

Copy `terraform.tfvars.example` to `terraform.tfvars` and customize the required variables. You can also set values for any other variables you'd like to customize beyond the default.n For example
1. vault_url = "https://vault-cluster-public-vault-xxxxxxxx.abcdefgh.aa.hashicorp.cloud:8200"
1. tfc_organization_name = "TFE_Demo" #this is the organization name from HCP Terraform/ TFE. 
1. tfc_hostname = "xxxxxxxx.ting.yyy.hashidemos.io" #If you are using HCP Terraform, you don't need to set this value, as the default value is set in vars.tf file; if you are using TFE, you need to set your tfe hostname.

Check if the Vault JWT auth backend is enabled. Run `vault auth list`. 
Please note, if you already has the default JWT auth backend path set up in Vault( "JWT/"), you will run into errors when run `terrafrom apply`, you can:
1. Option1: set a custom auth backend path in the terraform tf file (terraform.tfvars). As in this demo, we create the JWT auth backend in the default path "JWT"
2. Option2: import the existing JWT auth backend. e.g. `terraform import vault_auth_backend.jwt jwt`
3. Option3: Please use this with extrem careful. You can delete the existing JWT path by running `vault auth disable jwt`


Run 
1. `terraform init`
2. `terraform plan` to verify your setup, 
3. `terraform apply` to apply the change. This will add the JWT Auth backend path, JWT Auth role, and ACL policies to Vault. 


Congratulations! You now have a Terraform Cloud workspace where runs will automatically authenticate to the given Vault instance, allowing you to use
the Vault Terraform provider to manage and retrieve secrets via Terraform.
