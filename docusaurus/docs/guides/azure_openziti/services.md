---
sidebar_position: 50
sidebar_label: Services
title: Services
---

# 3.0 Ziti services
## 3.1 Introduction
In this guide, we will demonstrate ziti services with three examples:
- connection between two identities running ziti-edge-tunnel
- connection between ziti-edge-tunnel and a ziti-router (with tunnel enabled)
- connection from a non-OpenZiti endpoint using router as GW

![Diagram](/img/azure/service1.jpg)

This guide provides both CLI and GUI (ZAC) instructions. To use ZAC, make sure ZAC is installed. If you have not installed ZAC, and would like to use it in this section, please follow the [ZAC Setup Guide](/docs/learn/quickstarts/zac/) before continue.

## 3.2 Setup VMs for ziti network
### 3.2.1 Controller
#### 3.2.1.1 ZAC
Please make sure you can login to your network and see the welcome screen before continue.

![Diagram](/img/azure/zac-console.jpg)

#### 3.2.1.2 CLI
You will need to login to controller to provision identities and service. Please make sure you are performing the action on the right node.

On the controller, before performing the CLI command, you will need to login first:
```bash
source ~/.ziti/quickstart/$(hostname -s)/$(hostname -s).env

adding /home/om/.ziti/quickstart/omcontroller/ziti-bin/ziti-v0.27.9 to the path
$ zitiLogin
Token: a41dc021-ee1f-4da9-9922-98dfb74a9e00
Saving identity 'default' to /home/om/.ziti/quickstart/omcontroller/ziti-cli.json
```

The login token expires after a period of time. If the token expired, you will need to login again via the same command. You will see error message like this when the CLI logs out:
```
error: error listing https://20.219.123.248:8441/edge/management/v1/config-types?filter=id%3D%22host.v1%22 in Ziti Edge Controller. Status code: 401 Unauthorized, Server returned: {
    "error": {
        "code": "UNAUTHORIZED",
        "message": "The request could not be completed. The session is not authorized or the credentials are invalid",
        "requestId": "7zxWCy7Vx"
    },
    "meta": {
        "apiEnrollmentVersion": "0.0.1",
        "apiVersion": "0.0.1"
    }
}
```

### 3.2.2 Router
Follow the [Router guide](/docs/guides/azure_openziti/Router/) to setup a "Router with link listener and tunneler". We will name this router: **OM-ER-SI**

#### 3.2.2.1 ZAC
On the ZAC **ROUTERS** screen, you can check your router and make sure it is created correctly. You can also check the identity associated with the router by clicked on **IDENTITIES**

![Diagram](/img/azure/service1-router.jpg)

On the **IDENTITIES** screen, you will see an identity match your router name.

![Diagram](/img/azure/service2-router.jpg)

#### 3.2.2.2 CLI
Once the router is create, you can check the **controller** to make sure the router is created correctly.

