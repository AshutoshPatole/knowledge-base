[[terraform]] providers manage resources by communicating between Terraform and target APIs. Whenever the target APIs change or add functionality, provider maintainers may update and version the provider.

When multiple users or automation tools run the same Terraform configuration, they should all use the same versions of their required providers. There are two ways for you to manage provider versions in your configuration.

1. Specify provider version constraints in your configuration's `terraform` block.
2. Use the [dependency lock file](https://developer.hashicorp.com/terraform/language/files/dependency-lock)

If you do not scope provider version appropriately, Terraform will download the latest provider version that fulfills the version constraint. This may lead to unexpected infrastructure changes.

### Explore `terraform.lock.hcl`

When you initialize a Terraform configuration for the first time with Terraform 1.1 or later, Terraform will generate a new `.terraform.lock.hcl` file in the current working directory. You should include the lock file in your version control repository to ensure that Terraform uses the same provider versions across your team and in ephemeral remote execution environments.


>[!info]
>You should never directly modify the lock file.

If you want to upgrade the versions in the lock file, do `terraform init -upgrade`.  The `-upgrade` flag will upgrade all providers to the latest version consistent within the version constraints specified in your configuration.


### Providers that are no longer required

- To determine whether there still exists a dependency on a given provider, Terraform uses two sources of truth: the configuration itself, and the state. If you remove the last dependency on a particular provider from both your configuration and state, then `terraform init` will remove any existing lock file entry for that provider.

- If you add a new requirement for the same provider at a later date and run `terraform init` again, Terraform will treat it as if it were an entirely new provider and so will not necessarily select the same version that was previously selected and will not be able to verify that the checksums remained unchanged.