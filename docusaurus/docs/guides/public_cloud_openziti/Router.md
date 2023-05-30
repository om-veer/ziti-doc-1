---
sidebar_position: 30
sidebar_label: Router
title: Create new router
---

import Tabs from '@theme/Tabs';
import TabItem from '@theme/TabItem';

# 2.0 Configure new router using open ziti

In this section, we are describing how to setup the Edge Router (pub-er) for our [test network](Services#311-network-diagram-1).

## 2.1 Create the Edge Router VM 
Please follow **[Create a VM section](Controller/#11-create-a-vm-to-be-used-as-the-controller)** of the Controller Guide to setup a VM to be used as Router. 

## 2.2 Login and Update the repo and apps on VM

<Tabs
  defaultValue="DigitalOcean"
  values={[
      { label: 'Digital Ocean', value: 'DigitalOcean', },
      { label: 'Azure', value: 'Azure', },
      { label: 'AWS', value: 'AWS', },
      { label: 'Google Cloud', value: 'GCP', },
  ]}
>
<TabItem value="AWS">
Once the VM is created, we can get the IP address of the droplet from the Resources screen. 

Login to the VM by using user name "ubuntu", put the private key and IP address:
```bash
ssh -i <private_key> "ubuntu"@<ip>
```
</TabItem> 
</Tabs>

## 2.7 Route Table 
<Tabs
  defaultValue="DigitalOcean"
  values={[
      { label: 'Digital Ocean', value: 'DigitalOcean', },
      { label: 'Azure', value: 'Azure', },
      { label: 'AWS', value: 'AWS', },
      { label: 'Google Cloud', value: 'GCP', },
  ]}
>
<TabItem value="AWS">
After disable the source and distination check we have to setup the route for non ziti client using egde router with tunneler enabled as a gateway for remote site route and 100.64.0.1/32. Select the routing table in which VM subnet exist. select the edit route then add the remote routes and 100.64.0.1/32 select the target as instance VM (ER)

![Diagram](/img/AWS/route1.jpg)
![Diagram](/img/AWS/route2.jpg)
</TabItem>
</Tabs>

## 2.8 Source and Destination Check
<Tabs
  defaultValue="DigitalOcean"
  values={[
      { label: 'Digital Ocean', value: 'DigitalOcean', },
      { label: 'Azure', value: 'Azure', },
      { label: 'AWS', value: 'AWS', },
      { label: 'Google Cloud', value: 'GCP', },
  ]}
>
<TabItem value="AWS">

In Azure, the "Source and Destination Check" is named **IP forwarding**

From your VM screen, click on the **Network Interface** of that VM. On the right side menu, choose **click Action** (like the picture below). **select source and destination check** then **untick the enable**
![Diagram](/img/AWS/source-check0.jpg)
![Diagram](/img/AWS/source-check.jpg)
</TabItem>
</Tabs>

## 2.9 Firewall
<Tabs
  defaultValue="DigitalOcean"
  values={[
      { label: 'Digital Ocean', value: 'DigitalOcean', },
      { label: 'Azure', value: 'Azure', },
      { label: 'AWS', value: 'AWS', },
      { label: 'Google Cloud', value: 'GCP', },
  ]}
>
<TabItem value="AWS">

AWS default firewall is blocking all incoming access to the VM. You will need the following ports open for your ERs:

- 443/TCP (default port for edge listener)
- 80/TCP (default port for link listener)
- 53/UDP (when using as local gw)
- 22/TCP (SSH access)
- 8080/TCP (HTTP testing from non ziti client)

Following is the firewall setting for edge router which is using the GW for non ziti client.
![Diagram](/img/AWS/firewall-sg-er-nonziti.jpg)
</TabItem>
</Tabs>