You should see an router named **OM-ER-SI**
```
# ziti edge list edge-routers
╭────────────┬──────────────────────────┬────────┬───────────────┬──────┬────────────╮
│ ID         │ NAME                     │ ONLINE │ ALLOW TRANSIT │ COST │ ATTRIBUTES │
├────────────┼──────────────────────────┼────────┼───────────────┼──────┼────────────┤
│ 2N.8VA-lWr │ OM-ER-SI                 │ true   │ true          │    0 │            │
│ 3-5h7-VYqX │ omcontroller-edge-router │ true   │ true          │    0 │ public     │
│ OlmZIzLltr │ OM-ER-ME                 │ true   │ true          │    0 │            │
╰────────────┴──────────────────────────┴────────┴───────────────┴──────┴────────────╯

```
You should also see an identity named **OM-ER-SI**
```
# ziti edge list identities
╭────────────┬──────────────────────────┬────────┬────────────┬─────────────╮
│ ID         │ NAME                     │ TYPE   │ ATTRIBUTES │ AUTH-POLICY │
├────────────┼──────────────────────────┼────────┼────────────┼─────────────┤
│ 2N.8VA-lWr │ OM-ER-SI                 │ Router │            │ default     │
│ 2ZGtmGwpV  │ Default Admin            │ User   │            │ default     │
│ 3-5h7-VYqX │ omcontroller-edge-router │ Router │            │ default     │
│ fs24OJilWr │ OM-CL-ME                 │ Device │            │ default     │
│ qXLkSkiltr │ OM-CL-SI                 │ Device │            │ default     │
╰────────────┴──────────────────────────┴────────┴────────────┴─────────────╯

```
Since we created this router as link listener, you should also check the link listener function is correctly setup. Check the router on fabric to make sure it has listeners displayed for the router you created.
```
# ziti fabric list routers
╭────────────┬──────────────────────────┬────────┬──────┬──────────────┬──────────┬────────────────────────┬─────────────────────────────╮
│ ID         │ NAME                     │ ONLINE │ COST │ NO TRAVERSAL │ DISABLED │ VERSION                │ LISTENERS                   │
├────────────┼──────────────────────────┼────────┼──────┼──────────────┼──────────┼────────────────────────┼─────────────────────────────┤
│ 2N.8VA-lWr │ OM-ER-SI                 │ true   │    0 │ false        │ false    │ v0.27.9 on linux/amd64 │ 1: tls:20.219.121.237:80    │
│ 3-5h7-VYqX │ omcontroller-edge-router │ true   │    0 │ false        │ false    │ v0.27.9 on linux/amd64 │ 1: tls:20.219.123.248:10080 │
│ OlmZIzLltr │ OM-ER-ME                 │ true   │    0 │ false        │ false    │ v0.27.9 on linux/amd64 │                             │
╰────────────┴──────────────────────────┴────────┴──────┴──────────────┴──────────┴────────────────────────┴─────────────────────────────╯
results: 1-3 of 3

```

### 3.2.3 Tunnelers/Identities
We need two tunnelers for our testing. Please follow [controller guide](/docs/guides/azure_openziti/Controller/) section of "Create VM on Azure cloud" to create two VMs running Ubuntu 22.04.

#### 3.2.3.1 Create Identity with ZAC
To create Identities on ZAC, go to the Identities screen, and press **+** icon. On the **CREATE IDENTITY** widget, enter the **NAME** of the identity (Required and has to be unique). Leave the **IDENTITY TYPE** as "Device". Leave the **ENROLLMENT TYPE** as "One Time Token". Fill in other optional fields and press **SAVE**.

![Diagram](/img/azure/service1-client.jpg)

Once the identity is created, you can check your identity and download the **JWT** from the **MANAGE IDENTITIES** screen

![Diagram](/img/azure/service2-client.jpg)

**Create two identities before you continue.**

#### 3.2.3.2 Create Identity with CLI
Create two identities (OM-CL-SI, OM-CL-ME) on the **controller**:
```bash
ziti edge create identity user OM-CL-SI -o OM-CL-SI.jwt
ziti edge create identity user OM-CL-ME -o OM-CL-ME.jwt
```

#### 3.2.3.3 Register Identities
Login to your VMs created earlier, follow [Install Linux Package](/docs/reference/tunnelers/linux/) **Ubuntu Jammy 22.04** section to install and register your ziti-edge-tunnel.

After the tunnel is registered, check the status and make sure it is running
```
# systemctl status ziti-edge-tunnel
● ● ziti-edge-tunnel.service - Ziti Edge Tunnel
     Loaded: loaded (/etc/systemd/system/ziti-edge-tunnel.service; enabled; vendor preset: enabled)
     Active: active (running) since Wed 2023-05-10 09:35:26 UTC; 3s ago
    Process: 1211 ExecStartPre=/opt/openziti/bin/ziti-edge-tunnel.sh (code=exited, status=0/SUCCESS)
   Main PID: 1222 (ziti-edge-tunne)
      Tasks: 5 (limit: 4699)
     Memory: 34.4M
        CPU: 91ms
     CGroup: /system.slice/ziti-edge-tunnel.service
             └─1222 /opt/openziti/bin/ziti-edge-tunnel run --verbose=2 --dns-ip-range=100.64.0.1/10 --identity-dir=/opt/openziti/etc/id
```

#### 3.2.3.4 Gather info of the VMs
After you registered both tunnelers, write down the local IP and subnet of these VMs, we will be using the local IP to test our traffics.
```
OM-ER-SI: 10.7.0.5/24
OM-CL-SI: 10.7.0.6/24
OM-CL-ME: 10.5.0.6/24
```

