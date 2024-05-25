# Terraform

- [Guía exhaustiva de Terraform](https://blog.gruntwork.io/a-comprehensive-guide-to-terraform-b3d32832baca)
- [Libro de Terraform](http://www.terraformupandrunning.com/?ref=gruntwork-blog-comprehensive-terraform)
- [Infrastructure as code talk code examples](https://github.com/brikis98/infrastructure-as-code-talk)
- Terraform debugging: [Look for panic in crash.log](https://github.com/hashicorp/terraform/pull/5726/commits/209b69197179ac427981c64f1d89e21d43a542d7) or [docs](https://www.terraform.io/docs/internals/debugging.html)
- Directory Structure: [reddit](https://www.reddit.com/r/devops/comments/53sijz/how_do_you_structure_terraform_configurations/) - [typicalrunt](https://typicalrunt.me/2015/10/24/terraform-file-organization/) - [tecst](https://teckst.com/teckst-tech/organizing-terraform-projects/) - [pete's tips](https://medium.com/@petey5000/petes-terraform-tips-694a3c4c5169) - [opencredo](https://opencredo.com/terraform-infrastructure-design-patterns/)
- [Best Practices](https://stackoverflow.com/questions/33157516/best-practices-when-using-terraform)


## Locking Terraform state with Terragrunt

Terragrunt is a thin wrapper for Terraform that manages remote state for you automatically and provides locking by using Amazon DynamoDB. [How to manage terraform state](https://blog.gruntwork.io/how-to-manage-terraform-state-28f5697e68fa)


## About isolating environments
- Multiple state files per ENV → https://charity.wtf/2016/03/30/terraform-vpc-and-why-you-want-a-tfstate-file-per-env/


> The whole point of having separate environments is that they are isolated from each other, so if you are managing all the environments from a single set of Terraform templates, you are breaking that isolation. Just as a ship has bulkheads that act as barriers to prevent a leak in one part of the ship from immediately flooding all the others, you should have “bulkheads” built into your Terraform design.
> 
> The way to do that is to put the Terraform templates for each environment into a separate folder. For example, all the templates for the staging environment can be in a folder called “stage” and all the templates for the production environment can be in a folder called “prod”. That way, Terraform will use a separate state file for each environment, which makes it significantly less likely that a screw up in one environment can have any impact on another.


- [How to manage terraform state](https://blog.gruntwork.io/how-to-manage-terraform-state-28f5697e68fa)

**Folder Layout**

    stage
      └ vpc
      └ services
          └ frontend-app
          └ backend-app
              └ vars.tf
              └ outputs.tf
              └ main.tf
              └ .terragrunt
      └ data-storage
          └ mysql
          └ redis
    prod
      └ vpc
      └ services
          └ frontend-app
          └ backend-app
      └ data-storage
          └ mysql
          └ redis
    mgmt
      └ vpc
      └ services
          └ bastion-host
          └ jenkins
    global
      └ iam
      └ route53


> There is, however, one problem with this file layout: it makes it harder to use resource dependencies. If your app code was defined in the same Terraform template as the VPC code, then that app could directly access attributes of the VPC (e.g. subnet IDs) using Terraform’s interpolation syntax (e.g. “${aws_subnet.foo.id}”). But if if the app code and VPC code live in different folders, as recommended above, you can no longer do that. Fortunately, Terraform offers a solution: read-only state.


## Versioned Modules

If both your staging and production environment are pointing to the same module folder, then as soon as you make a change in that folder, it will affect both environments on the very next deployment. This sort of coupling makes it harder to test out a change in staging without any chance of affecting prod. A better approach is to create “versioned modules” so that you can use one version in staging and a different version in prod.

In the examples above, we set the “source” parameter of the module to a local file path, but Terraform supports other types of module sources, such as Git URLs, Mercurial URLs, and arbitrary HTTP URLs. The easiest way to create a versioned module is to put the code for the module in a separate Git repository and to set the source parameter to that repository’s URL. That means most Terraform projects consist of (at least) two repos:


- Infrastructure-modules: This repo defines reusable modules. Think of each module as a “blueprint” that defines a specific part of your infrastructure.
- Infrastructure-live: This repo defines the live infrastructure you’re running in each environment (stage, prod, mgmt, etc). Think of this as the “houses” you built from the “blueprints” in the infrastructure-modules repo.

Let’s also add a tag to this repo, which we can use as a version number. If you’re using GitHub, you can use the GitHub UI to create a release, which will create a tag under the hood. If you’re not using GitHub, you can use the Git CLI:


    git tag -a "v0.0.1" -m "First release of frontend-app"
    git push --follow-tags

Now you can use this versioned module in both staging and production by specifying a Git URL in the source parameter. Here is what that would look like for a GitHub repo with the organization name gruntwork-io (Note that the double-slash in the URL is required, as documented here):


    module "frontend" {
    source = "git::git@github.com:gruntwork-io/infrastructure-modules.git//frontend-app?ref=v0.0.1"
    (...)

