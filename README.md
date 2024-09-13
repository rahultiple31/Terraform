How to use Terraform Variables - Locals,Input,Output
Apr 1, 2021
· 8 min read
 · Last Modified : Jan 12, 2023 Share on:
 Author : 
Rahul Wagh


Terraform variables are a way to store values that can be reused throughout your Terraform configuration.

They allow you to define a value once and reference it in multiple places throughout your configuration, making it easier to manage and update your infrastructure.

Variables are defined in the variables block in your Terraform configuration file, where you can give a name and a default value. Please refer to the following screenshot exaplaining how variables are defined inside terraform-

terraform variables string, bool, number, list, set, map
Terraform variables can have various type such as string, number, boolean, list, map etc.
Variables can be set in the command line when running Terraform commands using the -var flag.
Variables can also be set using a separate file, called a variable file, using the -var-file flag.
Variables can be accessed in Terraform configuration files using the var function, for example var.example_variable
Variables are useful for storing values that may change between environments, for example, different values for test and production environments.

Table of Content
Types of Terraform Variables
Terraform Variables - string, number, bool
Terraform Variables - list, set, map
Terraform Output Variables
How to pass variable as tfvars file as command-line arguments using the -var-file flag?



Pre-requisite
Before we start working with Terraform variables, here are the pre-requisites -

You must install terraform (click here on how to install terraform)
You must have either AWS or Google Cloud account (Click to here for AWS and Google Cloud terraform setup tutorial)

1. Types of Terraform Variables
There are two types of variables in Terraform -

Simple values
Collection Variable
1.1 Simple Values variables
As the name suggests Simple Values variables are which hold only a single value. Here the types of Simple Value variables -

string
number
bool
1.2 Collection Variable
In the collection variable, it consists of -

List
Map
Set


2.Terraform Variables - string, number, bool
Let's take a simple example in which we are going to set up an EC2 instance on AWS.

So to create an EC2 instance we need two things -

provider
resource
Here is the main.tf which we are going to parameterized using terraform variables.

provider "aws" {
   region     = "eu-central-1"
   access_key = "<INSERT_YOUR_ACCESS_KEY>"
   secret_key = "<INSERT_YOUR_SECRET_KEY>"
}

resource "aws_instance" "ec2_example" {
   
   ami           = "ami-0767046d1677be5a0"
   instance_type = "t2.micro"
   
   tags = {
           Name = "Terraform EC2"
   }
}
bash


2.1 string variable type - We are going parameterized instance_type = "t2.micro"
The first rule to create a parameter in terraform file is by defining variable block

Example -

variable "instance_type" {
   description = "Instance type t2.micro"
   type        = string
   default     = "t2.micro"
} 
bash

For defining variable block you need
description : Small or short description about the purpose of the variable
type : What type of variable it is going to be ex - string, bool, number ...
default : What would be the default value of the variable

Let's replace the hardcoded value of instance_type with variable
 instance_type = var.instance_type
bash
Here is our final terraform file after replacing the hardcoded value of a variable -

provider "aws" {
   region     = "eu-central-1"
   access_key = "<INSERT_YOUR_ACCESS_KEY>"
   secret_key = "<INSERT_YOUR_SECRET_KEY>"
}

resource "aws_instance" "ec2_example" {

   ami           = "ami-0767046d1677be5a0"
   instance_type =  var.instance_type

   tags = {
           Name = "Terraform EC2"
   }
}

variable "instance_type" {
   description = "Instance type t2.micro"
   type        = string
   default     = "t2.micro"
}
bash
And now you can apply your terraform configuration

terraform apply 
bash


2.2 number variable type - We are going parameterized instance_count = 2
The next variable type we are going to take is number.

For example, we are going to increase the instance_count of the ec2_instances.

Let's create the variable first -

variable "instance_count" {
  description = "EC2 instance count"
  type        = number
  default     = 2
}  
bash
Here is the final terraform file with instance count -