## 3.3 Create Edge Router Policy
By default, the system created an edge router policy to bind all identities to any routers tag with **#public** attribute. So, we need to add #public to our router (OM-ER-SI). 

### 3.3.1 ZAC

From the **ROUTERS** screen, you can choose **Router Policies** to display the edge router policies. As you can see, the default policy is already in place.

![Diagram](/img/azure/service1-routerpolicy.jpg)

To add attribute "public" to the router, choose your router from the **ROUTERS** screen, click on **...** and choose **Edit**

![Diagram](/img/azure/service2-routerpolicy.jpg)

On the **EDIT EDGE ROUTER** screen, you can add **public** attribute and then press **SAVE**

![Diagram](/img/azure/service3-routerpolicy.jpg)

### 3.3.2 CLI
On the **controller**:

check the edge router policy
```
# ziti edge list edge-router-policies
╭────────────────────────┬───────────────────────────────┬───────────────────────────┬───────────────────────────╮
│ ID                     │ NAME                          │ EDGE ROUTER ROLES         │ IDENTITY ROLES            │
├────────────────────────┼───────────────────────────────┼───────────────────────────┼───────────────────────────┤
│ 111Cl8H2Z0x8zshP2Qwiyw │ allEdgeRouters                │ #public                   │ #all                      │
│ 2N.8VA-lWr             │ edge-router-2N.8VA-lWr-system │ @OM-ER-SI                 │ @OM-ER-SI                 │
│ 3-5h7-VYqX             │ edge-router-3-5h7-VYqX-system │ @omcontroller-edge-router │ @omcontroller-edge-router │
╰────────────────────────┴───────────────────────────────┴───────────────────────────┴───────────────────────────╯
results: 1-3 of 3

```

check the edge router attribute
```
# ziti edge list edge-routers
╭────────────┬──────────────────────────┬────────┬───────────────┬──────┬────────────╮
│ ID         │ NAME                     │ ONLINE │ ALLOW TRANSIT │ COST │ ATTRIBUTES │
├────────────┼──────────────────────────┼────────┼───────────────┼──────┼────────────┤
│ 2N.8VA-lWr │ OM-ER-SI                 │ true   │ true          │    0 │            │
│ 3-5h7-VYqX │ omcontroller-edge-router │ true   │ true          │    0 │ public     │
│ OlmZIzLltr │ OM-ER-ME                 │ true   │ true          │    0 │            │
╰────────────┴──────────────────────────┴────────┴───────────────┴──────┴────────────╯
results: 1-3 of 3

```

Add "public" attribute to our router:
```
# ziti edge update edge-router JAMES-ER-SF -a "public"
# ziti edge list edge-routers
╭────────────┬──────────────────────────┬────────┬───────────────┬──────┬────────────╮
│ ID         │ NAME                     │ ONLINE │ ALLOW TRANSIT │ COST │ ATTRIBUTES │
├────────────┼──────────────────────────┼────────┼───────────────┼──────┼────────────┤
│ 2N.8VA-lWr │ OM-ER-SI                 │ true   │ true          │    0 │ public     │
│ 3-5h7-VYqX │ omcontroller-edge-router │ true   │ true          │    0 │ public     │
│ OlmZIzLltr │ OM-ER-ME                 │ true   │ true          │    0 │            │
╰────────────┴──────────────────────────┴────────┴───────────────┴──────┴────────────╯
results: 1-3 of 3
```

## 3.4 Setup Tunnel to Tunnel ssh connection

In this section, we will be showing how to configure ziti to establish ssh service via our network. We will be using a pseudo dns name (t2tssh.ziti) to establish this connection.  The intercept side (ingress) will be OM-CL-SI, the server side (egress) will be OM-CL-ME.

For CLI procedures, all commands are done on the **controller**

### 3.4.1 Create a host.v1 config

This config is used instruct the server-side tunneler how to offload the traffic from the overlay, back to the underlay. We are dropping the traffic off our loopback interface, so we use the address "127.0.0.1“

