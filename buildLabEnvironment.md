# buildLabEnvorinment

In this project centered around Terraform, you're going to use VS Code and deploy an
Azure Linux machine along with virtual network, subnet, security groups, etc. that will serve as
a remote development environment that you can log into directly from VS Code. We'll utilize
several Terraform tools and functions such as Terraform State, Format, Replace, Console,
Variables, Conditionals, and more. We'll also use Azure custom data and a provisioner to bootstrap
the virtual machine with Docker and add its connection information to your VS Code SSH
configuration file, allowing you to modify it for your own projects. Since we'll make heavy use of
documentation and highlight the process behind developing a brand new Terraform deployment,
you'll have the skills you need to dive deeper into Terraform and add this extremely in-demand
skill. Now We're going to open a new project in VS Code

## VSCode Setup
Install the Azure CLI, connect to our Azure account, and install a Terraform extension.  
- Choose the operating system you're using and go ahead and install: 
[Install Azure CLI on Windows](https://learn.microsoft.com/en-us/cli/azure/install-azure-cli-windows?tabs=azure-cli)  
  `` winget install -e --id Microsoft.AzureCLI ``  

  I'm going to go ahead and open a terminal and run az login to get logged into my account. You can also validate by running ``az account show``  

- Add the Terraform extension
[HashiCorp Terraform](https://marketplace.visualstudio.com/items?itemName=HashiCorp.terraform)  

Open a new folder where we'll store our files, and I'm going to create a new folder. And we'll just call that Terraform Azure, just like
so. Go ahead and open that, select folder, and all set. And once again, I'll just make sure that Terraform Provider Init.

First thing I'm going to do is just create a new file. And we're going to call that main.tf, just like so. Go ahead and open that. And now I'm just going to pull that out of the way.

Now, at [Terraform Registry](https://registry.terraform.io/providers/hashicorp/azurerm/latest) we've got our Azure provider. And what this provider does is allows Terraform to communicate with the Azure API. That's how Terraform knows
how to deploy resources.

So as you can see, if you look down at [Azure Provider](https://registry.terraform.io/providers/hashicorp/azurerm/latest/docs), you'll see authenticating to Azure.
