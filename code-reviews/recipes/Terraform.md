# Terraform Code Reviews

## Style Guide

[CSE](../../CSE.md) developers follow the [hashicorp style guide](https://www.terraform.io/docs/configuration/style.html).

[CSE](../../CSE.md) projects should check Terraform scripts with automated tools.

## Code Analysis / Linting

### TFLint

[`TFLint`](https://github.com/terraform-linters/tflint) is a Terraform linter focused on possible errors, best practices, etc. Once TFLint installed in the environment, it can be invoked using the VS Code [`terraform extension`](https://marketplace.visualstudio.com/items?itemName=mauve.terraform).

## VS Code Extensions

The following VS Code extensions are widely used.

### [`Terraform extension`](https://marketplace.visualstudio.com/items?itemName=mauve.terraform)

This extension provides syntax highlighting, linting, formatting and validation capabilities.

### [`Azure Terraform extension`](https://marketplace.visualstudio.com/items?itemName=ms-azuretools.vscode-azureterraform)

This extension provides Terraform command support, resource graph visualization and CloudShell integration inside VS Code.

## Build Validation

Ensure you enforce the style guides during build. The following example script can be used to install terraform and a linter that
then checks for formatting and common errors.

```shell
#! /bin/bash
set -e

SCRIPT_DIR=$(dirname "$BASH_SOURCE")
cd "$SCRIPT_DIR"

TF_VERSION=0.12.4
TF_LINT_VERSION=0.9.1

echo -e "\n\n>>> Installing Terraform 0.12"
# Install terraform tooling for linting terraform
wget -q https://releases.hashicorp.com/terraform/${TF_VERSION}/terraform_${TF_VERSION}_linux_amd64.zip -O /tmp/terraform.zip
sudo unzip -q -o -d /usr/local/bin/ /tmp/terraform.zip

echo ""
echo -e "\n\n>>> Install tflint (3rd party)"
wget -q https://github.com/wata727/tflint/releases/download/v${TF_LINT_VERSION}/tflint_linux_amd64.zip -O /tmp/tflint.zip
sudo unzip -q -o -d /usr/local/bin/ /tmp/tflint.zip

echo -e "\n\n>>> Terraform version"
terraform -version

echo -e "\n\n>>> Terraform Format (if this fails use 'terraform fmt -recursive' command to resolve"
terraform fmt -recursive -diff -check

echo -e "\n\n>>> tflint"
tflint

echo -e "\n\n>>> Terraform init"
terraform init

echo -e "\n\n>>> Terraform validate"
terraform validate
```

## Code Review Checklist

In addition to the [Code Review Checklist](../readme.md) you should also look for these Terraform specific code review items

* [ ] Are all providers used in the terraform scripts [versioned](https://www.terraform.io/docs/configuration/providers.html#provider-versions) to prevent breaking changes in the future?
* [ ] Are modules split into separate `.tf` files where appropriate?
* [ ] Does the repository contain a `README.md` describing the architecture provisioned?
* [ ] Is the Terraform project configured using Azure Storage as remote state backend?
* [ ] Is the remote state backend storage account key is stored a secure location (e.g. Azure Key Vault)?
* [ ] If Terraform code is mixed with application source code, is the Terraform code isolated into a dedicated folder?
* [ ] If the infrastructure is going to different depending on the environment (e.g. Dev, UAT, Production), are the environment specific parameters supplied via `.tfvars` file?
* [ ] Is the project configured to use state file based on the environment and the deployment pipeline configured to supply the state file name dynamically?
* [ ] Are the resource definitions and data sources handled correctly in the Terraform scripts?
    resource : Indicates to Terraform that the current configuration is in charge of managing the life cycle of the object
    data: Indicates to Terraform that you only want to get a reference to the existing object, but don’t want to manage it as part of this configuration
* [ ] Is the code split into multiple reusable modules?
* [ ] Are unit tests used for Terraform code (e.g. [`Terratest`](https://terratest.gruntwork.io/))?
* [ ] Are the resource names starting with their containing provider's name followed by an underscore? e.g. resource from the provider `postgresql` might be named as `postgresql_database`?
* [ ] Is `try` function used only with simple attribute references and type conversion functions?, as overuse of `try` function to suppress errors will lead to a configuration that is hard to understand and maintain
* [ ] Are the explicit type conversion functions used to normalize types returned only in module outputs?, as the explicit type conversions are rarely necessary in Terraform because it will convert types automatically where required
* [ ] Is `Sensitive` property on schema set to `true` for the fields that contains sensitive information?. This will prevent the field's values from showing up in CLI output