provider "aws" {
   region     = "eu-central-1"
   access_key = "<INSERT_YOUR_ACCESS_KEY>"
   secret_key = "<INSERT_YOUR_SECRET_KEY>"
}

resource "aws_instance" "ec2_example" {

   ami           = "ami-0767046d1677be5a0"
   instance_type =  "t2.micro"
   count = var.instance_count

   tags = {
           Name = "Terraform EC2"
   }
}

variable "instance_count" {
  description = "EC2 instance count"
  type        = number
  default     = 2
}
bash


2.3 boolean variable type - We are going parameterized enable_vpn_gateway = false
The next variable type which we are going to discuss is bool.

The bool variable can be used to set true or false values inside your terraform file.

Here is an example to create your bool variable -

variable "enable_public_ip" {
  description = "Enable public IP address"
  type        = bool
  default     = true
}
bash
Let's create a complete terraform file with bool variable -

provider "aws" {
   region     = "eu-central-1"
   access_key = "<INSERT_YOUR_ACCESS_KEY>"
   secret_key = "<INSERT_YOUR_SECRET_KEY>"
}


resource "aws_instance" "ec2_example" {

   ami           = "ami-0767046d1677be5a0"
   instance_type =  "t2.micro"
   count = 1
  associate_public_ip_address = var.enable_public_ip

   tags = {
           Name = "Terraform EC2"
   }

}

variable "enable_public_ip" {
  description = "Enable public IP address"
  type        = bool
  default     = true
}
bash

3. Terraform Variables - list, set, map
When it comes to collection input variables then we are talking about -

List
Map
Set
3.1 List variable type
As the name suggests we are going to define a list that will contain more than one element in it.

Let's define our first List variable -

Here is the list of IAM users

variable "user_names" {
  description = "IAM usernames"
  type        = list(string)
  default     = ["user1", "user2", "user3s"]
}
bash
Here is our final terraform file with List variables -

provider "aws" {
   region     = "eu-central-1"
   access_key = "<INSERT_YOUR_ACCESS_KEY>"
   secret_key = "<INSERT_YOUR_SECRET_KEY>"
}
resource "aws_instance" "ec2_example" {

   ami           = "ami-0767046d1677be5a0"
   instance_type =  "t2.micro"
   count = 1

   tags = {
           Name = "Terraform EC2"
   }

}

resource "aws_iam_user" "example" {
  count = length(var.user_names)
  name  = var.user_names[count.index]
}

variable "user_names" {
  description = "IAM usernames"
  type        = list(string)
  default     = ["user1", "user2", "user3s"]
}
bash

3.2 Map variable type
Terraform also supports the map variable type where you can define the key-valye pair.

Let's take an example where we need to define project and environment, so we can use the map variable to achieve that.

Here is an example of map variable -

variable "project_environment" {
  description = "project name and environment"
  type        = map(string)
  default     = {
    project     = "project-alpha",
    environment = "dev"
  }
}
bash
Let's create a Terraform file

provider "aws" {
   region     = "eu-central-1"
   access_key = "<INSERT_YOUR_ACCESS_KEY>"
   secret_key = "<INSERT_YOUR_SECRET_KEY>"
}
resource "aws_instance" "ec2_example" {

   ami           = "ami-0767046d1677be5a0"
   instance_type =  "t2.micro"

   tags = var.project_environment

}


variable "project_environment" {
  description = "project name and environment"
  type        = map(string)
  default     = {
    project     = "project-alpha",
    environment = "dev"
  }
}
bash

4. Terraform Output Variables
In Terraform, output variables allow you to easily extract information about the resources that were created by Terraform. They allow you to easily reference the values of resources after Terraform has finished running.

Output variables are defined in the outputs block in the Terraform configuration file. Here's an example of an output variable that references the IP address of an EC2 instance:


output "instance_ip" {
   value = aws_instance.example.public_ip
}
terraform
In this example, the output variable is named "instance_ip" and its value is set to the public IP of an EC2 instance named "example" that is defined in the Terraform configuration.

What is terraform output command?

