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
  ]}
>
<TabItem value="AWS">
Refer to router AWS sec 2.7 and 2.8
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
  ]}
>
<TabItem value="AWS">
Modify **/etc/systemd/resolved.conf**. Put local IP of the "local-er" into the file. For example:
```
DNS=10.5.0.4  #local private IP of the ER eth0
```
**NOTE, the IP address should match your Next hop in the route table**

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
  ]}
>
<TabItem value="AWS">
Refer to router AWS sec 2.7 and 2.8
</TabItem>
</Tabs>
