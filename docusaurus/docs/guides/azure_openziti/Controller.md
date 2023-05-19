---
sidebar_position: 20
sidebar_label: Controller
title: Controller Config 
---

# 1.0 Configure the controller
## 1.1 Create a VM on Azure Cloud
Login to the Azure console, create a resource from the Azure services on the upper right hand side.

![Diagram](/img/azure/create1.jpg)

On the "Create" screen, Choose "**Ubuntu Server 22.04 LTS**".From the project details select the subscription to manage deployed resources and costs. Use resource groups like folders to organize and manage all your resources. In the instance details put the VM name, select the region, Leave default avilability options,Security type(Standard). Leave the selected image Ubuntu Server 22.04 LTS x64 Gen2
For the Size, choose the appropriate size for your application.  For this guide, Standard_B2s(2CPU,4 GB) size was used. 
![Diagram](/img/azure/create2.jpg)

Next, choose a ssh-key to login to the VM. We can put any username. Use the existing ssh key stored in the Azure. If ssh key was not store create a new key and download the private key with will use to login the VM. For inound port rule select the ssh. We can add extra port based on open ziti requrement. Leave everything default in disks. then **Create VM**
![Diagram](/img/azure/create3.jpg)
In the networking tab select the Virtual network name and select the subnet and select the auto genrated the public IP name. Leave the NIC network security group "none". Rest leave unchecked default. Leave management, monitoring,advanced,tags default.
![Diagram](/img/azure/create4.jpg)
Check the validation test. Once it passed then **Create**
![Diagram](/img/azure/create5.jpg)

## 1.2 Login and Setup Controller
Once the VM is created, we can get the IP address of the VM from the Resources screen. Login to the VM by using defined user "usename", private sshkey and IP address:
```bash
ssh -i <private_key> "username"@<ip>
```

Then follow the [Host OpenZiti Anywhere](/docs/learn/quickstarts/network/hosted/) to setup the controller. You must replace the EXTERNAL_DNS with the following command before running the quickstart.
 
**export EXTERNAL_DNS="$(curl -s eth0.me)"**

This ensures the Controller setup by the quickstart is advertising the external IP address of the VM.

## 1.3 Setup Ziti Administration Console (ZAC) 
**Optional**

ZAC provides GUI for managing the OpenZiti network. If you prefer UI over CLI to manage network, please following the [ZAC Setup Guide](/docs/learn/quickstarts/zac/) to setup ZAC before going to the next section.

## 1.4 Helpers

Following helpers are needed to complete the guides for router and services.

### 1.4.1 Add Environment Variables Back to the Shell
Source the environment variables when you log back in the shell
```bash
source ~/.ziti/quickstart/$(hostname -s)/$(hostname -s).env
```

If the environment variables are sourced correctly, you can do the following to check:
```bash
echo $ZITI_HOME
```
**Output:**
```
/root/.ziti/quickstart/OMSINER
```
 
### 1.4.2 Change Ziti edge admin password
Find the Current admin edge login password of controller (if you forget the password):
```bash
grep "export ZITI_PWD" ~/.ziti/quickstart/$(hostname -s)/$(hostname -s).env
```
Or if you have environment variable setup correctly:
```bash
echo $ZITI_PWD
```
To update the passwd
```bash
ziti edge update authenticator updb -s
```
**Important:** if you change the password, you must update the passwd (ZITI_PWD) in the "~/.ziti/quickstart/$(hostname -s)/$(hostname -s).env" file. 

### 1.4.3 Some useful command for the Router
** login the CLI**
```bash
zitiLogin
```

**Verify ER status**
```
ziti edge list edge-routers
```

**Delete the ER**
```
ziti edge delete edge-routers $ROUTER_NAME
ziti edge delete edge-routers $ROUTER_ID
```

**Update the ER**
```
ziti edge update edge-router $ROUTER_NAME [flags]
ziti edge update edge-router $ROUTER_ID [flags]
```
example to update attributes: 
```
ziti edge update edge-router $ROUTER_NAME -a private
```