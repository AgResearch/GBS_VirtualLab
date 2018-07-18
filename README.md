# GBS Virtual Lab

This repository contains [Ansible Playbooks](https://docs.ansible.com/ansible/latest/user_guide/playbooks.html) to create a [Genotype By Sequencing](https://en.wikipedia.org/wiki/Genotyping_by_sequencing) Virtual Lab in the Microsoft Azure Cloud.  This virtual lab is the basic building block for running the GBS pipeline developed by AgResearch Ltd.

## Prerequisites / Assumptions

These playbooks assumes the localhost, where playbooks are executed:

* has [MS Azure CLI](https://docs.microsoft.com/en-us/cli/azure/install-azure-cli?view=azure-cli-latest) installed
* has Ansible installed
* has Ansible [configured](https://docs.ansible.com/ansible/latest/scenario_guides/guide_azure.html) to use Azure modules
* the submodule included in the repository is initialised:

    ```
    $ git submodule update --init --recursive
    ```

## Configurations

There are three Ansible variable files included in this project in the ```var``` directory, which defines the behaviour of these playbooks. 

### azure.yml

This file specifies the following:

* ```resoruce_location``` the MS Azure location where where a resource group will be created;
* ```resource_group``` the name of a resource group that hosts all MS Azure resources required by a GBS virtual lab;
* ```vm_name``` name of the virtual machine which provide the virtual lab environment;
* ```admin_username``` user name of the administrator on the created virtual machine.

### users.yml

A list of users who will have access to the created virtual lab.  Each user has the following 4 attributes:

* ```name``` - full name of the user;
* ```usercode``` - usercode used by the user to logon the virtual lab via SSH;
* ```password``` - encrypted password of the user when the user logging on.  See documentation [here](https://docs.ansible.com/ansible/faq.html#how-do-i-generate-crypted-passwords-for-the-user-module) for details of various ways to generate an encrypted password; 
* ```email``` email address of the user, although it not used at present.

### secret.yml

This file include all sensitive information, such as a password in plain text, used by playbooks.  This shall be protected by using Ansible Vault.

* ```admin_password``` - password for the administrator.

Update values of the above variables to meet the deployment's requirements.

## Deploying GBS Virtual Lab

Execute the following Ansible playbooks in directory ```playbook``` on a host that meet perquisites defined in the beginning of this document.

1. Create a Resource Group to host all MS Azure resources required by the virtual lab:

    ```
    $ ansible-playbook playbook/01-CreateAzureResourceGroup.yml
    ```

2. Create resources in the created Resource Group:

    ```
    $ ansible-playbook playbook/02-CreateAzureResources.yml
    ```

    If ```secrets.yml``` is protected by Ansible Vault as recommended, use the following instead and then enter the password to decrypt the protected file.

    * Retrieve the public IP allocated to the newly created VM vi MS Azure Portal;
    * Update the ```inventory``` file accordingly, so Ansible can access the VM and become *root*;

3. Configure the VM as a GBS virtual lab:

    * Deploy a shared [miniconda](https://conda.io/miniconda.html) environment;
    * Deploy all software packages required by the GBS pipeline;
    * Create user accounts

    ```
    $ ansible-playbook --ask-become-pass playbook/03-DeployGBSVirtualLab.yml
    ```

4. **[Optional]** Destroy the Virtual Lab

    When the virtual lab is no longer required, destroy its parent Resource Group to free up all resource in MS Azure.  All data stored in the virtual lab will be **deleted** as a result.  **Be Very Careful** when you run this playbook.

    ```
    $ ansible-playbook playbook/04-DestroyAzureResources.yml
    ```
