# Getting Started with Terraform

HashiCorp Terraform is a software tool that enables Developers to define infrastructure using declarative configuration files, with a process known as Infrastructure as Code (IaC). Writing your infrastructure as code enables you to take advantage of industry-standard software practices like Version Control, Continuous Integration and Continuous Delivery, and Testing.

After completing this tutorial, you will know how to:

* Create a new Terraform project
* The structure of HashiCorp Configuration Langauge (HCL) as related to Terraform
* Write infrastrucure code with HCL in Terraform
* Safely deploy a Terraform project
* Safely terminate a Terraform project

## Prerequisites

Before starting this guide, you will need the following tools installed on your computer:

* Terraform. If you have not used or installed Terraform prior to this guide, visit the [Terraform installation guide](https://learn.hashicorp.com/tutorials/terraform/install-cli) and install it using the instructions for your platform (Windows, MacOS, Linux).
* Docker. In this guide, you will deploy a docker container to your local environment. Terraform doesn't install Docker for you. If you haven't installed Docker, navigate to Docker's [Get Docker guide](https://docs.docker.com/get-docker/) to learn how to install it in your environment.

## Creating a new terraform project

To start, create a new folder for your terraform project. This can be any name, but we recommend using a name that is easy to remember and relevant to what you are building. In this case, since you are getting started with Terraform, name it `terraform-getting-started`. Once the folder is created, navigate into the folder.

```shell
$ mkdir terraform-getting-started
$ cd terraform-getting-started
```

Next, create a file for your Terraform configuration code.

```shell
$ touch main.tf
```

Paste the following lines into the file.

```hcl
terraform {
  required_providers {
    docker = {
      source = "kreuzwerker/docker"
    }
  }
}
provider "docker" {
    host = "unix:///var/run/docker.sock"
}
resource "docker_container" "nginx" {
  image = docker_image.nginx.latest
  name  = "training"
  ports {
    internal = 80
    external = 80
  }
}
resource "docker_image" "nginx" {
  name = "nginx:latest"
}
```

Initialize Terraform with the `init` command. The AWS provider will be installed. 

```shell
$ terraform init
```

You shoud check for any errors. If it ran successfully, provision the resource with the `apply` command.

```shell
$ terraform apply
```

The command will take up to a few minutes to run and will display a message indicating that the resource was created.

Finally, destroy the infrastructure.

```shell
$ terraform destroy
```

Look for a message are the bottom of the output asking for confirmation. Type `yes` and hit ENTER. Terraform will destroy the resources it had created earlier.
