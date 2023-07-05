# Terraform Config & Test Deployment
Open a new folder where we'll store our files, and I'm going to create a new folder. And we'll just call that Terraform Azure, just like
so. Go ahead and open that, select folder, and all set. And once again, I'll just make sure that Terraform Provider Init.

First thing I'm going to do is just create a new file. And we're going to call that ``main.tf``, just like so. Go ahead and open that. And now I'm just going to pull that out of the way.

Now, at [Terraform Registry](https://registry.terraform.io/providers/hashicorp/azurerm/latest) we've got our Azure provider. And what this provider does is allows Terraform to communicate with the Azure API. That's how Terraform knows
how to deploy resources.

So as you can see, if you look down at [Azure Provider](https://registry.terraform.io/providers/hashicorp/azurerm/latest/docs), you'll see authenticating to Azure.

So this is our configuration that we're going to use for the provider.
```
terraform {
  required_providers {
    azurerm = {
      source  = "hashicorp/azurerm"
      version = "=3.0.0"
    }
  }
}
# Configure the Microsoft Azure Provider
provider "azurerm" {
  features {}
}

```

Now, as you can see, we've got Terraform, and then we've got required providers. So any of the providers we're going to use are specified in this block. So we've got the Azure RM provider.
The source here is HashiCorp Azure RM. And then we've got a version. Now, this is actually freezing this version to 3.0.0. Your version may be different, and that's perfectly fine. Of course,
if something does break, you can always revert to this version. Now, below that, we then actually specify the provider.  
Up here is indicating which providers we'll be using within Terraform. And then here is actually how we will do the
connection and specify any other options that we may want to specify.

Go to the [Argument Reference](https://registry.terraform.io/providers/hashicorp/azurerm/latest/docs#argument-reference) to notice wich are required.

Now, you can just copy and paste that in ``main.tf``

At our terminal, first we'll run a command: ``Terraform FMT``. And that stands for Terraform Format, And as you can see, that kind of cleaned up our code a little bit, removed some indentations, things like that. So it's a good habit to run Terraform Format anytime you are working with any type of files, especially before you commit to your repositories.

let's go to the next step and that is a command: ``Terraform init``, just like so. As you can see, we're initializing the backend and the backend is a local backend, which means that our Terraform state where everything is stored will be stored here.

<img width="719" alt="image" src="assets/tfInit.png">


We've got the.terraform directory here.
This is actually where the provider is stored. And this is a compiled file. So this is actually a compiled go file or in Windows case, an exe file, as you can see. And that is what is used to communicate with the API of Azure. And then what we have is our Terraform lock.hcl. And as you can see, this can constrain the version that we use. You typically want to commit this to your repository. So even if this version isn't set, this lock.hcl will maintain that version to ensure that your code always runs. Now, if you decide to upgrade that provider, you may want to just delete this file or modify the version. So just keep that in mind.

So now let's take a look at the resource group documentation: [azurerm_resource_group](https://registry.terraform.io/providers/hashicorp/azurerm/latest/docs/resources/resource_group). Now, as you can see, we've got some notes here. None of these are going to affect us, but do be aware that whenever you see notes in the documentation like this, it's probably a great idea to read them, as sometimes they will indicate some very, very big issues that you might encounter and how to solve them.
```
resource "azurerm_resource_group" "example" {
  name     = "example"
  location = "West Europe"
}
```
As you can see, we've got a resource here. So just like we have provider here, this is a resource. And this resource is ``azurerm_resource_group``. Now, this cannot change. This indicates the type of resource that you are deploying.
So you want to make sure you use the right name for that resource as Terraform will have no idea what you're trying to deploy if you don't use the right name.

Now, this is an alias --> ``"example"``. The following here can be anything you want. For the most part, you have to adhere to some grammar rules, but for the most part, it can be just about anything you want. And this is how you will reference this resource within Terraform.

```
resource "azurerm_resource_group" "k8sLabs-resources" {
  name     = "rg-k8sLabs"
  location = "West US 2"
  tags = {
    environment = "dev"
  }
}
```

So we've got a resource, Azure RM resource group. And let's just call that ``k8sLabs-resources``, just like so. Now the name, since this is actually in Azure, we're going to make that a little more specific in this case. We'll just call it ``rg-k8sLabs``. And this is also to illustrate that these can be different.

Now let's run a ``Terraform FMT`` once again, and then let's run a ``Terraform plan``, just like so.

<img width="719" alt="image" src="assets/tfPlan.png">

And what this is doing is showing us what is going to be built if we run a Terraform apply. So as you can see, we've got our plus signs for create. There are other signs, but right now, we're not changing anything. So you can see that we're creating this new resource. And then under that, you've got the arguments that we're passing in. And as you can see, we've got the ID known after apply, which means we will know that as soon as this is deployed to Azure.

## Deploy a Resource Group

So let's go ahead and deploy that: run ``Terraform apply`` and hit enter
<img width="719" alt="image" src="assets/tfApply.png">

Now in the future, we will use the auto approve flag just to skip that because we are running a plan between all of this. So we don't need to get another confirmation. But generally
speaking, when running from the CLI in production, it's probably a good idea to have this extra step just to ensure you don't make any mistakes.

## Deploy a Virtual Network
We're going to set the groundwork for our deployment by deploying a virtual network. In order to do that, we'll also need to learn how to reference attributes of other resources.

So we've got the documentation at [azurerm_virtual_network](https://registry.terraform.io/providers/hashicorp/azurerm/latest/docs/resources/virtual_network), which is where we always start.
Now there are some notes here and eventually these may be deprecated, but for right now they are fairly important. Some of the demos and examples are going to show inline subnets. We're actually not going to do that. We're going to specify our subnet as a separate resource. That gives us more control over the resource itself, how it relates to things, its dependencies, and lots of other little benefits that you'll see as you start to build your own larger deployments.

```
resource "azurerm_virtual_network" "k8sLabs-vnet"{
    name = "vnet-k8sLab"
    resource_group_name = azurerm_resource_group.k8sLabs-resources.name
    location = azurerm_resource_group.k8sLabs-resources.location
}
```

Now one thing you can take a look at here, the resource group name references the resource group. As you can see, azurerm_resource_group.k8sLabs-resources.name. So it is
resource azurerm_resource_group.k8sLabs-resources.name. So we've got the resource type, the resource alias that we provided to it, and then one of the attributes, all using a dot syntax. And then the same thing is specified with location.

Now you could also just specify ``k8sLabs-resources`` if you wanted to, but you don't want to define the same thing in multiple places, because if we were to change ``k8sLabs-resources``,
we'd have to change it in every resource that references it. Now, another reason why it's important to reference resources by their resource address instead of by hard coding them is because this creates an implicit dependency. Basically, now this network will not deploy before this resource group has been created, and the same whenever we destroy. It will not destroy this resource group until this virtual network is destroyed. So by referencing this resource group from the virtual network, we tell Terraform that this virtual network is dependent on this resource group.

And then what we want to do is specify the address space, also known as a citer block. The address space is, and this is actually a list. You can specify multiple address spaces here,
but we're just going to use one. We're going to use ``192.168.0.0/27``. And if you're not familiar with subnets, I strongly recommend you review them. 

```
resource "azurerm_virtual_network" "k8sLabs-vnet" {
  name                = "vnet-k8sLab"
  resource_group_name = azurerm_resource_group.k8sLabs-resources.name
  location            = azurerm_resource_group.k8sLabs-resources.location
  address_space       = ["192.168.0.0/27"]

  tags = {
    envorinment = "dev"
  }
}
```
I think that's everything. Let's go ahead and run our ``Terraform format``. And after that, let's run our ``Terraform plan``.

<img width="719" alt="image" src="assets/tfPlanVnet1.png">

Let's go ahead and get this network deployed. So Terraform apply. And as I said, we're just going to use a ``Terraform apply -auto-approve`` here. And if you want to see everything else, you can actually do a ``-help``.
<img width="719" alt="image" src="assets/tfApplyVnet1.png">

# Test Terraform State

You may have noticed this ``Terraform.state`` file and this ``Terraform.tfstate.backup`` file. Under very, very, very few circumstances should you ever modify
this state manually. There are specialized commands for making state changes and things.

<img width="719" alt="image" src="assets/tfState1.png">

So for the most part, one, you probably shouldn't have direct access to it. It should be stored remotely. But two, don't go around and start changing things in this state. It's not the
right way to manage your resources.

It specifies the Terraform version. It's got a serial number, a lineage, a lot of really good information to help you understand which state you're dealing with. And every time the state is modified, it actually creates a backup.

So once we run a Terraform apply or destroy again, the Terraform state will be backed up with this Terraform state information and a new ``Terraform.tfstate`` will be created. So as you look through our state here, you can see a lot of very familiar attributes.

Right here, we've got our Azure RM resource group with a name. Then you've got the provider there also. Then under that, we've got ID, location, name, tags, et cetera, et cetera. And as you scroll
down, you'll see we also have our virtual network here. So that's pretty cool. And all of that, of course, is exactly what was deployed within Azure. 

Because again, typically speaking, you're not going to store your state locally. You're going to send it off to some sort of other storage. Some people with AWS store it in S3. You can store it in Terraform's cloud. You can store it in services such as Spacelift and more.

let's go ahead and run a ``Terraform state list``

<img width="719" alt="image" src="assets/tfStateList1.png">

So that's an easy way to see all of the resources we have. And then if we want more details about what's in those resources, we can run a ``Terraform state show <name_of_recource>``.

<img width="719" alt="image" src="assets/tfStateShow1.png">

So now we get the information from that resource.

And now if we want to see the entire state, we actually just run a ``Terraform show``. So keep that in mind. ``Terraform show`` will show you all of the state, as you can
see here. And then ``Terraform state show <name_of_recource>`` is how you view specific resources, just like so.

<img width="719" alt="image" src="assets/tfShow1.png">

So if we take a look at the [Terraform docs here](https://developer.hashicorp.com/terraform/language/state), there's actually a good bit that you can read through. I strongly suggest you get very, very familiar with it.

# Tesing Terraform Destroy & Rebuild

<img width="719" alt="image" src="assets/tfDestroyDocs1.png">

if we take a look at the [Terraform docs: Command: destroy](https://developer.hashicorp.com/terraform/cli/commands/destroy), we can actually see that a few things have changed since Terraform came out. Originally, Terraform destroy was the proper way to do this. And it still works. As you can see Terraform destroy followed by options. This is an alias for Terraform apply dash destroy. Now you can also run a Terraform plan dash destroy, which of course will not perform the destruction. It will just let you know what it's going to do.

Now, of course, what Terraform destroy does here is destroy everything that you've created

But let's take a look at a few things first. First, what I'm going to do is just run a ``Terraform state list``: We have two objects here. As you know, we've deployed both of those

<img width="719" alt="image" src="assets/tfStateList1.png">

And then if I run a ``Terraform plan -destroy``, as you can see, we've got  **zero to add, zero to change, and two to destroy** . And if you scroll up, you can see that everything has this
little red dash next to it indicating that it's going to be deleted. So if you ever see that red dash, that means that that resource is going to be deleted.

<img width="719" alt="image" src="assets/tfPlanDestroy1.png">

So I'm going to run ``Terraform apply -destroy``, or you can just run Terraform destroy if you'd like. Let's go ahead and run that.

<img width="719" alt="image" src="assets/tfApplyDestroy1.png">

As you can see, it took it a while to delete that resource group. Let's take a look at the ``terraform.tfstate`` and ``terraform.tfstate.backup`` again. Here is the tfstate. As you can see,
there is nothing in here anymore. And the serial has incremented to six. Now if we take a look at the tfstate.backup, here you go. This was our old tfstate. So now if you ever needed to replace the
state or you made a big mistake or something went wrong with the state, somehow the state got corrupted, you have this backup right here. You can delete this tfstate and rename this one

# Deploy a Subnet
And we're going to deploy a subnet into our virtual network. This way we have an IP address space that can be used by our virtual machines.

So as usual, let's take a peek at our [azurerm_subnet](https://registry.terraform.io/providers/hashicorp/azurerm/latest/docs/resources/subnet) resource.

It's got a very similar note here that we saw on our virtual network resource that indicated that these subnets can be deployed in line with the virtual network, but in our case and most other cases, it's better to deploy it separately from the virtual network.

```
resource "azurerm_subnet" "k8sLabs-subNnet-control" {
  name                 = "subNnet-k8sLab-control"
  resource_group_name  = azurerm_resource_group.k8sLabs-resources.name
  virtual_network_name = azurerm_virtual_network.k8sLabs-vnet.name
  address_prefixes     = ["192.168.0.0/28"]
}

resource "azurerm_subnet" "k8sLabs-subNnet-worker" {
  name                 = "subNnet-k8sLab-worker"
  resource_group_name  = azurerm_resource_group.k8sLabs-resources.name
  virtual_network_name = azurerm_virtual_network.k8sLabs-vnet.name
  address_prefixes     = ["192.168.0.16/28"]
}
```

You may run ``Terraform plan`` to see what gonna be happening. As you can see, those were populated just fine by referencing the name of the subnet. So go ahead and run a ``Terraform apply -auto-approve``.

<img width="719" alt="image" src="assets/tfApplySubnet1.png">

# Deploy a Security Group

First, let's take a look at [azurerm_network_security_group](https://registry.terraform.io/providers/hashicorp/azurerm/latest/docs/resources/network_security_group). And as you can see, we have another note. Now with the security group, you can actually define your **security group rules** in line, just like this, or as separate resources. Now to keep things consistent, we're going to deploy our security group rule as a separate resource, instead of adding it in line. This will make maintaining our security group rules a lot easier and allow us to make deploys and changes without any real unexpected consequences.

```
resource "azurerm_network_security_group" "k8sLabs-nsg" {
  name                = "nsg-k8sLab"
  location            = azurerm_resource_group.k8sLabs-resources.location
  resource_group_name = azurerm_resource_group.k8sLabs-resources.name

  tags = {
    environment = "dev"
  }
}

```
I'm going to select the [azurerm_network_security_rule](https://registry.terraform.io/providers/hashicorp/azurerm/latest/docs/resources/network_security_rule) resource. And if you take a look, Azure RM network security rule example, you've got all of these items here that need to be filled in. And then you reference the resource group name and the network security group name.

```
resource "azurerm_network_security_rule" "k8sLabs-nsgRule" {
  name                        = "k8sLabs-AllTraffic"
  priority                    = 100
  direction                   = "Inbound"
  access                      = "Allow"
  protocol                    = "Tcp"
  source_port_range           = "*"
  destination_port_range      = "*"
  source_address_prefix       = "*"
  destination_address_prefix  = "*"
  resource_group_name         = azurerm_resource_group.k8sLabs-resources.name
  network_security_group_name = azurerm_network_security_group.k8sLabs-nsg.name
}
```
Let's run ``Terraform plan`` to see if we have any typo error. As you can see, those were populated just fine by referencing the name of the subnet. So go ahead and run a ``Terraform apply -auto-approve``.

<img width="719" alt="image" src="assets/tfApplyNsgRule1.png">

# Deploy a Security Group Associations

We're going to associate our brand new security group with our subnet so it can be actually used to protect it.

As you can see here, we have our [azurerm_subnet_network_security_group_association](https://registry.terraform.io/providers/hashicorp/azurerm/latest/docs/resources/subnet_network_security_group_association), Luckily, it's not that long of a resource. It's actually very, very straightforward and very simple.

```

resource "azurerm_subnet_network_security_group_association" "nsgRuleAssociation-k8sLabs-AllTraffic-controlSubnet" {
  subnet_id                 = azurerm_subnet.k8sLabs-subNnet-control.id
  network_security_group_id = azurerm_network_security_group.k8sLabs-nsg.id
}
resource "azurerm_subnet_network_security_group_association" "nsgRuleAssociation-k8sLabs-AllTraffic-workerSubnet" {
  subnet_id                 = azurerm_subnet.k8sLabs-subNnet-worker.id
  network_security_group_id = azurerm_network_security_group.k8sLabs-nsg.id
}

```

Let's run ``Terraform plan`` to see if we have any typo error. As you can see, those were populated just fine by referencing the name of the subnet. So go ahead and run a ``Terraform apply -auto-approve``.

<img width="719" alt="image" src="assets/tfApplyNsgRuleAssociation1.png">

# Deploy a Public IP

We're going to give our future virtual machine a way to the internet by creating a public IP. So let's get started. All right, so here we are with the [azurerm_public_ip](https://registry.terraform.io/providers/hashicorp/azurerm/latest/docs/resources/public_ip).

```
resource "azurerm_public_ip" "k8sLabs-PIP-Control" {
  name                = "k8sLabs-PIP-Control"
  resource_group_name = azurerm_resource_group.k8sLabs-resources.name
  location            = azurerm_resource_group.k8sLabs-resources.location
  allocation_method   = "Dynamic"

  tags = {
    environment = "dev"
  }
}
```
Let's run ``Terraform plan`` and run a ``Terraform apply -auto-approve``.

<img width="719" alt="image" src="assets/tfApplyPIPControl1.png">

Go ahead and run our ``Terraform state list``, I'm going to do is grab this IP **azurerm_public_ip.k8sLabs-PIP-Control**

<img width="719" alt="image" src="assets/tfStateListPIPControl1.png">

When you run ``Terraform state show azurerm_public_ip.k8sLabs-PIP-Control`` will notice there if no Ip address assigned, this is because this resource is not currently attached. But once we deploy our other resources,
we will be able to extract that IP address and use it in the future.

# Deploy a Virtual Network Interface

This NIC will receive its public IP address from the IP address we just created.
So here is our network interface: [azurerm_network_interface](https://registry.terraform.io/providers/hashicorp/azurerm/latest/docs/resources/network_interface)

As you can see, it kind of gets our usual suspects, the name, location, resource group name. We then get the IP configuration under that. And we've got a name for that, subnet ID and private IP address allocation. But as you can see, we don't have public IP address listed here, which is something that is very important to us, so we'll add this argument to our new definition as ``public_ip_address_id = azurerm_public_ip.k8sLabs-PIP-Control.id``

```

resource "azurerm_network_interface" "nic01-k8sLabs-Control" {
  name                = "nic01-k8sLabs-Control-VM"
  location            = azurerm_resource_group.k8sLabs-resources.location
  resource_group_name = azurerm_resource_group.k8sLabs-resources.name

  ip_configuration {
    name                          = "ControlInternal"
    subnet_id                     = azurerm_subnet.k8sLabs-subNnet-control.id
    private_ip_address_allocation = "Dynamic"
    public_ip_address_id          = azurerm_public_ip.k8sLabs-PIP-Control.id
  }
}

```

So if we run a ``Terraform state list``, we've got our nick there. And if we run a ``Terraform state show``, add that to the end. As you can see, we have a private IP address right there, but I'm not seeing a public IP address yet.



