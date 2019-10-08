# Getting started with Ansible on Microsoft Azure

Ansible interacts with the Azure resource manager’s REST APIs to manage infrastructure components using Python SDK provided by Microsoft, which requires credentials of an authorized user or service to work with Azure REST APIs. Ansible modules that interact with Azure resourcer manager are packaged as part of Ansible cloud modules.


### Prerequisites

Microsoft Azure Account: You will need a valid and active Azure account for the Azure labs. If you do not have one, you can sign up for a [free trial](https://azure.microsoft.com/en-gb/free/).

## Setting up the environment

We should start by installing Azure SDK on a host running Ansible:

**`$ pip install 'ansible[azure]'`**


Using the Azure Resource Manager modules requires authenticating with the Azure API. You can choose from two authentication strategies:

- [Active Directory Username/Password](#active-directory-username-password)
- [Service Principal Credentials](#create-azure-service-principal)


#### Create Azure service principal

First thing is to login to [Azure portal](https://portal.azure.com/).
After you've logged in to Azure using your Microsoft account you should create something called Azure service principal, which we will need for Azure Resource Manager modules to authenticate with the Azure API.

Azure service pricipal can be created using:
- Azure portal
- [Azure Cloud Shell](#via-azure-cloud-shell-or-cli)
- [Azure CLI](#via-azure-cloud-shell-or-cli)

#### via Azure portal

For creating the Service Principal through Azure portal, refer to [Microsofts Documentation](https://docs.microsoft.com/en-gb/azure/active-directory/develop/howto-create-service-principal-portal).

#### via Azure Cloud Shell or CLI

Once logged in to [Azure portal](https://portal.azure.com/) click on **Cloud Shell** (1) and select **Bash** (2)

![Image of Azure portal](https://github.com/teemu-u/ansible-msworkshop/blob/master/images/sp_cli.png)

When the Cloud Shell has been initialized type the following command by replacing <YourServicePrincipalNameHere> -value with naming of your choice:
  
**`$ az ad sp create-for-rbac --name AnsibleServicePrincipal`**

After running the command, it will output a JSON blob similar to this:

![Create service principal](https://github.com/teemu-u/ansible-msworkshop/blob/master/images/sp_cli_create.png)

Output should be copied to your text editor, as we will need the details going forward.

We will also need your Azure SubsciptionID. That can be fetched by running the following command on the Cloud Shell or CLI:

**`$ az account show`**

![Azure account info](https://github.com/teemu-u/ansible-msworkshop/blob/master/images/sp_cli_account.png)

Copy this information to your text editor as well.

#### Pulling all together

Once we have created all the credentials required by Ansible to use Azure APIs, we can create a credential file on the **Ansible control node**

First we'll have to create a directory for Azure credentials under the home directory:

`$ mkdir ~/.azure`

Then we'll create a file for the credentials:

`$ vim ~/.azure/credential`

And paste in the information we gathered previously:

```
[default] 
subscription_id=xxxxxx-xxxxx-xxxxxx-xxxx 
client_id=xxxxxx-xxxx-xxxx-xxxxx 
secret=xxxxxxxxxx 
tenant=xxxxx-xxxx-xxx-xxx-xxx
```

## Creating an Azure virtual machine

Before we jump into creating a Linux VM, we should know the following terms with respect to Azure:

* **Resource groups:** 
These are logical containers where Azure resources are deployed. We can deploy resources into a specific resource group for a specific use case. For example, we can have resource groups named production for all the production resources and staging for all the resources required for staging.

* **Image:** Azure Marketplace has various images for creating virtual machines. We can select an image of our choice, based on the use case, and create our virtual machine. In this, we will be using an Ubuntu Server image. There are four parameters linked to an image:

  * **Offer** defines the distribution type. For example, RHEL, Debian, or CoreOS.
  * **Publisher** refers to the organization that created the image. For example, Red Hat, credativ, or CoreOS.
  * **SKU** defines the instance of the image offered.
  * **Version** defines the version of an image SKU. When defining an image, we can set the version to Latest to select the latest SKU of an image.

* **Storage account:** The Azure Storage Account is a Microsoft-managed cloud service which provides scalable, highly available, and redundant storage. Azure Storage offers three kinds of storage:

  * **Blob storage:** This stores our files in the same way that they are stored on local computers. For example, images, PDFs, and so on. The storage can consist of a virtual hard disk (VHD) format, large log files, and so on. This kind of storage is used for creating disks attached to a virtual machine.
  * **File storage:** This is the network file share, which can be accessed through standard Server Message Block (SMB) protocol.
  * **Queue storage:** This is a messaging storage system. A single queue can store millions of messages, and each message can be up to 64 KB in size.

* **Location:** This is a region where we can deploy our resources in Azure Cloud. All of these will be deploying resources in the westus region. We will define this location as azure_region in vars -section of the playbook:

`azure_region: westeu`

To do this, follow the steps below:

1. Create a resource group for deploying resources:

```
- name: Create resource group
  azure_rm_resourcegroup:
     name: example
     location: "{{azure_region}}"
```

2. Create a storage account for our VM disk:

```
- name: Create a storage account
  azure_rm_storageaccount:
     resource_group: example
     name: examplestorage01
     type: Standard_LRS
     location: "{{azure_region}}"
```

3. Let’s create our first VM in Azure Cloud:

```
- name: Create VM with default settings
  azure_rm_virtualmachine:
    resource_group: example
    name: testvm10
    vm_size: Standard_D4
    storage_account: examplestorage01
    admin_username: ansible
    admin_password: Ansible123!
    image:
      offer: CentOS
      publisher: OpenLogic
      sku: '7.1'
      version: latest
```

**In step 1:** we created a resource group for deploying resources in our defined location. We will be using this resource group name in subsequent tasks to deploy all of the resources in this resource group.

**In step 2:** we created a storage account, which will be required for the OS disk of our virtual machine. Azure offers multiple storage types depending upon the use case and availability-Locally Redundant Storage (LRS), Geo-Redundant Storage (GRS), Zone-Redundant Storage (ZRS), or Read Access Geo-Redundant Storage (RAGRS). We are using Standard_RAGRS.

**In step 3:** we created our first virtual machine in Azure Cloud. This task will take care of setting up an admin user and setting a password for it. We have set the admin_username as ansible. Once our virtual machine is ready, we can connect to our virtual machine using the SSH protocol and the password associated with it, that we used in the task.

## Managing network interfaces

An Azure virtual machine can be attached to multiple network interface cards. With a network interface card, the virtual machine can access the internet and communicate with other resources both inside and outside Azure Cloud. In the, Creating an Azure virtual machine, while creating a virtual machine, it creates a default NIC card for the VM with default configurations. In this, we will see how to create, delete, and manage a NIC with custom settings.

Before we move ahead and create a NIC, we should be aware of the following term:

* **Virtual network:** This is a logical separation of the network within the Azure Cloud. A virtual network can be assigned a CIDR block. Any virtual network has the following attributes associated with it:
  * **Name**
  * **Address Space**
  * **Subnet**


To do this, follow the steps below:


1. Create a virtual network:

```
- name: Create Virtual Network
  azure_rm_virtualnetwork:
    name: vnet01
    resource_group: example
    address_prefixes_cidr:
       - "10.2.0.0/16"
       - "172.1.0.0/16"
  state: present 
```

2. Create a subnet:

```
- name: Create subnet
  azure_rm_subnet:
    name: subnet01
    virtual_network_name: my_first_subnet
    resource_group: example
    address_prefix_cidr: "10.2.0.0/24" 
    state: present 
```

3. Create a Network Interface Card:

```
- name: Create network interface card
  azure_rm_networkinterface:
    name: nic01
    resource_group: example
    virtual_network_name: vnet01
    subnet_name: subnet01
    public_ip: no 
    state: present 
  register: network_interface 
```

4. Access the private IP address:

```
- name: Show private ip
  debug:
    msg: "{{network_interface.ip_configuration.private_ip_address}}"
```


**In step 1:** we created a virtual network with the name vnet01 within the same resource group we used in the first, Creating an Azure virtual machine. Since we are using a resource group, the resources can pick the default location of the resource group. We have defined the CIDR network addresses as 10.2.0.0/24 and 172.1.0.0/16. We can also define multiple network addresses using YAML syntax.

**In step 2:** we created a subnet using the azure_rm_subnet module inside the virtual network vnet01.

**In step 3:** we created a network interface card using azure_rm_networkinterface and named it nic01. We specified the public_ip as no, which will ensure that Azure will not allocate any public IP address to the NIC, but also that NIC will be associated with a private IP from the subnet address space.

**In step 4:** we use the debug module to print the private IP address allocated to the network interface using the variable registered in step 3.


## Working with public IP addresses

In this, we will create a public IP address and associate it with the network interface.

Azure allocates the public IP address using one of two methods, static or dynamic. An IP address allocated with the static method will not change, irrespective of the power cycle of the virtual machine; whereas, an IP address allocated with the dynamic method is subject to change. In this, we will create a public IP address allocated with the Static method.

To do this, follow the steps below:

1. Create a public IP:

```
- name: Create Public IP address
  azure_rm_publicipaddress:
    resource_group: example
    name: pip01
    allocation_method: Static
    domain_name: test 
    state: present 
  register: publicip 
```

2. Display a public IP:

```
- name: Show Public IP address
  debug:  
     msg: "{{ publicip.ip_address }}" 
```

## Using public IP addresses with network interfaces and virtual machines

In this, we will create a virtual machine and network interface using the public IP we created. After creating a virtual machine with the public network interface, we will log into it using SSH.

1. Create a NIC with the existing public IP address:

```
- name: Create network interface card using existing public ip address
  azure_rm_networkinterface:
    name: nic02
    resource_group: example
    virtual_network_name: vnet01
    subnet_name: subnet01
    public_ip_address_name: pip01 
    state: present 
  register: network_interface02 
```

2. Create a virtual machine with the existing network interface:

```
- name: Create VM using existing virtual network interface card
  azure_rm_virtualmachine:
    resource_group: example
    name: first_vm 
    location: "{{azure_region}}"
    vm_size: Standard_D4
    storage_account: examplestorage01
    admin_username: ansible
    admin_password: Ansible123!
    network_interfaces: nic02 
    image:
      offer: CentOS
      publisher: OpenLogic
      sku: '7.1'
      version: latest 
```

2. Log into the VM using the public IP from the recipe, Working with public IP addresses:

`$ ssh ansible@13.33.23.24`

In the first two steps, we created a NIC with an existing public IP, created in the recipe Working with public IP addresses, and a virtual machine using that NIC.

In step 3, we logged into the virtual machine created with the public NIC.


## Managing an Azure network security group

In Azure, a network security group is an access control list (ACL), which allows and denies network traffic to subnets or an individual NIC. In this recipe, we will create a network security group with some basic rules for allowing web and SSH traffic and denying the rest of the traffic.

Since a network security group is the property of the network and not the virtual machine, we can use subnets to group our virtual machines and keep them in the same network security group for the same ACL.

1. To Create a network security group:

```
- name: Create network security group 
  azure_rm_securitygroup:
    resource_group: example
    name: mysg01
    purge_rules: yes
    rules: 
        - name: 'AllowSSH'
          protocol: TCP
          source_address_prefix: *
          destination_port_range: 22
          access: Allow
          priority: 100
          direction: Inbound 
        - name: 'AllowHTTP' 
          protocol: TCP 
          source_address_prefix: * 
          destination_port_range: 80 
          priority: 101 
          direction: Inbound 
        - name: 'AllowHTTPS' 
          protocol: TCP 
          source_address_prefix: * 
          destination_port_range: 443 
          priority: 102 
          direction: Inbound 
        - name: 'DenyAll' 
          protocol: TCP 
          source_address_prefix: * 
          destination_port_range: * 
          priority: 103 
          direction: Inbound 
```

2. Attach the subnet to the security group:

```
- name: Create subnet
  azure_rm_subnet:
    name: subnet01
    virtual_network_name: my_first_subnet
    resource_group: example
    address_prefix_cidr: "10.2.0.0/24" 
    state: present 
    security_group_name: mysg01 
```

3. Attach the NIC card to the security group:

```
- name: Create network interface card using existing public ip address and security group
  azure_rm_networkinterface:
    name: nic02
    resource_group: example
    virtual_network_name: vnet01
    subnet_name: subnet01
    public_ip_address_name: pip01 
    security_group_name: mysg01 
    state: present 
  register: network_interface02 
```
