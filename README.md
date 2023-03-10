# Terraform Enterprise online installation with a custom agent image

With this repository you will be able to do a TFE (Terraform Enterprise) online installation on AWS with a custom agent. With the release of Terraform Enterprise Februari 2023 (v202302-1 (681)) the custom worker is changed and you must switch to the new agent based image before the release of May 2023

See [here](https://developer.hashicorp.com/terraform/enterprise/admin/infrastructure/worker-to-agent-migration) for the details 

This repository is based on the following https://github.com/munnep/TFE_aws_external for creating the default TFE environment

The Terraform code will do the following steps

- Create S3 buckets used for TFE
- Upload the necessary software/files for the TFE installation to an S3 bucket
- Generate TLS certificates with Let's Encrypt to be used by TFE
- Create a VPC network with subnets, security groups, internet gateway
- Create a RDS PostgreSQL to be used by TFE
- create roles/profiles for the TFE instance to access S3 buckets
- Create a EC2 instance on which the TFE online installation will be performed
- Create a custom work that has the Azure-cli installed which will be the default

# Diagram

![](diagram/diagram_external.png)  

# Prerequisites

## License
Make sure you have a TFE license available for use

Store this under the directory `files/license.rli`

## AWS
We will be using AWS. Make sure you have the following
- AWS account  
- Install AWS cli [See documentation](https://docs.aws.amazon.com/cli/latest/userguide/install-cliv2.html)

## Install terraform  
See the following documentation [How to install Terraform](https://learn.hashicorp.com/tutorials/terraform/install-cli)

## TLS certificate
You need to have valid TLS certificates that can be used with the DNS name you will be using to contact the TFE instance.  
  
The repo assumes you have no certificates and want to create them using Let's Encrypt and that your DNS domain is managed under AWS. 

## Docker file
Alter the Dockerfile under `files/Dockerfile` to match how you would want the worker to be

# How to

- Clone the repository to your local machine
```
git clone https://github.com/munnep/tfe_aws_custom_agent.git
```
- Go to the directory
```
cd tfe_aws_custom_agent
```
- Set your AWS credentials
```
export AWS_ACCESS_KEY_ID=
export AWS_SECRET_ACCESS_KEY=
export AWS_SESSION_TOKEN=
```
- Store the files needed for the TFE online installation under the `./files` directory, See the notes [here](./files/README.md)
- create a file called `variables.auto.tfvars` with the following contents and your own values
```
tag_prefix               = "patrick-tfe"                        # TAG prefix for names to easily find your AWS resources
region                   = "eu-north-1"                         # Region to create the environment
vpc_cidr                 = "10.234.0.0/16"                      # subnet mask that can be used 
ami                      = "ami-09f0506c9ef0fb473"              # AMI of the Ubuntu image  
rds_password             = "Password#1"                         # password used for the RDS environment
filename_license         = "license.rli"                        # filename of your TFE license stored under ./files
dns_hostname             = "patrick-tfe5"                       # DNS hostname for the TFE
dns_zonename             = "bg.hashicorp-success.com"           # DNS zone name to be used
tfe_password             = "Password#1"                         # TFE password for the dashboard and encryption of the data
certificate_email        = "patrick.munne@hashicorp.com"        # Your email address used by TLS certificate registration
tfe_release_sequence     = ""                                   # sequence version of TFE to install
terraform_client_version = "1.1.7"                              # version of terraform cli installed on the client machine
public_key               = "ssh-rsa AAAAB3Nz"                   # The public key for you to connect to the server over SSH
```
- Terraform initialize
```
terraform init
```
- Terraform plan
```
terraform plan
```
- Terraform apply
```
terraform apply
```
- Terraform output should create 40 resources and show you the public dns string you can use to connect to the TFE instance
```
Apply complete! Resources: 40 added, 0 changed, 0 destroyed.

Outputs:

ssh_tfe_server = "ssh ubuntu@patrick-tfe5.bg.hashicorp-success.com"
ssh_tfe_server_ip = "ssh ubuntu@13.51.23.34"
tfe_appplication = "https://patrick-tfe5.bg.hashicorp-success.com"
tfe_dashboard = "https://patrick-tfe5.bg.hashicorp-success.com:8800"
```

## Test the agent

- go to the directory test-code
```
cd terraform-example
```
- Make sure the `main.tf` reflects your hostname 
- run `terraform login` for the api token to use
```
terraform login <dns_of_your_tfe_environment>
```
- run terraform init
```
terraform init
```
- run terraform apply
```
terraform apply
```
- You should see the output of the azure cli version installed on the Terraform Custom Worker
```
null_resource.test: Provisioning with 'local-exec'...
null_resource.test (local-exec): Executing: ["/bin/sh" "-c" "az --version"]
null_resource.test (local-exec): WARNING: You have 1 updates available.

null_resource.test (local-exec): Please let us know how we are doing: https://aka.ms/azureclihats
null_resource.test (local-exec): and let us know if you're interested in trying out our newest features: https://aka.ms/CLIUXstudy
null_resource.test (local-exec): azure-cli                         2.39.0

null_resource.test (local-exec): core                              2.39.0
null_resource.test (local-exec): telemetry                          1.0.6 *

null_resource.test (local-exec): Dependencies:
null_resource.test (local-exec): msal                            1.18.0b1
null_resource.test (local-exec): azure-mgmt-resource             21.1.0b1
```