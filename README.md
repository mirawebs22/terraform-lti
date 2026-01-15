# terraform-tfc-cpchem

A Terraform root module used to manage onboarding applications to the platform.

## Application Onboarding

Applications are onboarded by creating:

- A TFC Project
- A TFC Team

## Environment Onboarding

Environments for the application (e.g. development, production) use the same
Terraform code in the application's repository but in its own TFC Workspace.

Environments are onboarded by creating:

- A TFC Workspace
- TFC Team access to the TFC Workspace
- A VCS Connection from the Git repository to the TFC Workspace

## Managing the TFC Organization

In addition to applications, this repository also manages the TFC resources from
a Platform Team's perspective. The following outlines how this was setup.

### Initial Setup

At some point there is a requirement to create the minimum resources needed to
manage resources with the TFC/TFE provider. These resources include:

- A TFC Project (Optional)
- A TFC Workspace
- A TFC Variable Set with the `TFE_TOKEN` variable.

These resouces can be created in any way, and for this project, we set them up
with Terraform locally and then migrated the state to the TFC Workspace.

#### Bootstrap Resources

The environment variable `TFE_TOKEN` must be a *user* API token, tied to an
individual with access to the GitHub Organization.

Since the workspace we are creating will be used to manage itself, we have to
first deploy it to Terraform Cloud using a local statefile (i.e. do not specify
the `cloud` block in `versions.tf`).

The following resources have been deployed this way:

- Project
- Workspace
- Project Variable Set
- `TFE_TOKEN` Environment Variable

Crucially, the credentials required to manage TFC resources have been passed in
the command line when running Terraform commands:

```shell
export TFE_TOKEN=<USER_GENERATED_TOKEN>
terraform init
terraform plan
terraform apply
```

#### Migrate to Terraform Cloud

Once the Admin project, workspace and token have been created, we need to
migrate the state. To do this, we need to update the `terraform` block to
include a `cloud` block which specifies which TFC organization and workspace to
use:

```hcl
cloud {
  organization = "cpchem"

  workspaces {
    name = "terraform-cloud-admin"
  }
}
```

Once updated, you simply have to run `terraform init` and you will be asked to
migrate the state file to the TFC workspace.

At this point, we simply need to make changes to the git repository and TFC will
manage plans and applies for us.
