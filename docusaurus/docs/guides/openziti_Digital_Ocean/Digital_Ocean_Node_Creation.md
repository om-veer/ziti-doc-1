---
sidebar_position: 15
sidebar_label: Digital_Ocean_Node_Creation
title: Digital_Ocean_Node_Creation
---

Create droplet using digital ocean cloud

1-	Choose the region
Ex : choose Bangalore

2-	VPC Network selection:
Select the default VPC or defined VPC network

3-	Choose the image:
Select the OS marketplace Ubuntu 22.10x64(latest image)

4-	Choose size:
Select droplet size basic(default) or General purpose based on network size. Select the CPU option regular(default) for controller select 2 GB ram and 2 CPU. Disk type SSD. Select SSD size default based on RAM and CPU.

5-	Leave default Volume

6-	Choose authentication Method:
Select the SSH key based authentication. Create the New SSH key if you don’t have ssh key.

7-	Finalize the details:
Leave quantity 1 droplet. You can put any hostname as per choice. Put any tag(optional). Select project netfoundry Partnership.

8-	Now create the droplet.

9-	SSH the VM using your private key.  Example “ssh -i private.key root@public_ip_VM

Networking:

- Digital Ocean VM has 2 Interface. One is eth0 acting as WAN and other interface eth1 acting as LAN. Default routes/default Gateway has points towards the WAN interface eth0.
- Ip routes requred to reach the remote Ip/Subnet the same VPC instance. Use the LAN IP eth0 next VM as Gateway.

Example :

- Add the route

ip route add 100.64.0.0/10 via 10.122.0.4 dev eth1 proto static

- Delete the route: 

ip route delete 100.64.0.0/10 via 10.122.0.4