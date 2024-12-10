# Solving Infrastructure Challenges with Terraform

## What is Terraform
A tool for building, changing and versioning infrastructure safely and efficiently.

#### Infrastructure as Code (IaC)

- Reproducible Environments
- Idempotence and Convergence -Only the code that is meant to take the infrastructure to the desired state is executed. No other code is executed.
- Easing Collaboration - Key members can get specific version of the code and edit the infrastructure according to their preference
- Self Service Infrastructure - Developers and Operations members alike, can create and customize infrastructure according to their needs

**IaC Option for AWS:** Amazon CloudFormation

#### When to Use Terraform

- Disposable Environments
- Multi-Cloud Deployment
- Multi-Tier Applications
- Resource Schedulers

#### Typical Workflow

- Write/Modify Configuration Files - Declare the desired state of your infrastructure
- Create and execution plan using the ```plan``` command. The configuration plan describes how to bring the current infrastructure to the desired state. Terraform will take the fewest possible steps to get to your desired configuration. It also decideds in which order the dependencies must be installed. A good plan can save you time and resources and it is far inexpensive than applying changes. But plan generation can be a time consuming process. If you want to work around this then you can suggest Terraform to work on a subset of the infrastructure or tell Terraform to use previous state information to generate a plan.
- Review the plan. At this step if you don't like what you see in the plan, you can go back and modify your configuration.
- Apply the changes using ```apply``` command. If you don't provide a plan, then apply will generate one for you and apply it. If anything goes wrong then Terraform won't just roll back to the previous state. This is because apply adheres to the plan, as there was no resource deletion in the plan, that's why terraform won't do it. If you version control your infrastructure code, you can just use a previous configuration to roll back.

#### Resource Graph

- As previously mentioned, Terraform captures the dependency information in your infrastructure. The dependencies are naturally described by your configuration and you do not have to mark what depends on what, Terraform will understand it for you. 
- If there are no dependencies, Terraform will enable parallel changes and create the infrastructure as effeciently as possible
- The resource graph of terraform can be visualized

#### Automation Workflows

The typical workflow described before is a manual workflow but terraform supports automation. You may integrate terraform with a CI/CD pipeline that would automatically create infrastructure with each push and deploy to branches. If you turn on auto-accept, it can automatically apply new plans and update the infrastructure without any human intervention. But this might not be the best practice, so you are well off with a manual acceptance. You can also go for a Plan only setting. 

## Terraform Configuration

#### HashiCorp Configuration Language (HCL)
A structured configuration language that is both human and machine friendly for use with command-line tools, but specifically targeted towards DevOps tools, servers, etc. JSON is also supported by Terraform and it is mainly aimed at automation configuration generation that won't be seen by humans.

**Reosources**

Components of your Infrastructure - Virutal Machines, Databases or Version Control Systems
```
resource "google_compute_disk" "default" {
    name = "example-disk"
    zone = "us-central1-a"
}
# the resources is a google_compute_disk, which the config identifies as default. And its properties are in the curly braces. 
```

**Providers**

Resources are made available through providers. The providers are infrastructure integrations. The configurations for providers usually includes some form of credentials to grant you access to the provider's API.

```
provider 'google' {
    credentials = "${file("account.json")}"
    project     = "gcp-project-id"
    region      = "us-central1"
}
```

**Variables**

Input variables to use in your configuration

```
variable "environment_tag"{
    type = "string"
    default = "staging"
    description = "Value of the infrastructure environment tag"
}
```

**Outputs**

Outputs are infrastructure values that you want to easily access. You can think of them as highlighted infrastructure value. 

```
output "cluster_endpoint" {
    value = "${google_container_cluster.cluster.endpoint}"
}
``` 
You can easily access any output using the terraform ```output``` command.

**Data Sources**

Data sources fetch information outside of the current configuration. For example, if you have divided your infrastructure into seperate projects, you can source your data from outside your project. It is not necessary for the data to be managed by Terraform. For example - 

```
data "google_compute_address" "static_address" {
    name = "example-address"
}
```

In this example, the provider of this data is a google compute instance, which might be running an application that your configuration needs. 

#### HCL Syntax

**Commenting**

``` hcl
/*
    Multi line commenting
*/

# Single line commenting
```

**Interpolation Syntax**

```
${var.name}
${var.name["key"]}
${var.name[index]}

# Resource Attributes
${resourceType.resourceName.attribute}

# Data Sources
${data.type.name.attribute}
```

#### Interpolation Conditions
You can use if-else to make branching decisions in configuration. The syntax for this is -
```terraform
${condition ? true_expression : false_expression}
```
Condition must evaluate to ```true``` or ```false```. 

- You can also use equality operators ```==``` and ```!=```
- You can use numerical comparisons as well ```<, >, >= and <=```
- Logical Operators ```&&, || and !```

```true``` and ```false``` expressions can be any valid interpolation. Both must return the same type.

Interpolation supports basic mathematical operations on numbers. +, -, *, / and % (modulo or remainder for integers only).

Built-in functions greatly extend interpolation. Syntax: ```name(arg_1, arg_2,..)```

**Example Functions**

```hcl
abs(num)
index(list, elem)
file(path)
replace(string, search, replace)
```

#### Assigning Variables
**Default**

The most basic way to assign variables in assigning the ```default``` value. If no value is mentioned explicitly then the ```default``` value will be used.

- Used to infer type
- No interpolation allowed

```
variable "image" {
    default = "debian-cloud/debian-8"
}
```

**Interactive**

- Repetitive
- Not automation friendly

**-var options**

- Few variables
- May not be version controlled

```
-var 'env=staging"
```

**Variable Files**

