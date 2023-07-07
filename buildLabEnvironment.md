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

or from Powershell

  `` Invoke-WebRequest -Uri https://aka.ms/installazurecliwindows -OutFile .\AzureCLI.msi; Start-Process msiexec.exe -Wait -ArgumentList '/I AzureCLI.msi /quiet'; rm .\AzureCLI.msi``

  I'm going to go ahead and open a terminal and run az login to get logged into my account. You can also validate by running ``az account show``  

- Add the Terraform extension
[HashiCorp Terraform](https://marketplace.visualstudio.com/items?itemName=HashiCorp.terraform)  

