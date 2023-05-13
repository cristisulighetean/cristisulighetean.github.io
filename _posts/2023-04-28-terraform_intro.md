---
layout: post
title: "An Extensive Introduction to Terraform with Ansible"
tags: [terraform, iac]
---

How to use Terraform and Ansible to deploy an Nginx server on an AWS EC2 instance.

## Introduction

Infrastructure as Code (IaC) is an important concept in DevOps that automates the provisioning of infrastructure, eliminating the need for manual configuration. Terraform, created by HashiCorp, is an IaC tool that helps you define and provide data center infrastructure using a declarative configuration language. In this blog post, we'll explore how to use Terraform with Ansible to deploy an Nginx server on an AWS EC2 instance.

## Terraform and Ansible

While Terraform is excellent for provisioning and managing infrastructure, it doesn't handle configuration management at the server level, and this is where Ansible steps in. Ansible is an open-source tool that automates software provisioning, configuration management, and application deployment. By combining Terraform and Ansible, you can efficiently manage your entire infrastructure.

## Prerequisites

Before we begin, ensure that you have the following:

- Terraform installed on your local system.
- An AWS account.
- AWS CLI installed and configured with your AWS credentials.
- Ansible installed on your local system.

## Step 1: Creating a Terraform Configuration

First, we will create an AWS EC2 instance using Terraform. Create a new directory for your Terraform configuration and add a new main.tf file:

```sh
mkdir terraform-nginx && cd terraform-nginx
touch main.tf
```

In the main.tf file, add the following Terraform configuration:

```py
provider "aws" {
  region = "us-west-2"
}

resource "aws_instance" "nginx_server" {
  ami           = "ami-0c94855ba95c574c8" # replace with the latest Amazon Linux 2 LTS for your specific region
  instance_type = "t2.micro"
  
  tags = {
    Name = "NginxServer"
  }

  provisioner "remote-exec" {
    inline = [
      "sudo yum install -y nginx", 
      "sudo systemctl start nginx"
    ]
  }
}
```

This configuration tells Terraform to create an AWS EC2 instance in the us-west-2 region using the Amazon Linux 2 LTS AMI. It also uses a `remote-exec` provisioner to install and start Nginx on the server.

## Step 2: Applying the Terraform Configuration

To initialize your Terraform configuration and download the necessary provider plugins, run:

```sh
terraform init
```

After initialization, you can apply your configuration using:

```sh
terraform apply
```

This command will show a plan of resources to be created and prompt you for confirmation. Type yes to proceed.

## Step 3: Configuring the Server with Ansible

Once the instance is running, we can use Ansible to configure the Nginx server. First, create an Ansible playbook named nginx-playbook.yml:

```sh
touch nginx-playbook.yml
```

Add the following Ansible configuration to the playbook:

```yml
---
- hosts: all
  become: yes
  tasks:
    - name: Ensure Nginx is running
    service: nginx
      state: started
      enabled: yes
```

This playbook ensures that the Nginx service is running and enabled to start on system boot.

Before you can run the Ansible playbook, you need to have SSH access to the AWS EC2 instance. Make sure that the security group attached to your instance allows inbound SSH traffic from your IP address.

Also, ensure that you have the private key file (.pem file) associated with the key pair that you selected or created when you launched the instance.

Now, to run the playbook, use the following command:

```sh
ansible-playbook -i 'your-aws-instance-public-ip,' -u ec2-user --private-key /path/to/your/private-key.pem nginx-playbook.yml
```

In this command:

- `-i 'your-aws-instance-public-ip,'`: specifies the inventory, which is your AWS instance's public IP address. The trailing comma is necessary when you specify the inventory as a list of hosts.
- `-u ec2-user`: specifies the remote user to use. For Amazon Linux 2 instances, the default user name is ec2-user.
- `--private-key /path/to/your/private-key.pem`: specifies the path to the private key file associated with the instance.
nginx-playbook.yml: is the playbook file you want to run.

## Step 4: Verifying the Nginx Server

To verify that Nginx is running correctly, open a web browser and navigate to your EC2 instance's public IP address. You should see the Nginx welcome page, indicating that Nginx has been successfully installed and is running.

## Conclusion

In this guide, we've shown you how to use Terraform and Ansible to provision an AWS EC2 instance and configure an Nginx server. This is a simple example, but these tools are incredibly flexible and can manage much more complex infrastructure configurations and application deployments.

With Terraform and Ansible, you can automate your infrastructure management and configuration tasks, making your DevOps workflow more efficient and reliable.