- Use HCL syntax
- ```terraform.tfvars``` and ```*auto.tfvars``` in the same directory are included automatically

It is written in the configuration file like this - ```-var-file = ../vars.auto.tfvars```

Here is an example tfvars file - 

```
env = "staging"

list = ["tagA", "tagB"]

map = {
    key = "value
}
```

**Environment Variables**

It must begin with ```TF_VAR_```. In this example, you can see and environment variable named staging - ```TF_VAR_env=staging```

#### Variable Precendence
From highest to lowest precedence - 

- -var-file and -var
- Automatic variable files
- Environment variables
- defaults

Point to be noted that, map does not adhere to this variable precendence rule.

The variables that are defined in the same precendence, the last defined variable value will be used.

#### Configuration File Discovery
When you run the apply command, Terraform will look for files that end with ```.tf``` or ```.tf.json``` is using JSON. Terraform can detect a mixture of HCL and JSON. Although all of the configuration can be written in a single file, a more common pattern is dividing the files like this - 

- variables.tf
- outputs.tf
- main.tf

#### Modules

- Allow you to share and reuse configurations
- Shared accross many sources 
    - File Systems
    - GitHub
    - Amazon S3
    - Terraform Registries - it has the benefit of supporting the version specified in HCL
        - Public Registry - <https://registry.terraform.io>
        - Private

```
module "module_name" {
    source = "..."
}
```

## Provider
It is the logical abstraction of an upstream API. The provider wraps the CRUD api calls needed to manage resources. You can download the necessary providers using the ```init``` command.

#### Custom Providers
If your need is not met with the public providers, or perhaps you need a custom provider that supports your on premise storage, you can write Custom Providers. Custom Providers are - 

- Written in Go
- HashiCorp provides guides

#### Provider Versioning
Terraform will choose the latest version of the provider by default if a specific version is not mentioned. But you can also provide the version for the provider in these ways - 

- ```1.12.1``` - specifying the version number
- ```>=1.2, <1.12``` - this specifies that the version number should be greater than or equal to 1.2 but less than 1.12
- ```~> 1.7``` - This specifies that the version should be equal or greater than 1.7.0 but should be less than 2.0

You can use the same version control for modules or terraform itself

#### Provider Aliases
If you have multiple instances from a single provider, for example you want to have multiple settings from a single provider you need to identify each of them using ```alias```. One ```default``` is allowed. The default isntance is automatically specified when an alias is not mentioned. 

To use a certain provider in a resource - 
```
resource "google_compute_disk" "default" {
    provider = "google.west"
}
```

## Resources
Simply put, it is a component of your infrastructure. It can be anything from a virtual machine, a database to a version control system. Each type of resource has their own attributes and arguments. 

#### Meta-Parameters
Meta-parameters are configuration keys available to all resources. Here are some examples - 

Meta-Parameter   | Example
---------------  | ----------------------------------------------------
```provider```   | ```provider = "google.west"```
```depends_on``` | ```depends_on = ["google_compute_instance.server"]```

#### Provisioners

- Resources can configure provisioners to run scripts when created or destroyed
- Resources can have zero or more provisioners
- The syntax for including provisioners is to have a provisioner block in the resources section - 

```
provisioner "name" {
    when = "create | destroy" 
    /* indicating when the provisioner should run. 
    create is the default
    destroy maybe used to extract data or cleanup routines */
    ...
}
```

To ensure destroy time provisioner is executed, you should set the count of the resource to zero. If you instead deleted the resource the resource block, Terraform wouldn't have the configuration of the provisioner.

Provisioner can also optionally specify ```connection``` block. Many provisioners require accessing remote resources and the connection block allows you how to connect either using SSH or WinRM. For example - 
```
provisioner "name" {
    ...
    connection {
        type     = "ssh|winrm"
        user     = "username"
        password = "${var.password}"
    }
}
```

#### Lifecycle
All resources can specify a ```lifecycle``` block to customize their lifecycle behaviour. The lifecycle behaviour can specify up to three optional keys - 

- ```create_before_destroy```  - It controls whether a new resource is created before replacing it with the old destroyed resource. The default is ```false```.
- ```prevent_destroy``` - will throw an error any time an attempt is made to destroy the resource if it is set to ```true```. The default is ```false```.
- ```ignore_changes``` - tells terraform to ignore changes to specified resource attributes. You can have a list of attributes in here. 

## State
For terraform to work perfectly, it must keep track of the state of the resources that it manages. 

State maps real world resources to you configurations, keep track of metadata, and improves performance for large infrastructures. By default terraform will refresh its state before any operation. The state is used to generate plans. Terraform can persist data in several different backend such as cloud services like - AWS S3 and enhanced backends that appear to be running locally but running on a virtual machine.

#### Backends
- The default is ```local```. This stores data on the local file system. The local backend stores state in Terraform's working directory in ```terraform.tfstate```. This can be configured by adding a ```terraform``` block -
```
terraform {
    backend "local" {
        path = "relative/path/terraform.tfstate"
    }
}
```
The contents are written JSON. There is also a Remote State option for teams. This is also more secure as the data can be encrypted at rest. Terraform only stores remote state on memory, not on disk. Requests for remote state is also done using TLS or Transport Layer Security. You can also access state using data source. Terraform also supports lock in of state, which will lock the state during operation.

#### Workspaces
Workspaces are like containers for state. You can use workspaces to manage infrastructures using the same configuration files. You may want to create a workspace for every branch in the version control system. 

Workspace are managed using the ```workspace``` command. You can create new workspaces using the ```new``` subcommand and select spaces using the ```select``` subcommand. If you select new workspace then there is no state until you apply the configuration. 

The current workspace can be interpolated using - ```${terraform.workspace}```