You can use the terraform output command to access the value of an output variable:

 terraform output instance_ip
52.11.222.33
bash
In addition to being able to reference output variables from the command line, you can also reference them in other parts of the Terraform configuration files using output function. For example:

resource "aws_security_group_rule" "example" {
   ...
cidr_blocks = [output.instance_ip]
}
 
terraform
Output variables are especially useful when you need to pass information about the infrastructure that Terraform has created to other systems or scripts. They also enable you to access the values of resources that are not directly visible in the Terraform state, such as the IP address of an EC2 instance.

It's worth noting that you can also set the output variable to be sensitive, in that case, Terraform will mask the output value when it appears in output, making it more secure.

5. How to pass variable as tfvars file as command-line arguments using the -var-file flag?
In Terraform, you can pass variables from a tfvars file as command-line arguments using the -var-file flag. The -var-file flag allows you to specify a file containing variable values, which will be used when running Terraform commands.

Here is an example of one my terraform project where I have created terraform.tfvars file -


terraform.tfvars file inside the terraform project
Here's an example of how you might use the -var-file flag to pass variables from a tfvars file named terraform.tfvars.

# You need to supply variable during the terraform init
terraform init -var-file=terraform.tfvars 

# You need to supply variable during the terraform plan
terraform plan -var-file=terraform.tfvars
 
# You need to supply variable during the terraform apply
terraform apply -var-file=terraform.tfvars 
bash
What tfvars file should contain?

A typical tfvars file should contain the variables that you want to pass to Terraform. Each variable should be in the form of variable_name = value. For example

project_id = "gcp-terraform-307119"
location   = "europe-central2"
terraform
But you should also create a variable.tf file also to define the variable type -


variables.tf and terraform.tfvars file containing the variables
variable "project_id" {
  type        = string
  description = "The Project ID"
}

variable "location" {
   description = "Region for project"
   default     = "europe-central2"
   type        = string
}
terraform
How to pass multiple variables files using -var-file?

You can also specify multiple variable files by using the -var-file flag multiple times on the command line. For example:

terraform apply -var-file=myvars-1.tfvars -var-file=myvars-2.tfvars
bash
It's worth noting that variables defined in the command line options will have higher priority than the variables defined in the tfvars files.


Read More - Terragrunt -

How to use Terragrunt?


Posts in this series
Securing Sensitive Data in Terraform
Boost Your AWS Security with Terraform : A Step-by-Step Guide
How to Load Input Data from a File in Terraform?
Can Terraform be used to provision on-premises infrastructure?
Fixing the Terraform Error creating IAM Role. MalformedPolicyDocument Has prohibited field Resource
In terraform how to handle null value with default value?
Terraform use module output variables as inputs for another module?
How to Reference a Resource Created by a Terraform Module?
Understanding Terraform Escape Sequences
How to fix private-dns-enabled cannot be set because there is already a conflicting DNS domain?
Use Terraform to manage AWS IAM Policies, Roles and Users
How to split Your Terraform main.tf File into Multiple Files
How to use Terraform variable within variable
Mastering the Terraform Lookup Function for Dynamic Keys
Copy files to EC2 and S3 bucket using Terraform
Troubleshooting Error creating EC2 Subnet InvalidSubnet Range The CIDR is Invalid
Troubleshooting InvalidParameter Security group and subnet belong to different networks
Managing strings in Terraform: A comprehensive guide
How to use terraform depends_on meta argument?
What is user_data in Terraform?
Why you should not store terraform state file(.tfstate) inside Git Repository?
How to import existing resource using terraform import comand?
Terraform - A detailed guide on setting up ALB(Application Load Balancer) and SSL?
Testing Infrastructure as Code with Terraform?
How to remove a resource from Terraform state?
What is Terraform null Resource?
In terraform how to skip creation of resource if the resource already exist?
How to setup Virtual machine on Google Cloud Platform
How to use Terraform locals?
Terraform Guide - Docker Containers & AWS ECR(elastic container registry)?
How to generate SSH key in Terraform