#### 3.4.1.1 ZAC
Create the configuration from **MANAGE CONFIGURATIONS** screen.

![Diagram](/img/digital_ocean/Services10.png)

#### 3.4.1.2 CLI
```bash
ziti edge create config t2thostconf host.v1 '{"protocol":"tcp", "address":"127.0.0.1", "port":22}'
```
### 3.4.2 Create an intercept.v1 config 
This config is used to instruct the intercept-side tunneler how to correctly intercept the targeted traffic and put it onto the overlay. We are setting up intercept on dns name "t2tssh.ziti"

#### 3.4.2.1 ZAC
![Diagram](/img/digital_ocean/Services11.png)

Check the provisioned configs on the configuration screen:

![Diagram](/img/digital_ocean/Services12.png)

#### 3.4.2.2 CLI
```bash
ziti edge create config t2tintconf intercept.v1 '{"protocols": ["tcp"], "addresses": ["t2tssh.ziti"], "portRanges": [{"low": 22, "high": 22}]}'
```
If the command finished successfully, you will see two configs:
```
# ziti edge list configs
╭────────────────────────┬─────────────┬──────────────╮
│ ID                     │ NAME        │ CONFIG TYPE  │
├────────────────────────┼─────────────┼──────────────┤
│ 1R9iDpU7OvREH6LNxQapPO │ t2tintconf  │ intercept.v1 │
│ gIeZ1KgilGXzkY4RxDF2f  │ t2thostconf │ host.v1      │
╰────────────────────────┴─────────────┴──────────────╯
results: 1-2 of 2
```
### 3.4.3 Create Service
Now we need to put these two configs into a service. We going to name the service "t2tssh":

#### 3.4.3.1 ZAC
Create service from **Services** menu from **MANAGE EDGE SERVICES** screen.

![Diagram](/img/digital_ocean/Services13.png)

#### 3.4.3.2 CLI
```bash
ziti edge create service t2tssh -c t2tintconf,t2thostconf
```

**Check Service**
```
# ziti edge list services
╭───────────────────────┬────────┬────────────┬─────────────────────┬────────────╮
│ ID                    │ NAME   │ ENCRYPTION │ TERMINATOR STRATEGY │ ATTRIBUTES │
│                       │        │  REQUIRED  │                     │            │
├───────────────────────┼────────┼────────────┼─────────────────────┼────────────┤
│ CZJi5tDxTk8C8XymCbu7f │ t2tssh │ true       │ smartrouting        │            │
╰───────────────────────┴────────┴────────────┴─────────────────────┴────────────╯
results: 1-1 of 1
```
Please note, the **Encryption = true** and **smartrouting** are default setting.

### 3.4.4 Create Bind Service policy
Now we need to specify which tunneler (in our case, **OM-CL-ME**)is going to host the service by setting up a bind service policy

#### 3.4.4.1 ZAC

Create a **service policy** from **MANAGE SERVICE POLICIES** screen:

![Diagram](/img/azure/service1-bind.jpg)

#### 3.4.4.2 CLI
```bash
ziti edge create service-policy t2tssh.bind Bind --service-roles '@t2tssh' --identity-roles "@OM-CL-ME"
```
### 3.4.5 Create Dial Service policy
Now we need to specify the intercept side tunneler (**OM-CL-SI**) for the service by setting up a dial service policy

#### 3.4.5.1 ZAC

![Diagram](/img/azure/service1-dial.jpg)

We should have two service policies created. The services name should match on two policies.  One policy is Bind and the other ons is Dial.  They should be set to different identities.

