# Terraform Beginner Bootcamp 2023

## Semantic versioning :mage:

This project is going to utilize semantic versioning for its tagging. [semver.org](https://semver.org/)

The general format:

**MAJOR.MINOR.PATCH**, eg. `1.0.1`

- **MAJOR** version when you make incompatible API changes
- **MINOR** version when you add functionality in a backward compatible manner
- **PATCH** version when you make backward compatible bug fixes

## Install the terraform CLI

### Considerations with the Terraform CLI changes
The terraform CLI installation instructions have changed due to gpg keyring changes. So we needed to refer to the latest install CLI instructions via Terraform Documentation and change the scripting for install. 

[Install terraform CLI](https://developer.hashicorp.com/terraform/tutorials/aws-get-started/install-cli)

### Considerations for Linux distribution

This project is built aganist Ubuntu. Please consider checking your Linux Distribution and change accordingly to distribution needs.
[How to check OS version in Linux](https://www.ionos.com/digitalguide/server/know-how/how-to-check-your-linux-version/#:~:text=The%20command%20%E2%80%9Cuname%20%2Dr%E2%80%9D,0%2D26.)

Example of checking OS version
```sh
$ cat /etc/os-release

PRETTY_NAME="Ubuntu 22.04.3 LTS"
NAME="Ubuntu"
VERSION_ID="22.04"
VERSION="22.04.3 LTS (Jammy Jellyfish)"
VERSION_CODENAME=jammy
ID=ubuntu
ID_LIKE=debian
HOME_URL="https://www.ubuntu.com/"
SUPPORT_URL="https://help.ubuntu.com/"
BUG_REPORT_URL="https://bugs.launchpad.net/ubuntu/"
PRIVACY_POLICY_URL="https://www.ubuntu.com/legal/terms-and-policies/privacy-policy"
UBUNTU_CODENAME=jammy
```

### Refactoring into Bash scripts

While fixing the Terraform CLI gpg deprecation issuers we notice that bash scripts steps were a considerable amount more code. So we decided to create a bash script to install the Terraform CLI.

This bash script is located here: [./bin/install_terraform_cli](./bin/install_terraform_cli.sh)

- This will keep the Gitpod Task File ([.gitpod.yml](.gitpod.yml)) tidy.
- This allow us an easier to debug and execute manually Terraform CLI install
- This will allow better portability for other projects that need to install Terraform CLI.


#### Shebang

A Shebang (pronouced Sha-bang) tells the bash script what program will interpret the script. eg `#!/bin/bash`

ChatGPT recommended that we use this format for bash `#!/usr/bin/env bash`

- for portability for different OS distributions
- will search the user's PATH for the bash executable

https://en.wikipedia.org/wiki/Shebang_(Unix)

#### Execution considerations

When executing bash scripts, we can use the `./` shorthand notation to execute the bash script.

eg. `./bin/install_terraform_cli`

If we are using a script in .gitpod.yml we need to point the script to a program to interpret it.

eg. `source ./bin/install_terraform_cli`


#### Linux permissions considerations

In order to make our bash script executable we need to change linux permissinos for the fix to be executable at the user mode.

```sh
chmod u+x ./bin/install_terraform_cli
```

alternatively:

```sh
chmod 744 ./bin/install_terraform_cli
```

https://en.wikipedia.org/wiki/Chmod

### Github Lifecycle (Before, Init, Command)

We need to be careful when using the Init because it will not rerun if we restart an existing workspace.

https://www.gitpod.io/docs/configure/workspaces/tasks


### Working with Env Vars

We can list out all environment variables using the `env` command

We can filter specific environment variables using grep eg. `env | grep AWS_`

#### Setting and Unsetting Env Vars

In the terminal we can set using `export HELLO='world'`

In the terminal we can unset using `unset HELLO`

We can set and env var temporarily when just running a command 

```sh
#!/usr/bin/env bash

HELLO='world'

echo $HELLO
```

#### Printing Env vars

We can print an env var using echo eg. `echo $HELLO`

#### Scoping of Env vars

When you open up new bash terminals in vscode, it will not be aware of env vars that you have set in another window.

If you want env vars to persist across all future terminals, that are open, you need to set env vars in your bash profile. eg. `.bash_profile`

#### Persisting Env Vars in gitpod

We can persist env vars in gitpod by storing them in Gitpod secretes storage

```
gp env HELLO='world'
```

All future workspaces launced will set the env vars for all bash terminal opened in those workspaces.

You can also set env vars in the `.gitpod.yml` but this can only contain non-sensitive env vars


We can check if our AWS credentials is configured correctly by running the following AWS CLI command:
```sh
aws sts get-caller-identity
```

If it is successful you should see a json payload return that looks like this
```json
{
    "UserId": "SDODAFLDJFKJSFDF",
    "Account": "1226712172871",
    "Arn": "arn:aws:iam::1226712172871:user/terraform-beginner-bootcamp"
}
```

We'll need to generate AWS CLI credentials for IAM user in order to use the AWS CLI 


## Terraform Basics

### Terraform Registry

Terraform sources their providers and modules from the terraform registry which is located at [registry.terraform.io](https://registry.terraform.io/)

- **Providers** is an interface to APIs that will allow to create resources in terraform.
[Random terraform provider](https://registry.terraform.io/providers/hashicorp/random)
- **Modules** are a way to refactor or to make large amounts of terraform code modular, portable and sharable.

### Terraform Console

We can see a list of all the terraform command by simply typing `terraform`

#### Terraform init

At the start of a new terraform project we will run `terraform init` to download the binaries for the terraform providers that we'll use in this project. 

#### Terraform plan

`terraform plan`

This will generate out a changeset, about the state of our infrastructure and what will be changed.

We can output this changeset i.e. "plan" to be passed to an apply, but often you can just ignore outputting.

#### Terraform apply

`terraform apply`

This will run a plan and pass the changeset to be executed by terraform. Apply should prompt us yes or no.

If we want to automatically approve an apply we can provide the auto approve flag. eg `terraform apply --auto-approve`.

### Terraform lock files

`.terraform.lock.hcl` contains the locked versioning for the providers or modules that should be used with this project.

The Terraform Lock File **should be committed **to your Version Control System (VCS) eg. GitHub.

### Terraform State files

`.terraform.tfstate` contains information about the current state of your infrastructure.

This file **should not be committed** to your VCS. This file can contain sensitive data. If you lose this file, you lose knowing the state of your infrastructure.

`.terraform.tfstate.backup` is the previous state file state.

### Terraform directory

`.terraform` directory contains binaries of terraform providers.