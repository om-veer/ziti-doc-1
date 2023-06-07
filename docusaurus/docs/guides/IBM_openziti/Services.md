---
sidebar_position: 50
sidebar_label: Services
title: Services
---

import Tabs from '@theme/Tabs';
import TabItem from '@theme/TabItem';

# 3.0 Ziti services
## 3.1 Introduction

#### 3.6.7.1 Test IP intercept
<Tabs
  defaultValue="DigitalOcean"
  values={[
      { label: 'Digital Ocean', value: 'DigitalOcean', },
      { label: 'Azure', value: 'Azure', },
      { label: 'AWS', value: 'AWS', },
      { label: 'Google Cloud', value: 'GCP', },
      { label: 'IBM', value: 'IBM', },
  ]}
>
<TabItem value="IBM">
**setup the route first**. The route is via our ER in the same DC (10.162.209.220)

```
root@Non-OpenZiti-Client:~# ip route add 11.11.11.11/32 via 10.162.209.220
```
</TabItem>
</Tabs>

#### 3.6.7.2 Modify the resolver

<Tabs
  defaultValue="DigitalOcean"
  values={[
      { label: 'Digital Ocean', value: 'DigitalOcean', },
      { label: 'Azure', value: 'Azure', },
      { label: 'AWS', value: 'AWS', },
      { label: 'Google Cloud', value: 'GCP', },
      { label: 'IBM', value: 'IBM', },
  ]}
>
<TabItem value="IBM">
Modify **/etc/systemd/resolved.conf**. Put External(public) IP of the "local-er" into the file. For example:

```
DNS=169.38.108.24    #External Public IP of the ER eth1
```

Restart the systemd-resolved service 
```bash
systemctl restart systemd-resolved.service
```

</TabItem>
</Tabs>

#### 3.6.7.3 Test DNS intercept
<Tabs
  defaultValue="DigitalOcean"
  values={[
      { label: 'Digital Ocean', value: 'DigitalOcean', },
      { label: 'Azure', value: 'Azure', },
      { label: 'AWS', value: 'AWS', },
      { label: 'Google Cloud', value: 'GCP', },
      { label: 'IBM', value: 'IBM', },
  ]}
>
<TabItem value="IBM">
The route is via our ER in the same DC (10.162.209.220). The DNS is going to resolve to 100.64.0.0/10 address.

```
sudo ip route add 100.64.0.0/10 via 10.162.209.220 dev eth0
```
</TabItem>
</Tabs>