![Diagram](/img/azure/service1-manpolicy.jpg)
#### 3.4.5.2 CLI
```bash
ziti edge create service-policy t2tssh.dial Dial --service-roles '@t2tssh' --identity-roles "@OM-CL-SI"
```
Make sure both policies are setup correctly：
```
# ziti edge list service-policies
╭────────────────────────┬─────────────┬──────────┬───────────────┬────────────────┬─────────────────────╮
│ ID                     │ NAME        │ SEMANTIC │ SERVICE ROLES │ IDENTITY ROLES │ POSTURE CHECK ROLES │
├────────────────────────┼─────────────┼──────────┼───────────────┼────────────────┼─────────────────────┤
│ 5YDNBt4rkOyMmMNIyT6oKX │ t2tssh.bind │ AllOf    │ @t2tssh       │ @OM-CL-ME      │                     │
│ 6bkUXw2J6NV6I3VfCgs8uw │ t2tssh.dial │ AllOf    │ @t2tssh       │ @OM-CL-SI      │                     │
╰────────────────────────┴─────────────┴──────────┴───────────────┴────────────────┴─────────────────────╯
results: 1-2 of 2
```
You should also make sure the policy advisor display correctly:
```
# ziti edge policy-advisor services |grep t2tssh
OKAY : OM-CL-ME (2) -> t2tssh (3) Common Routers: (2/2) Dial: N Bind: Y
OKAY : OM-CL-SI (2) -> t2tssh (3) Common Routers: (2/2) Dial: Y Bind: N
```
Make sure there is a "Dial" line and a "Bind" line. They are both "OKAY" and has at least 1 Common Routers.

### 3.4.6 Verify the connection
Login to the intercept side tunneler (**OM-CL-SI**) node, you should be able to ssh to the server (**OM-CL-ME**) by using dns name "t2tssh.ziti"
```
om@OMCLSI:~$ ssh t2tssh.ziti
om@t2tssh.ziti's password:
Welcome to Ubuntu 22.04.2 LTS (GNU/Linux 5.15.0-1037-azure x86_64)

 * Documentation:  https://help.ubuntu.com
 * Management:     https://landscape.canonical.com
 * Support:        https://ubuntu.com/advantage

  System information as of Thu May 11 08:49:28 UTC 2023

  System load:  0.19287109375     Processes:             110
  Usage of /:   6.1% of 28.89GB   Users logged in:       0
  Memory usage: 8%                IPv4 address for eth0: 10.5.0.6
  Swap usage:   0%                IPv4 address for tun0: 100.64.0.1

 * Strictly confined Kubernetes makes edge and IoT secure. Learn how MicroK8s
   just raised the bar for easy, resilient and secure K8s cluster deployment.

   https://ubuntu.com/engage/secure-kubernetes-at-the-edge

Expanded Security Maintenance for Applications is not enabled.

1 update can be applied immediately.
To see these additional updates run: apt list --upgradable

Enable ESM Apps to receive additional future security updates.
See https://ubuntu.com/esm or run: sudo pro status


Last login: Wed May 10 09:26:45 2023 from 47.9.90.183
om@OMCLME:~$ exit
logout
Connection to t2tssh.ziti closed.
om@OMCLSI:~$
```

## 3.5 Setup Tunnel to ERT http connection
Edge Router Tunnel (ERT) has combine router and tunneler function. In this section, we will demonstrate a connection between a Ziti-Edge-Tunnel (OM-CL-ME) to a ERT (OM-ER-SI)

The procedure is similar to tunnel to tunnel service, so please refer to that section for detail explanation of each step.

### 3.5.1 Create a host.v1 config
#### 3.5.1.1 ZAC
![Diagram](/img/digital_ocean/Services20.png)
#### 3.5.1.2 CLI
```bash
ziti edge create config t2ehostconf host.v1 '{"protocol":"tcp", "address":"127.0.0.1", "port":8080}'
```
### 3.5.2 Create an intercept.v1 config 
Use destination local ip (10.7.0.5) as intercept address
#### 3.5.2.1 ZAC
![Diagram](/img/azure/service1-conf.jpg)

And you should have two configs:

