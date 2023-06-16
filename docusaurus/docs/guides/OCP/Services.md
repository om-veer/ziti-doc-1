---
sidebar_position: 50
sidebar_label: Services
title: Services
---

import Tabs from '@theme/Tabs';
import TabItem from '@theme/TabItem';

# 3.0 Ziti services
## 3.1 Introduction
#### 3.2.3.3 Register Identities
- Note: As of now ip based controller DNS only working for tunneler ver 0.21.0. to install 0.21.0 tunneler use this command.

```
(
set -euo pipefail

curl -sSLf https://get.openziti.io/tun/package-repos.gpg \
  | sudo gpg --dearmor --output /usr/share/keyrings/openziti.gpg

echo 'deb [signed-by=/usr/share/keyrings/openziti.gpg] https://packages.openziti.org/zitipax-openziti-deb-stable jammy main' \
  | sudo tee /etc/apt/sources.list.d/openziti.list >/dev/null

sudo apt update
sudo apt install ziti-edge-tunnel=0.21.0
)
```

Login to your VMs created earlier, follow [Install Linux Package](/docs/reference/tunnelers/linux/) **Ubuntu Jammy 22.04** section to install and register your ziti-edge-tunnel.

After the tunnel is registered, check the status and make sure it is running

```
# systemctl status ziti-edge-tunnel
● ziti-edge-tunnel.service - Ziti Edge Tunnel
     Loaded: loaded (/etc/systemd/system/ziti-edge-tunnel.service; enabled; vendor preset: enabled)
     Active: active (running) since Wed 2023-04-05 20:57:04 UTC; 1min 41s ago
    Process: 2640 ExecStartPre=/opt/openziti/bin/ziti-edge-tunnel.sh (code=exited, status=0/SUCCESS)
   Main PID: 2641 (ziti-edge-tunne)
      Tasks: 5 (limit: 2323)
     Memory: 6.5M
        CPU: 752ms
     CGroup: /system.slice/ziti-edge-tunnel.service
             └─2641 /opt/openziti/bin/ziti-edge-tunnel run --verbose=2 --dns-ip-range=100.64.0.1/10 --identity-dir=/opt/openziti/etc/identities
```
#### 3.6.7.1 Test IP intercept
<Tabs
  defaultValue="OCP"
  values={[
      { label: 'OCP', value: 'OCP', },
  ]}
>
<TabItem value="OCP">
TO test the connectivity from non ziti client we have to do 2 things

1- add the route in the OCP routering table

go to router sec 2.7

2- disable the source/destination check from the local-ER

Go to router sec 2.8

</TabItem>
</Tabs>

#### 3.6.7.2 Modify the resolver

<Tabs
  defaultValue="OCP"
  values={[
      { label: 'OCP', value: 'OCP', },
  ]}
>
<TabItem value="OCP">
Modify **/etc/systemd/resolved.conf**. Put local IP of the "local-er" into the file. For example:
```
DNS=10.0.0.115  #local private IP of the ER eth0
```
**NOTE, the IP address should match your Next hop in the route table**

Restart the systemd-resolved service 
```bash
systemctl restart systemd-resolved.service
```

</TabItem>
</Tabs>

