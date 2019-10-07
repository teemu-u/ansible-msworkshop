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

Once logged in to [Azure portal](https://portal.azure.com/) click on **Azure Active Directory** (1)

![Image of Azure portal](https://github.com/teemu-u/ansible-msworkshop/blob/master/images/azureAD.png)

From Azure Active Directory select **User settings** (1)

![Image of Azure AD](https://github.com/teemu-u/ansible-msworkshop/blob/master/images/azureADuserSettings.png)

Check that the **App Registrations** setting is set to **Yes**.
When set to Yes, anyone can register an application and we can proceed. If set to No, either we need to be the admin in order to register an application or we should get it enabled by the admin of the Azure account:

![Image of Azure AD](https://github.com/teemu-u/ansible-msworkshop/blob/master/images/azureADappReg.png)

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

Markdown is a lightweight and easy-to-use syntax for styling your writing. It includes conventions for

```markdown
Syntax highlighted code block

# Header 1
## Header 2
### Header 3

- Bulleted
- List

1. Numbered
2. List

**Bold** and _Italic_ and `Code` text

[Link](url) and ![Image](src)
```

For more details see [GitHub Flavored Markdown](https://guides.github.com/features/mastering-markdown/).

### Jekyll Themes

Your Pages site will use the layout and styles from the Jekyll theme you have selected in your [repository settings](https://github.com/teemu-u/ansible-msworkshop/settings). The name of this theme is saved in the Jekyll `_config.yml` configuration file.

### Support or Contact

Having trouble with Pages? Check out our [documentation](https://help.github.com/categories/github-pages-basics/) or [contact support](https://github.com/contact) and we’ll help you sort it out.

# Service Principal Credentials