![Diagram](/img/azure/service2-conf.jpg)
#### 3.5.2.2 CLI
```bash
ziti edge create config t2eintconf intercept.v1 '{"protocols": ["tcp"], "addresses": ["10.5.0.4"], "portRanges": [{"low": 80, "high": 80}]}'
```
```
# ziti edge list configs |grep t2e
│ 3puZjTKLnpyUuHmXPZO4Me │ t2eintconf  │ intercept.v1 │
│ Ah3VkZy5tXfwhKfeLELFd  │ t2ehostconf │ host.v1      │
```
### 3.5.3 Create Service
Now we need to put these two configs into a service. We going to name the service "t2ehttp":
#### 3.5.3.1 ZAC
![Diagram](/img/azure/service2.jpg)
#### 3.5.3.2 CLI
```
ziti edge create service t2ehttp -c t2eintconf,t2ehostconf
```
**Check Service**
```
# ziti edge list services |grep t2e
│ 5FI8Aw0IQ7xaUMgH2rKvps │ t2ehttp │ true       │ smartrouting        │            │
```
### 3.5.4 Create Bind Service policy
#### 3.5.4.1 ZAC
![Diagram](/img/azure/service2-bind.jpg)
#### 3.5.4.2 CLI
```bash
ziti edge create service-policy t2ehttp.bind Bind --service-roles '@t2ehttp' --identity-roles "@OM-ER-ME1"
```
### 3.5.5 Create Dial Service policy
Now we need to specify the intercept side tunneler (**OM-CL-SI**) for the service by setting up a dial service policy
#### 3.5.5.1 ZAC
![Diagram](/img/azure/service2-dial.jpg)

Check two service polices:

![Diagram](/img/azure/service2-manpolicy.jpg)
#### 3.5.5.2 CLI
```bash
ziti edge create service-policy t2ehttp.dial Dial --service-roles '@t2ehttp' --identity-roles "@OM-CL-SI"
```
Make sure both policies are setup correctly：
```
# ziti edge list service-policies |grep t2e
│ 6xfw04TBBaRnyAprlUlU4w │ t2ehttp.dial │ AllOf    │ @t2ehttp      │ @OM-CL-SI      │                     │
│ V8qQvn1nAsqjcZLt6Pkvu  │ t2ehttp.bind │ AllOf    │ @t2ehttp      │ @OM-ER-ME1     │                     │

```
You should also make sure the policy advisor display correctly:
```
# ziti edge policy-advisor services |grep t2ehttp
OKAY : OM-ER-SI (2) -> t2ehttp (3) Common Routers: (2/2) Dial: N Bind: Y
OKAY : OM-CL-ME (2) -> t2ehttp (3) Common Routers: (2/2) Dial: Y Bind: N
```
Make sure there is a "Dial" line and a "Bind" line. They are both "OKAY" and has at least 1 Common Routers.
### 3.5.6 Verify the connection
Login to the router (**OM-ER-ME1**), setup a file to send via http.
```
om@ommeer:~$ cat hello.txt
this is er from middle east
om@ommeer:~$ python3 -m http.server 8080
Serving HTTP on 0.0.0.0 port 8080 (http://0.0.0.0:8080/) ...
127.0.0.1 - - [12/May/2023 06:33:40] "GET /hello.txt HTTP/1.1" 200 -
```

Login to the intercept side tunneler (**OM-CL-SI**) node.
```
om@OMCLSI:~$ curl http://10.5.0.4/hello.txt
this is er from middle east
om@OMCLSI:~$
```

### 3.5.7 Conclusion
In section, we demonstrated intercepting http (port 80) request to an IP address and forward the request to a remote http server listening to port 8080 via ziti network.


## 3.6 Setup Connection from a non-OpenZiti client

### 3.6.1 Introduction
In this section, we will demonstrate how to setup non-OpenZiti client to use the ziti service. The following conditions need to be met in order for this connection to work correctly:
- The non-OpenZiti client has to be in the same data center as the edge router. 
- The edge router is created with tunneler enabled.

We are also need to do the following to successfully demonstrate the connection:
- Create host and intercept config
- Create service
- Create dial and bind policy
- Setup route on the non-OpenZiti client

We will create an intercept at 11.11.11.11:80, and drop off at local IP (10.7.0.6) **OM-CL-NY" port (80).
![Diagram](/img/azure/service3.jpg)

### 3.6.2 Create a host.v1 config
Used address **10.7.0.6** as host side destination, and port **80** as destination port.
#### 3.6.2.1 ZAC
![Diagram](/img/azure/service3-host.jpg)
#### 3.6.2.2 CLI
```bash
ziti edge create config e2thostconf host.v1 '{"protocol":"tcp", "address":"10.7.0.6", "port":80}'
```
### 3.6.3 Create an intercept.v1 config 
Use a pseudo ip (11.11.11.11) as intercept address
#### 3.6.3.1 ZAC
![Diagram](/img/digital_ocean/Services31.png)

