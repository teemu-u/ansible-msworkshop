# Getting started with Ansible on Microsoft Azure

Ansible interacts with the Azure resource manager’s REST APIs to manage infrastructure components using Python SDK provided by Microsoft, which requires credentials of an authorized user or service to work with Azure REST APIs. Ansible modules that interact with Azure resourcer manager are packaged as part of Ansible cloud modules.


### Prerequisites

Microsoft Azure Account: You will need a valid and active Azure account for the Azure labs. If you do not have one, you can sign up for a [free trial](https://azure.microsoft.com/en-gb/free/).

## Setting up the environment

We should start by installing Azure SDK on a host running Ansible:

**`$ pip install ansible[azure]`**


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

And paste in the information we gather previously:

```
[default] 
subscription_id=xxxxxx-xxxxx-xxxxxx-xxxx 
client_id=xxxxxx-xxxx-xxxx-xxxxx 
secret=xxxxxxxxxx 
tenant=xxxxx-xxxx-xxx-xxx-xxx
```
