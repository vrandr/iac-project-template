# Terraform/OpenTofu Template

This is a template containing a basic skeleton for infrastructure as code projects with Terraform/OpenTofu wrapped in Terragrunt.

## General structure

### Environments

Root modules that are holding state are stored in one of the environments. The environments `global`, `staging` and `production` are a good starting point.

## Create own Modules

See `environment/global/state-storage` for an example of an module.

## Hierarchical loading of data

- module loads `inputs.yaml`
- environment loads `environment_inputs.yaml`
- root loads `global_inputs.yaml`

All the loaded data is merged and passed to the called modules. The YAML files are merged with priority to specific data. That means if a value is set on root level and on module level, the module level wins.

Note that deep merging of nested values does not work. If *any* sub-values are set in a higher priority file, the complete value is overwritten and *all* sub-values that are not defined in the higher priority file will not be present after merging.


## Version constraints

It is a good idea to have version constraints defined for your code. By default a globally managed `versions_override.tf` will be created.

If you don't want to use it, for example you use an external module which already has versions defined, simple remove the `include "versions"` block.

## Root Variables

The following variables are generated by Terragrunt and added to every module:

- `environment`
- `project`
- `module`
- `region`

## Generated Code

Generated files are named "*_generated.tf", which is already excluded from git.

The templates for generated code live in `code_snippets` and can be included with `file()` or `templatefile()`.

## Terraform plugin cache dir

The environment variable `TF_PLUGIN_CACHE_DIR` is set to `~/.cache/terraform/plugin-cache` by default and can be overwritten by setting the variable otherwise. The cache directory is created if not existent. This suppresses a warning raised by terraform.

## Use OpenTofu

When using OpenTofu add the following to the `terraform` section.

```hcl
terraform {
   extra_arguments "OpenTofu binary" {
     commands = ["*"]
     env_vars = {
       TERRAGRUNT_TFPATH = "tofu"
     }
   }
}
```

## Secrets

Secrets can be stored in `secrets.yaml`. Its content is added to the inputs hash.

Add the following code to the `locals` block of `root.hcl` to use this feature.

```hcl
secrets = yamldecode(file(find_in_parent_folders("secrets.yaml")))
```

## GitLab as state storage

When using GitLab as state storage, the backend configuration required can be generated by Terragrunt.

Set following environment variables:

- `TF_HTTP_USERNAME`: Your GitLab user name
- `TF_HTTP_PASSWORD`: A  Project Access Token with scope `api` and role `Maintainer`
- `CI_API_V4_URL`: The GitLab API v4 root URL
- `CI_PROJECT_ID`: The ID of the GitLab project

Enable the `remote_state` block in the `root.hcl` file.
