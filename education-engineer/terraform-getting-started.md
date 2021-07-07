# Getting Started with Terraform

HashiCorp Terraform is a software tool that enables Developers to define infrastructure using declarative configuration files, with a process known as Infrastructure as Code (IaC). Writing your infrastructure as code enables you to take advantage of industry-standard software practices like Version Control, Continuous Integration and Continuous Delivery, and Testing.

After completing this tutorial, you will know how to:

* Create a new Terraform project
* Write infrastrucure code with the Terraform language
* Validate a Terraform project
* Terminate a Terraform project

## Prerequisites

Before starting this guide, you will need the following tools installed on your computer:

- Terraform. If you have not used or installed Terraform before, visit the [Terraform installation guide](https://learn.hashicorp.com/tutorials/terraform/install-cli) and install it using the instructions for your environment (Windows, MacOS, Linux).

- Docker. In this guide, you will deploy a docker container to your local environment. Terraform doesn't install Docker for you. If you haven't installed Docker, navigate to Docker's [Get Docker guide](https://docs.docker.com/get-docker/) to learn how to install it in your environment.

## Creating a new Terraform project folder

To start, create a new folder for your Terraform project. This can be any name, but we recommend using a name that is easy to remember and relevant to what you are building. In this case, since you are getting started with Terraform, name it `terraform-getting-started`. Once the folder is created, navigate into the folder.

```shell
$ mkdir terraform-getting-started
$ cd terraform-getting-started
```



## Initializing a new Terraform project

Every Terraform  project must be initialized before you are able to deploy infrastructure. This initialization can be run in two scenarios:

* Before any other project files are present in the folder.
* After any other project files are present in the folder.

Since this project is new, initialize the project now, before any other project files are present. We will show you what `terraform init` does when there are files present in a later step.

```shell
$ terraform init
Terraform initialized in an empty directory!

The directory has no Terraform configuration files. You may begin working
with Terraform immediately by creating Terraform configuration files.
```

With the terraform project initialized, you are now ready to begin adding files to this project.

## Building out a Terraform project

Start building this project by creating a new file in the folder, `main.tf`

```shell
$ touch main.tf
```

The file `main.tf` represents the *primary* entrypoint for the [*root module* of your project](https://learn.hashicorp.com/tutorials/terraform/module?in=terraform/modules#what-is-a-terraform-module). Each project will have exactly one root module, and is required to exist in the root directory of your project.

Terraform's configuration language is built on top of [HashiCorp Configuration Language (HCL).](https://www.terraform.io/docs/language/syntax/configuration.html) It's primary purpose is to enable you to declare resources for your project, represented as *infrastructure objects*. All other parts of the Terraform language exist to support and streamline the creation of these objects.

## Writing Terraform code

Start by opening up `main.tf` in your preferred text editor. Add the following code to the beginning of the file.

```hcl
terraform {
  required_providers {
    docker = {
      source = "kreuzwerker/docker"
    }
  }
}
```

This code defines the basic Terraform resource, and specifies that Docker is a required provider, and declares the source of the provider. In your case, this source is `kreuzwerker/docker` the primary [Docker provider for Terraform.](https://registry.terraform.io/providers/kreuzwerker/docker/latest/docs) 

Add the following lines of directly below the previously inserted code:

```hcl
provider "docker" {
    host = "unix:///var/run/docker.sock"
}
```

With the root `terraform` object delcaring docker as a required provider, you will specify Docker's host location, via the `host` configuration value.

With the provider created, you will now build out the docker container to be deployed with this provider. To do so, you will create two resources: One for the Docker image you'll want to use, and then a resource for the docker container built from that image. Then, you will specify the configuration for that container. Insert these lines of code directly below the previously inserted code:

```hcl
resource "docker_image" "nginx" {
  name = "nginx:latest"
}

resource "docker_container" "nginx" {
  image = docker_image.nginx.latest
  name  = "training"
  ports {
    internal = 80
    external = 80
  }
}

```

The `docker_image` resource specifies the name of the docker image, and it's revision. Since you are deploying a web server, you are using the latest revision of nginx, `nginx:latest`.

The `docker_container` resource uses the  `docker_image` resource by calling its *reference name* with the revision of the image to use in the `image` key. 

Next,  you will specify a name for the container, `training`, and the internal & external ports for the container, `80`. Docker will use these values when building the container.

Once completed, your file should be structured as shown below:

```hcl
terraform {
  required_providers {
    docker = {
      source = "kreuzwerker/docker"
    }
  }
}

resource "docker_image" "nginx" {
  name = "nginx:latest"
}

resource "docker_container" "nginx" {
  image = docker_image.nginx.latest
  name  = "training"
  ports {
    internal = 80
    external = 80
  }
}
```



## Managing provider plugins

With our Terraform project in place, validate that the project is capable of running a deployment. Use `terraform plan` to run this validation.

```shell
$ terraform plan

Error: Could not load plugin
│ 
│ 
│ Plugin reinitialization required. Please run "terraform init".
│ 
│ Plugins are external binaries that Terraform uses to access and manipulate
│ resources. The configuration provided requires plugins which can't be located,
│ don't satisfy the version constraints, or are otherwise incompatible.
│ 
│ Terraform automatically discovers provider requirements from your
│ configuration, including providers used in child modules. To see the
│ requirements and constraints, run "terraform providers".
│ 
│ failed to instantiate provider "registry.terraform.io/kreuzwerker/docker" to obtain schema: unknown provider
│ "registry.terraform.io/kreuzwerker/docker"
```

This error is expected. Don't worry! Earlier, we mentioned that there are two instances in which you run `terraform init` - Once when the folder is empty of files, and again when it's not empty. This is the second instance. In the previous step, you built a resource provider for Docker. One of `terraform init`'s responsibilities is to load plugins for specified providers if the plugin doesn't exist in the project. In this case, you ran `terraform init` before a resource provider was defined, which means the plugin for it was not loaded.

Run `terraform init` again to install the required plugins.

```shell
$ terraform init
Initializing the backend...

Initializing provider plugins...
- Finding latest version of kreuzwerker/docker...
- Installing kreuzwerker/docker v2.13.0...
- Installed kreuzwerker/docker v2.13.0 (self-signed, key ID 24E54F214569A8A5)

Partner and community providers are signed by their developers.
If you'd like to know more about provider signing, you can read about it here:
https://www.terraform.io/docs/cli/plugins/signing.html

Terraform has created a lock file .terraform.lock.hcl to record the provider
selections it made above. Include this file in your version control repository
so that Terraform can guarantee to make the same selections by default when
you run "terraform init" in the future.

Terraform has been successfully initialized!

You may now begin working with Terraform. Try running "terraform plan" to see
any changes that are required for your infrastructure. All Terraform commands
should now work.

If you ever set or change modules or backend configuration for Terraform,
rerun this command to reinitialize your working directory. If you forget, other
commands will detect it and remind you to do so if necessary.
```



## Terraform plan, continued

Now that you have successfully installed the plugins required for the Docker provider, run `terraform plan` again to validate the code, and understand the resources being created.

```shell
$ terraform plan
Terraform used the selected providers to generate the following execution plan. Resource actions are indicated
with the following symbols:
  + create

Terraform will perform the following actions:

  # docker_container.nginx will be created
  + resource "docker_container" "nginx" {
      + attach           = false
      + bridge           = (known after apply)
      + command          = (known after apply)
      + container_logs   = (known after apply)
      + entrypoint       = (known after apply)
      + env              = (known after apply)
      + exit_code        = (known after apply)
      + gateway          = (known after apply)
      + hostname         = (known after apply)
      + id               = (known after apply)
      + image            = (known after apply)
      + init             = (known after apply)
      + ip_address       = (known after apply)
      + ip_prefix_length = (known after apply)
      + ipc_mode         = (known after apply)
      + log_driver       = "json-file"
      + logs             = false
      + must_run         = true
      + name             = "training"
      + network_data     = (known after apply)
      + read_only        = false
      + remove_volumes   = true
      + restart          = "no"
      + rm               = false
      + security_opts    = (known after apply)
      + shm_size         = (known after apply)
      + start            = true
      + stdin_open       = false
      + tty              = false

      + healthcheck {
          + interval     = (known after apply)
          + retries      = (known after apply)
          + start_period = (known after apply)
          + test         = (known after apply)
          + timeout      = (known after apply)
        }

      + labels {
          + label = (known after apply)
          + value = (known after apply)
        }

      + ports {
          + external = 80
          + internal = 80
          + ip       = "0.0.0.0"
          + protocol = "tcp"
        }
    }

  # docker_image.nginx will be created
  + resource "docker_image" "nginx" {
      + id          = (known after apply)
      + latest      = (known after apply)
      + name        = "nginx:latest"
      + output      = (known after apply)
      + repo_digest = (known after apply)
    }

Plan: 2 to add, 0 to change, 0 to destroy.

──────────────────────────────────────────────────────────────────────────────────────────────────────────────

Note: You didn't use the -out option to save this plan, so Terraform can't guarantee to take exactly these
actions if you run "terraform apply" now.
```

Values that display "(Known after apply)" means that these values are generated when the resources are created when `terraform apply` is invoked. `terraform plan` doesn't create any resources; it evaluates the project and outputs what it *plans* to create. We highly recommend to run `terraform plan`  before you run `terraform apply`. This can help you catch errors like typos or other unexpected behavior in your project, before the resources are created.

## Running Terraform apply

You are now ready to deploy your project with `terraform apply`. The output will prompt you to confirm your changes by typing in `yes` at the prompt. Press Enter when ready to continue.

```shell
$ terraform apply
Terraform used the selected providers to generate the following execution plan. Resource actions are indicated
with the following symbols:
  + create

Terraform will perform the following actions:

  # docker_container.nginx will be created
  + resource "docker_container" "nginx" {
      + attach           = false
      + bridge           = (known after apply)
      + command          = (known after apply)
      + container_logs   = (known after apply)
      + entrypoint       = (known after apply)
      + env              = (known after apply)
      + exit_code        = (known after apply)
      + gateway          = (known after apply)
      + hostname         = (known after apply)
      + id               = (known after apply)
      + image            = (known after apply)
      + init             = (known after apply)
      + ip_address       = (known after apply)
      + ip_prefix_length = (known after apply)
      + ipc_mode         = (known after apply)
      + log_driver       = "json-file"
      + logs             = false
      + must_run         = true
      + name             = "training"
      + network_data     = (known after apply)
      + read_only        = false
      + remove_volumes   = true
      + restart          = "no"
      + rm               = false
      + security_opts    = (known after apply)
      + shm_size         = (known after apply)
      + start            = true
      + stdin_open       = false
      + tty              = false

      + healthcheck {
          + interval     = (known after apply)
          + retries      = (known after apply)
          + start_period = (known after apply)
          + test         = (known after apply)
          + timeout      = (known after apply)
        }

      + labels {
          + label = (known after apply)
          + value = (known after apply)
        }

      + ports {
          + external = 80
          + internal = 80
          + ip       = "0.0.0.0"
          + protocol = "tcp"
        }
    }

  # docker_image.nginx will be created
  + resource "docker_image" "nginx" {
      + id          = (known after apply)
      + latest      = (known after apply)
      + name        = "nginx:latest"
      + output      = (known after apply)
      + repo_digest = (known after apply)
    }

Plan: 2 to add, 0 to change, 0 to destroy.

Do you want to perform these actions?
  Terraform will perform the actions described above.
  Only 'yes' will be accepted to approve.

  Enter a value: yes

docker_image.nginx: Creating...
docker_image.nginx: Creation complete after 8s [id=sha256:36741ec2ad2ba699d1b7161a06d4a7dff7d1d6602bddf862d1a29ce09158c58bnginx:latest]
docker_container.nginx: Creating...
docker_container.nginx: Creation complete after 1s [id=f46d96b7b4076cf70069333207f2cd745ceb14dcb5867b037aaf7e9b16d1add6]
Apply complete! Resources: 2 added, 0 changed, 0 destroyed.
```

Because this project is smaller in size, the resources creation will occur quickly. As your project grows with more resources, the amount of time for `terraform apply` to run will increase.

## Validating resource creation

Since Terraform created this resource, you can validate with the provider this resource exists. In the case of this docker container, you can check with two steps:

* Validate the docker container named `training` is running.
* Run the `curl` command in your terminal to the local address, or visit the local address in your web browser to confirm the server is running and responding to requests.

```shell
$ docker ps
CONTAINER ID   IMAGE          COMMAND                  CREATED          STATUS          PORTS                NAMES
f46d96b7b407   36741ec2ad2b   "/docker-entrypoint.…"   59 minutes ago   Up 59 minutes   0.0.0.0:80->80/tcp   training
```

```shell
$ curl localhost:80
<!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx!</title>
<style>
    body {
        width: 35em;
        margin: 0 auto;
        font-family: Tahoma, Verdana, Arial, sans-serif;
    }
</style>
</head>
<body>
<h1>Welcome to nginx!</h1>
<p>If you see this page, the nginx web server is successfully installed and
working. Further configuration is required.</p>

<p>For online documentation and support please refer to
<a href="http://nginx.org/">nginx.org</a>.<br/>
Commercial support is available at
<a href="http://nginx.com/">nginx.com</a>.</p>

<p><em>Thank you for using nginx.</em></p>
</body>
</html>
```

![image](https://user-images.githubusercontent.com/24903897/124830066-01bc8d80-df3f-11eb-8549-5177ce462681.png)


## Cleaning up with terraform `destroy`

To finish, you can clean up these resources by running `terraform destroy` which will remove the running container and remove the docker image from your system. It will prompt you again to confirm that you want to remove these resources from the provider.

```shell
$ terraform destroy
docker_image.nginx: Refreshing state... [id=sha256:36741ec2ad2ba699d1b7161a06d4a7dff7d1d6602bddf862d1a29ce09158c58bnginx:latest]
docker_container.nginx: Refreshing state... [id=f46d96b7b4076cf70069333207f2cd745ceb14dcb5867b037aaf7e9b16d1add6]

Note: Objects have changed outside of Terraform

Terraform detected the following changes made outside of Terraform since the last "terraform apply":

  # docker_container.nginx has been changed
  ~ resource "docker_container" "nginx" {
      + dns               = []
      + dns_opts          = []
      + dns_search        = []
      + group_add         = []
        id                = "f46d96b7b4076cf70069333207f2cd745ceb14dcb5867b037aaf7e9b16d1add6"
      + links             = []
      + log_opts          = {}
        name              = "training"
      + sysctls           = {}
      + tmpfs             = {}
        # (31 unchanged attributes hidden)

        # (1 unchanged block hidden)
    }

Unless you have made equivalent changes to your configuration, or ignored the relevant attributes using ignore_changes, the following plan may include
actions to undo or respond to these changes.

──────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────

Terraform used the selected providers to generate the following execution plan. Resource actions are indicated with the following symbols:
  - destroy

Terraform will perform the following actions:

  # docker_container.nginx will be destroyed
  - resource "docker_container" "nginx" {
      - attach            = false -> null
      - command           = [
          - "nginx",
          - "-g",
          - "daemon off;",
        ] -> null
      - cpu_shares        = 0 -> null
      - dns               = [] -> null
      - dns_opts          = [] -> null
      - dns_search        = [] -> null
      - entrypoint        = [
          - "/docker-entrypoint.sh",
        ] -> null
      - env               = [] -> null
      - gateway           = "172.17.0.1" -> null
      - group_add         = [] -> null
      - hostname          = "f46d96b7b407" -> null
      - id                = "f46d96b7b4076cf70069333207f2cd745ceb14dcb5867b037aaf7e9b16d1add6" -> null
      - image             = "sha256:36741ec2ad2ba699d1b7161a06d4a7dff7d1d6602bddf862d1a29ce09158c58b" -> null
      - init              = false -> null
      - ip_address        = "172.17.0.2" -> null
      - ip_prefix_length  = 16 -> null
      - ipc_mode          = "private" -> null
      - links             = [] -> null
      - log_driver        = "json-file" -> null
      - log_opts          = {} -> null
      - logs              = false -> null
      - max_retry_count   = 0 -> null
      - memory            = 0 -> null
      - memory_swap       = 0 -> null
      - must_run          = true -> null
      - name              = "training" -> null
      - network_data      = [
          - {
              - gateway                   = "172.17.0.1"
              - global_ipv6_address       = ""
              - global_ipv6_prefix_length = 0
              - ip_address                = "172.17.0.2"
              - ip_prefix_length          = 16
              - ipv6_gateway              = ""
              - network_name              = "bridge"
            },
        ] -> null
      - network_mode      = "default" -> null
      - privileged        = false -> null
      - publish_all_ports = false -> null
      - read_only         = false -> null
      - remove_volumes    = true -> null
      - restart           = "no" -> null
      - rm                = false -> null
      - security_opts     = [] -> null
      - shm_size          = 64 -> null
      - start             = true -> null
      - stdin_open        = false -> null
      - sysctls           = {} -> null
      - tmpfs             = {} -> null
      - tty               = false -> null

      - ports {
          - external = 80 -> null
          - internal = 80 -> null
          - ip       = "0.0.0.0" -> null
          - protocol = "tcp" -> null
        }
    }

  # docker_image.nginx will be destroyed
  - resource "docker_image" "nginx" {
      - id          = "sha256:36741ec2ad2ba699d1b7161a06d4a7dff7d1d6602bddf862d1a29ce09158c58bnginx:latest" -> null
      - latest      = "sha256:36741ec2ad2ba699d1b7161a06d4a7dff7d1d6602bddf862d1a29ce09158c58b" -> null
      - name        = "nginx:latest" -> null
      - repo_digest = "nginx@sha256:3ca76089b14cf7db77cc5d4f3e9c9eb73768b9c85a0eabde1046435a6aa41c06" -> null
    }

Plan: 0 to add, 0 to change, 2 to destroy.

Do you really want to destroy all resources?
  Terraform will destroy all your managed infrastructure, as shown above.
  There is no undo. Only 'yes' will be accepted to confirm.

  Enter a value: yes

docker_container.nginx: Destroying... [id=f46d96b7b4076cf70069333207f2cd745ceb14dcb5867b037aaf7e9b16d1add6]
docker_container.nginx: Destruction complete after 1s
docker_image.nginx: Destroying... [id=sha256:36741ec2ad2ba699d1b7161a06d4a7dff7d1d6602bddf862d1a29ce09158c58bnginx:latest]
docker_image.nginx: Destruction complete after 0s

Destroy complete! Resources: 2 destroyed.
```

You will see similar output. Note: In the case of this example, `terraform destroy` indicated that the `id` and `name` values were updated since we previously  ran `terraform apply`. When we ran `terraform apply` for the first time, it created the value for `id` and created the docker container `training` at runtime. This error message can be confusing for new Terraform users. This [Issue thread on GitHub](https://github.com/hashicorp/terraform/issues/28803) goes into a bit more detail if you are curious for more information.

## Next steps

Congratulations, you have made it all the way through this guide! You have created a new Terraform project that utilizes Docker to create a new local Docker image to run nginx inside your local environment. You have written Terraform code, validated it, deployed it, and terminated it, all on your own. Be proud!

Are you ready for more tutorials? The [HashiCorp Learn portal](https://learn.hashicorp.com/), contains tutorials of different skill levels that go into more detail. Some recommended next steps to go into more detail with Terraform:

* [HashiCorp Configuration Language](https://learn.hashicorp.com/collections/terraform/configuration-language)
* [Terraform Modules tutorials](https://learn.hashicorp.com/collections/terraform/modules)
* [Example applications deployed with Terraform](https://learn.hashicorp.com/collections/terraform/applications)

