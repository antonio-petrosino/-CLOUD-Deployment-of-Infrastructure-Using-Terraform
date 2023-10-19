# [CLOUD] Deployment of Infrastructure Using Terraform
Terraform configuration for Google Cloud Deployment.


## Introduction
Terraform is a powerful tool that allows you to create, modify, and enhance infrastructure in a safe and predictable manner. It's an open-source utility that translates APIs into declarative configuration files. These configuration files can be shared among team members, treated like code, edited, reviewed, and versioned.

This project focuses on using Terraform to automate the deployment of Google Cloud infrastructure. Specifically, it involves the creation of a Terraform configuration that utilizes modules to deploy a Google Cloud network in auto mode, set up a firewall rule, and provision two virtual machine (VM) instances. You can get a visual representation of the infrastructure in the diagram below:

## Objectives
Throughout this project, you will gain hands-on experience in the following tasks:

1. Crafting a configuration for an auto mode network.
2. Creating a configuration for a firewall rule.
3. Constructing a module for VM instances.
4. Generating and deploying a configuration using Terraform.
5. Confirming the successful deployment of the configuration.

Finally, to deploy the configuration, write the following cmd:

```bash
terraform fmt 
```

```bash
terraform init
```

```bash
terraform plan
```

```bash
terraform apply
```
## Conclusion

A Terraform configuration has been created with a module to automate the deployment of Google Cloud infrastructure. As your configuration changes, Terraform can create incremental execution plans, which allows you to build your overall configuration step by step.
The instance module allowed you to re-use the same resource configuration for multiple resources while providing properties as input variables. You can leverage the configuration and module that you created as a starting point for future deployments
