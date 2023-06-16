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

