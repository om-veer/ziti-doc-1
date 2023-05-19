---
sidebar_position: 30
sidebar_label: Router
title: Create new router
---
# 2.0 Configure new router using open ziti

## 2.1 Create a VM on Azure cloud
Please follow **Section 1.1** of the [Controller Guide](/docs/guides/azure_openziti/Controller/) to setup a VM to be used as Router. 

## 2.2 Login and Update the repo and apps on VM
Once the VM is created, we can get the IP address of the droplet from the Resources screen. Login to the VM by using user "username" and IP address:
```bash
ssh username@<ip> 
If you are using ssh key method then use the following commmand
ssh -i "privatekey.pub username@ip_address
```

### 2.2.1 apt update
```bash
sudo apt update
sudo apt upgrade
```

### 2.2.2 Download ziti_router_auto_enroll binary
**ziti_router_auto_enroll** is an easy way to setup your router automatically.
```bash
wget https://github.com/netfoundry/ziti_router_auto_enroll/releases/latest/download/ziti_router_auto_enroll.tar.gz
tar xf ziti_router_auto_enroll.tar.gz
```
You should have a file **ziti_router_auto_enroll** under the directory.

For detail info on ziti_router_auto_enroll, please checkout the [github page](https://github.com/netfoundry/ziti_router_auto_enroll)

## 2.3 Create and setup Router directly on router VM

You can setup the router directly on the router VM with one command if you did not block your controller's edge-management port. At this time, the quickstart for setting up controller does not separate edge-management port from edge-client port, so the edge-management port has to be open. You may continue this section if you know your controller's management password, Fabric Port (default 8440) and Management Port (default 8441).

You can also choose to create router on the controller and then register with the jwt file (created when creating the router) on the router. The procedure for this is detailed in **section 2.4**

### 2.3.1 Info needed for creating Router
In order to create the Router, the VM needs to contact controller. We need the following information before we can continue:
- Controller IP
- Controller Fabric Port: On the controller, issue this command **echo $ZITI_CTRL_PORT**
- Controller Management Port: On the controller, issue this command **echo $ZITI_EDGE_CONTROLLER_PORT**
- Controller Passwd: On the controller, issue this command **echo $ZITI_PWD**
- Router Name: Name for this router

### 2.3.2 Info gathered for creating Router
Here is information I gathered from previous step:
- Controller IP: 20.219.123.248
- Controller Fabric Port: 8440 **(default value if following controller setup guide)**
- Controller Management Port: 8441 **(default value if following controller setup guide)**
- Controller Passwd: Test@123
- Router Name: OM-ER-ME

We are also going to create the router without healthcheck section and metrics, so the following two options will be used to create the router:
- --disableHealthChecks
- --disableMetrics

If you choose to explore these two functionalities, you can remove the options (from command line) when creating router.

### 2.3.3 Create the Router with link listener
```
./ziti_router_auto_enroll -f -n --controller 20.219.123.248 --controllerFabricPort 8440 --controllerMgmtPort 8441 --adminUser admin --adminPassword Test@123 --assumePublic --disableHealthChecks --disableMetrics --routerName OM-ER-ME 
```

**Alternative way of creating router**

Instead of passing parameters through the command line to create routers, the parameters can be specified via environmental variables. Here is example on how to accomplish that.
```
export CONTROLLER="20.219.123.248"
export CONTROLLERFABRICPORT="8440"
export CONTROLLERMGMTPORT="8441"
export ADMINUSER="admin"
export ADMINPASSWORD="Test@123"
./ziti_router_auto_enroll -f -n --assumePublic --disableHealthChecks --disableMetrics --routerName OM-ER-ME
```

### 2.3.4 Create the Router with link listener and tunneler
```
./ziti_router_auto_enroll -f -n --controller 20.219.123.248 --controllerFabricPort 8440 --controllerMgmtPort 8441 --adminUser admin --adminPassword Test@123 --assumePublic --disableHealthChecks --disableMetrics --autoTunnelListener --routerName OM-ER-ME
```

### 2.3.5 Create the Router with edge listener only (no link listener)
```
./ziti_router_auto_enroll -f -n --controller 20.219.123.248 --controllerFabricPort 8440 --controllerMgmtPort 8441 --adminUser admin --adminPassword Test@123 --disableHealthChecks --disableMetrics --routerName OM-ER-ME 
```
**output**
```
Failed to stop ziti-router.service: Unit ziti-router.service not loaded.
Writing jwt file: OM-ER-ME_enrollment.jwt
Version not specified, going to check with controller
Found version 0.27.9
Downloading file: https://github.com/openziti/ziti/releases/download/v0.27.9/ziti-linux-amd64-0.27.9.tar.gz
Downloading: 100%|███████████████████████████████████████████████████████████████████████████████████| 115M/115M [00:06<00:00, 19.2MiB/s]
Successfully downloaded file
Starting binary install
Installing service unit file
Creating config file
Starting Router Enrollment
Successfully enrolled Ziti
Service ziti-router.service start successful.
Created symlink /etc/systemd/system/multi-user.target.wants/ziti-router.service → /etc/systemd/system/ziti-router.service.
Service ziti-router.service enable successful.
```
### 2.3.6 Create the Router with edge listener and tunneler
```
./ziti_router_auto_enroll -f -n --controller 20.219.123.248 --controllerFabricPort 8440 --controllerMgmtPort 8441 --adminUser admin --adminPassword Test@123 --disableHealthChecks --disableMetrics --autoTunnelListener --routerName OM-ER-ME1
```
**output**
```
Service ziti-router.service stop successful.
Writing jwt file: OM-ER-ME1_enrollment.jwt
Starting Ubuntu DNS setup
Service systemd-networkd restart successful.
Service systemd-resolved restart successful.
Version not specified, going to check with controller
Found version 0.27.9
Downloading file: https://github.com/openziti/ziti/releases/download/v0.27.9/ziti-linux-amd64-0.27.9.tar.gz
Downloading: 100%|███████████████████████████████████████████████████████████████████████████████████| 115M/115M [00:05<00:00, 22.5MiB/s]
Successfully downloaded file
Starting binary install
Installing service unit file
Service ziti-router daemon-reload successful.
Creating config file
Starting Router Enrollment
Successfully enrolled Ziti
Service ziti-router.service start successful.
Service ziti-router.service enable successful.
```
## 2.4 Creating Router on Controller first
**If you already go through the procedure in section 2.3, you can skip to section 2.5**

You can create the router on the controller first then register the router on the router VM.

### 2.4.1 Creating Router on the controller Using ZAC
**If you prefer to create router using CLI, you can jump to next section.**

**In order to complete the procedures in this section, you need to install ZAC first and have access to the controller using the ZAC. If you have trouble using ZAC, you can use the CLI procedures to create router.**

From the ZAC welcome screen, choose the **ROUTERS**

![Diagram](/img/azure/zac1-router.jpg)

Click on **+** to bring up the **CREATE EDGE ROUTER** widget. The **NAME** of Router is required, and it has to be unique. Also choose whether you want the tunneler to be enable or not on the router.  Enter other optional fields and hit **SAVE**

![Diagram](//img/azure/zac2-router.jpg)

If the router is created successfully, you will be back to the **MANAGE EDGE ROUTERS** screen.  From the list of edge routers, you will see the **JWT** icon on the newly created router. You need this JWT for the registration.

![Diagram](/img/azure/zac3-router.jpg)

Click on the **JWT ICON**, the JWT will be downloaded to your machine. On Chrome browser, the downloaded file will appear on the bottom right corner of the browser like picture below.

![Diagram](/img/azure/zac4-router.jpg)

Open the JWT file, and copy the content.  Now you are ready for the registration.

![Diagram](/img/azure/zac5-router.jpg)


### 2.4.2 Creating Router on the controller Using CLI
**If you already created router using ZAC, you can skip ahead to register the router.** Otherwise, this section provides CLI commands to create routers on the controller.

**login to controller**
login to CLI first
```bash
zitiLogin
```

To create an edge router (no tunneler)
```bash
ziti edge create edge-router OM-ER-SI -o OM-ER-SI.jwt
```
**output**
```
New edge router OM-ER-SI created with id: BzUtjC7E.
Enrollment expires at 2023-04-07T03:52:03.997Z
```

To create an edge router with tunneler
```bash
ziti edge create edge-router OM-ER-SI -t -o OM-ER-SI.jwt
```

Cat out the content of the jwt file. We will need to use the jwt to register router
```
# cat  OM-ER-SI.jwt
eyJhbGciOiJSUzI1NiIsInR5cCI6IkpXVCJ9.eyJlbSI6ImVyb3R0IiwiZXhwIjoxNjgzNjQyMjg0LCJpc3MiOiJodHRwczovLzIwLjIxOS4xMjMuMjQ4Ojg0NDEiLCJqdGkiOiI1NzdkYmU1My04MGM3LTQ0YTMtODVlZS1iMTcxMjcyOGEwYjciLCJzdWIiOiIyTi44VkEtbFdyIn0.IESzkMxMgAyjGzjq1tbzIjBOLuzEAxSwBrTF9f5BrDbo_dDUrouYLjvSxGrPUvLgkFe9MoQrr_J5DZKkiZ684seLXJXcq8ItvfKljccrknf1JoDDEC1nfJd249MSdJ22LWBwf_Rn-hvmq6UIU_N493KdCDDlWA91evodlgyHvbDqNOm-p0RjCj3ral7wV-TSOSxHj3tex9JCIEaryLEAYAElm6SGhABFGeBxPsNd-93hcCShrgxHx1aW4mQfj-B2qRgJN3LlSUNOpCOC33MEqP3kwvUTMyaspQoYMfW-sLOZJLTEw5NLMbQ5WPq2k90mkIWnE7jMypQa-tJQNIk_wY53YSwJV-cT4T7g2gg1r1ovD0y9d4KpBXyAHr-CI6qhHYuBY31vCz38pxk-eZh-2pjIJx0vJjsG8DzvMxg0DmUeTFgXpH_E-N5nwoZ38TYLCgsM6pQNXk09ZLDflKUmZgSaaYPuf_90CVCJcQkfSY1g6k8A4cey_u9i2f2BkAy8LoCs2NmW6jZM_qIJLp3TYh3KGBkFWiG4BUnGmfSxLlgxnFcRGbpvf67RdRlBA24J3JEU3rdenVVsADunmReElylnEcBvjpW7cLfitiHH4gafFt3qeu0cLeD2McGoMxUZ4ML-tMouF9KtQnxF4X_PeqxDZ6KRvXDXg6_1tmULzPw
```

We also need the management port (default 8441) and fabric port (default 8440) of the controller to register the router
```
# echo $ZITI_EDGE_CONTROLLER_PORT
8441
# echo $ZITI_CTRL_PORT
8440
```

### 2.4.3 Register the Router with link listener
**Perform this on the Router VM**

**command**
```
./ziti_router_auto_enroll -f -n --controllerFabricPort 8440 --controllerMgmtPort 8441 --assumePublic --disableHealthChecks --disableMetrics <jwt content>
```

```
sudo ./ziti_router_auto_enroll -f -n --controllerFabricPort 8440 --controllerMgmtPort 8441 --assumePublic --disableHealthChecks --disableMetrics eyJhbGciOiJSUzI1NiIsInR5cCI6IkpXVCJ9.eyJlbSI6ImVyb3R0IiwiZXhwIjoxNjgzNjQyMjg0LCJpc3MiOiJodHRwczovLzIwLjIxOS4xMjMuMjQ4Ojg0NDEiLCJqdGkiOiI1NzdkYmU1My04MGM3LTQ0YTMtODVlZS1iMTcxMjcyOGEwYjciLCJzdWIiOiIyTi44VkEtbFdyIn0.IESzkMxMgAyjGzjq1tbzIjBOLuzEAxSwBrTF9f5BrDbo_dDUrouYLjvSxGrPUvLgkFe9MoQrr_J5DZKkiZ684seLXJXcq8ItvfKljccrknf1JoDDEC1nfJd249MSdJ22LWBwf_Rn-hvmq6UIU_N493KdCDDlWA91evodlgyHvbDqNOm-p0RjCj3ral7wV-TSOSxHj3tex9JCIEaryLEAYAElm6SGhABFGeBxPsNd-93hcCShrgxHx1aW4mQfj-B2qRgJN3LlSUNOpCOC33MEqP3kwvUTMyaspQoYMfW-sLOZJLTEw5NLMbQ5WPq2k90mkIWnE7jMypQa-tJQNIk_wY53YSwJV-cT4T7g2gg1r1ovD0y9d4KpBXyAHr-CI6qhHYuBY31vCz38pxk-eZh-2pjIJx0vJjsG8DzvMxg0DmUeTFgXpH_E-N5nwoZ38TYLCgsM6pQNXk09ZLDflKUmZgSaaYPuf_90CVCJcQkfSY1g6k8A4cey_u9i2f2BkAy8LoCs2NmW6jZM_qIJLp3TYh3KGBkFWiG4BUnGmfSxLlgxnFcRGbpvf67RdRlBA24J3JEU3rdenVVsADunmReElylnEcBvjpW7cLfitiHH4gafFt3qeu0cLeD2McGoMxUZ4ML-tMouF9KtQnxF4X_PeqxDZ6KRvXDXg6_1tmULzPw
```
**output**
```
Failed to stop ziti-router.service: Unit ziti-router.service not loaded.
Version not specified, going to check with controller
Found version 0.27.9
Downloading file: https://github.com/openziti/ziti/releases/download/v0.27.9/ziti-linux-amd64-0.27.9.tar.gz
Downloading: 100%|███████████████████████████████████████████████████████████████████████████████████| 115M/115M [00:06<00:00, 17.1MiB/s]
Successfully downloaded file
Starting binary install
Installing service unit file
Creating config file
Starting Router Enrollment
Successfully enrolled Ziti
Service ziti-router.service start successful.
Created symlink /etc/systemd/system/multi-user.target.wants/ziti-router.service → /etc/systemd/system/ziti-router.service.
Service ziti-router.service enable successful.
```

### 2.4.4 Register the Router with link listener and tunneler
```
./ziti_router_auto_enroll -f -n --controllerFabricPort 8440 --controllerMgmtPort 8441 --assumePublic --disableHealthChecks --disableMetrics --autoTunnelListener <jwt content>
```

### 2.4.5 Register the Router with edge listener only (no link listener)
```
./ziti_router_auto_enroll -f -n --controllerFabricPort 8440 --controllerMgmtPort 8441 --disableHealthChecks --disableMetrics <jwt content>
```
### 2.4.6 Register the Router with edge listener and tunneler
```
./ziti_router_auto_enroll -f -n --controllerFabricPort 8440 --controllerMgmtPort 8441 --disableHealthChecks --disableMetrics --autoTunnelListener <jwt content>
```

## 2.5 Auto start the router
After enroll the router, a systemd service file is automatically created and enabled. To check the status of the service file, issue the following command:
```bash
systemctl status ziti-router
```
**Output**
```
om@OMERSI:~$ systemctl status ziti-router
● ziti-router.service - Ziti-Router
     Loaded: loaded (/etc/systemd/system/ziti-router.service; enabled; vendor preset: enabled)
     Active: active (running) since Tue 2023-05-09 11:39:45 UTC; 5min ago
   Main PID: 41524 (ziti)
      Tasks: 8 (limit: 4699)
     Memory: 28.1M
        CPU: 816ms
     CGroup: /system.slice/ziti-router.service
             └─41524 /opt/ziti/ziti router run /opt/ziti/config.yml
```
If the status shows **active (running)**, then the setup finished correctly.

On the controller, you can check the status of the routers. Please refer to the controller guide (useful command for the Router) section for more information.
