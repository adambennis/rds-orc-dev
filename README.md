# AWS RDS - PostgreSQL

## Summary
This is a sample stack from the ICE team to create a PostgreSQL RDS instance with production appropriate sizing and configuration. This configuration will allow you to create instance(s) across multiple availability zones in a region and connect using Route53 writer and readonly endpoint records.

## Getting started
To get started, clone the repository and change the directory names under each tm-* directory to your environment/name. Please note that creating RDS instances can take approximately 10-15 minutes to complete.

```
git clone git@git.tmaws.io:ice/sample-rds-postgres-prod.git
cd sample-rds-postgres-prod
mv tm-intl-prod/ice1 tm-intl-prod/prod1
```
## Developing
You'll need to change the following in the terraform.tfvars file for each environment (i.e. tm-intl-prod). The file contains environment-specific variables that will get called during execution time.

* Tags

 - product_code_tag (product codes can be found [here](http://pim.tmaws.io/))
 - inventory_code_tag
 - environment_tag
 - product_name
 - resource_creator

Additional information about naming tags can be found [here.](https://contegixapp1.livenation.com/confluence/pages/viewpage.action?title=Required+Resource+Tags&spaceKey=AWS)

* VPC - vpc_remote_state_key

Please see [here](https://contegixapp1.livenation.com/confluence/display/AWS/Accessing+VPC+config+with+Terraform+remote+state#AccessingVPCconfigwithTerraformremotestate-Usage) for information on how to set <strong>vpc_remote_state_key</strong> in your terraform.tfvars.

* Variables

 - vrds_identifier (the name of the RDS instance, if omitted, Terraform will assign a random, unique identifier)
 - rds_engine_version (the engine version to use)
 - vrds_ro_count (number of read only instances to create)
 - vrds_instance_class (RDS instance class - computation and memory capacity of your instances)
 - vrds_multi_az (whether or not to create Multi-AZ instances)
 - alarm_sns_topic (a valid SNS Topic to send RDS Event alerts to)
 - cw_alarms (choose whether to receive alarms from CloudWatch)

* Using Credtool to setup the RDS master user password (in the master's region). For multi-region setup you need to run this in all relevant regions.

RDS creation requires the master user and master user password. We use [Credtool](https://contegixapp1.livenation.com/confluence/display/AWS/Credtool) to store sensitive data.
In order to store the secrets, you need to setup Credtool by adding the following alias to your bash_profile (or windows equivalent)
```
credtool() {
  docker run -it --rm -v ~/.aws:/root/.aws -v $(pwd):/credtool tmhub.io/cet/credtool:latest "$@"
}
```
Use credtool to setup DynamoDB and KMS key. Here is an example for prd298 ice1

```
   credtool setup tm-intl-prod eu-west-1 prd298 ice1
   credtool put tm-intl-prod eu-west-1 prd298 ice1 ice-sql admin

   Outputs "{encrypted}:eu-west-1:prd298-eu-west-1-ice1-credtool:prd298:ice1:ice-sql:admin"
```

* [Jana](https://contegixapp1.livenation.com/confluence/display/AWS/Jana)

Put `"{encrypted}:eu-west-1:prd298-eu-west-1-ice1-credtool:prd298:ice1:ice-sql:admin"` in config/tm-intl-prod/ice1/secret.tfvars.values

  * NOTE: this template shows an example of having the Jana config under the same repo. You can have the Jana config in a different repo, in which case, you will need to modify the get_secret.sh script with the `-gitRepo` parameter.

  * About get_secret.sh

        get_secret.sh <account_tag>/<environment_tag> <Jana config repo>

  * Modify .gitlab-ci.yml for your own RDS deployment pipeline.

  Modify .gitlab-ci.yml to use Jana to retrieve the password and apply to terraformer

## Terraformer Version

If you are running terraformer locally (not via .gitlab-ci.yml), ensure your environment is set to use the recommended version for RDS in your ~/.bash_profile

Please see for more details - https://contegixapp1.livenation.com/confluence/display/AWS/Terraformer+Version

## Planning
After the above tags and variable values have been changed, you can run <strong>terraformer plan</strong> to see what Terraform intends to create.

```
terraformer -region eu-west-1 tm-intl-prod/prod1 plan 
```

This will check against your .tfstate file to see both what exists and what will be created. If any of the items specified in the configuration do not exist you'll see a report of what Terraform intends to create.

## Deploying / Publishing
If the output of <strong>terraformer plan</strong> looks acceptable you can run the <strong>terraformer apply</strong> command to actually create the listed resources in AWS.

```
terraformer -region eu-west-1 tm-intl-prod/prod1 apply
```

## Destroying
If you wish to destroy your RDS resources in AWS, run <strong>terraformer destroy</strong>.

```
terraformer -region eu-west-1 tm-intl-prod/prod1 destroy
```

## gitlab-ci.yml
Plan, Apply, Destroy and Release-Lock stages can also be run using pipelines in gitlab.

## Terraformer Outputs
  * CNAME for each instance, whether it's the writeable master instance, or the read-only instances.

## Database Parameters
  * Avoid hard coding size related parameters. RDS provides default settings based on the instance type.
  * For addtional cluster and/or instance parameters that need customized values, add to parameters.tf

## Appendix

-- aws syntax for rds

http://docs.aws.amazon.com/AmazonRDS/latest/APIReference/API_CreateDBInstance.html

http://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/Concepts.DBInstanceClass.html

-- terraform postgres provider

https://www.terraform.io/docs/providers/postgresql/index.html

-- terraform syntax

https://www.terraform.io/docs/providers/aws/r/db_instance.html

-- terraformer versions

https://contegixapp1.livenation.com/confluence/display/AWS/Terraformer+Version

## TODO

*
