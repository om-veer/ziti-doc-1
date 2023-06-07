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
      { label: 'IBM', value: 'IBM', },
  ]}
>
<TabItem value="IBM">
Once the VM is created, we can get the IP address of the VM from the Devices screen.  

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
      { label: 'IBM', value: 'IBM', },
  ]}
>
<TabItem value="IBM">
IBM cloud do not have routeing table. we can setup the route directly on the VM
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
      { label: 'IBM', value: 'IBM', },
  ]}
>
<TabItem value="IBM">
Not applicable
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
      { label: 'IBM', value: 'IBM', },
  ]}
>
<TabItem value="IBM">

IBM cloud by default does not setup firewall for the VM.
</TabItem>
</Tabs>