And you should have two configs:

![Diagram](/img/digital_ocean/Services32.png)
#### 3.6.3.2 CLI
```bash
ziti edge create config e2tintconf intercept.v1 '{"protocols": ["tcp"], "addresses": ["11.11.11.11"], "portRanges": [{"low": 80, "high": 80}]}'
```
```
# ziti edge list configs |grep e2t
│ 2KWEVBCfrxpR2ZixAqjb6V │ e2thostconf │ host.v1      │
│ 3WqQOgoJScWEYb6E5hSBpP │ e2tintconf  │ intercept.v1 │

```
### 3.6.4 Create Service
Now we need to put these two configs into a service. We going to name the service "t2ehttp":
#### 3.6.4.1 ZAC
![Diagram](/img/azure/service33.jpg)
#### 3.6.4.2 CLI
```
ziti edge create service e2thttp -c e2thostconf,e2tintconf
```
**Check Service**
```
#  ziti edge list services |grep e2t
│ 4jCupltrqcsQZeFqi06h0D │ e2thttp │ true       │ smartrouting        │            │

```
### 3.6.5 Create Bind Service policy
#### 3.6.5.1 ZAC
![Diagram](/img/azure/service3-bind.jpg)
#### 3.6.5.2 CLI
```bash
ziti edge create service-policy e2thttp.bind Bind --service-roles '@e2thttp' --identity-roles "@OM-CL-SI"
```
### 3.6.6 Create Dial Service policy
Now we need to specify the intercept side tunneler (**OM-ER-ME1**) for the service by setting up a dial service policy
#### 3.6.6.1 ZAC
![Diagram](/img/azure/service3-dial.jpg)

Check two service polices:

![Diagram](/img/azure/service3-manpolicy.jpg)
#### 3.6.6.2 CLI
```bash
 ziti edge create service-policy e2thttp.dial Dial --service-roles '@e2thttp' --identity-roles "@OM-ER-ME1"
```
Make sure both policies are setup correctly：
```
# ziti edge list service-policies |grep e2t
│ 3UDiZHDwG4hrtSd8gys9iX │ e2thttp.bind │ AllOf    │ @e2thttp      │ @OM-CL-SI      │                     │
│ 5lrku7yUYYpLq76qHGifno │ e2thttp.dial │ AllOf    │ @e2thttp      │ @OM-ER-ME1     │                     │

```
You should also make sure the policy advisor display correctly:
```
# ziti edge policy-advisor services |grep e2thttp
ziti edge policy-advisor services |grep e2thtt
OKAY : OM-ER-ME1 (3) -> e2thttp (3) Common Routers: (3/3) Dial: Y Bind: N
OKAY : OM-CL-SI (2) -> e2thttp (3) Common Routers: (2/2) Dial: N Bind: Y
```
Make sure there is a "Dial" line and a "Bind" line. They are both "OKAY" and has at least 1 Common Routers.
### 3.6.7 Verify the connection
Login to the Client (**OM-CL-SI**), setup a file to send via http.
```
om@OMCLSI:~$ cat hello.txt
this is a file from client pc in south india
om@OMCLSI:~$ sudo python3 -m http.server 80
Serving HTTP on 0.0.0.0 port 80 (http://0.0.0.0:80/) ...

```


Login to the non-OpenZiti client machine (**OM-CL-ME**).

**setup the route first**. The route is via our ER in the same DC (10.5.0.4). Add route table in Azure route table. Also enable the IP forwording on primary ip address interface.
#### Azure route table
![Diagram](/img/azure/ip-forwording.jpg)
![Diagram](/img/azure/azure-route.jpg)
We also need to allow the DNS port 53 UDP on azure cloud firewall in inbound.
![Diagram](/img/azure/firewall-dns.jpg)
```
 Add the local Ip to the DNS for ER running in tunneler mode.Edit the resolved.conf file 

vi /etc/systemd/resolved.conf

DNS=10.47.0.6  #local private Ip of the ER eth0

systemctl restart systemd-resolved.service

resolvectl

Now test the connection
```
om@OMCLME:~$ curl http://11.11.11.11/hello.txt

this is a file from client pc in south india

om@OMCLME:~$

```