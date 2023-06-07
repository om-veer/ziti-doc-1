---
sidebar_position: 20
sidebar_label: Controller
title: Controller Config 
---

import Tabs from '@theme/Tabs';
import TabItem from '@theme/TabItem';

# 1.0 Configure the controller
## 1.1 Create a VM to be used as the Controller

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
login to aws console. go to EC2 dashboard. Click on instance. Lanch the instance

![Diagram](/img/AWS/create1.jpg)
Name the instance.
![Diagram](/img/AWS/create2.jpg)
On Quick start select the ubuntu. Select the ubuntu server 22.04 LTS. Select the instance type T2 medium
![Diagram](/img/AWS/create3.jpg)
Now select the key pair login. Choose the key name which you aready created. In the network setting presh edit. Choose the VPC name you want to attach. Select the subnet. Let the auto assign public IP
![Diagram](/img/AWS/create4.jpg)
Name the security group. Put any description name. Create the Firewall rule based on controller and ER. for controller we have to allow the TCP port 8440-8443 along with ssh port. for ER we have to allow 80, 443, 22.
SG For the controller.
![Diagram](/img/AWS/firewall-sg-cr.jpg)
SG for the ER
![Diagram](/img/AWS/firewall-sg-er.jpg)
Now click on launch the instance.
</TabItem> 
</Tabs>

## 1.2 Login and Setup Controller

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

Then follow the [Host OpenZiti Anywhere](/docs/learn/quickstarts/network/hosted/) to setup the controller. You must replace the EXTERNAL_DNS with the following command before running the quickstart.
 
**export EXTERNAL_DNS="$(curl -s eth0.me)"**

This ensures the Controller setup by the quickstart is advertising the external IP address of the VM.
</TabItem>
</Tabs>

## 1.3 Setup Ziti Administration Console (ZAC) 
**Optional**

ZAC provides GUI for managing the OpenZiti network. If you prefer UI over CLI to manage network, please following the [ZAC Setup Guide](/docs/learn/quickstarts/zac/) to setup ZAC before going to the next section.

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

To setup npm executables, you can follow [install Node.js guide](https://www.digitalocean.com/community/tutorials/how-to-install-node-js-on-ubuntu-22-04).

For example, this is how to install the version of node needed for ZAC.

Setup the repo:
```bash
cd ~
sudo apt update
curl -sL https://deb.nodesource.com/setup_18.x -o nodesource_setup.sh
sudo bash nodesource_setup.sh
```

Install nodejs:
```bash
sudo apt install nodejs
```
Now check the npm version
```bash
npm version
{
  npm: '9.5.1',
  node: '18.16.0',
  acorn: '8.8.2',
  ada: '1.0.4',
  ares: '1.19.0',
  brotli: '1.0.9',
  cldr: '42.0',
  icu: '72.1',
  llhttp: '6.0.10',
  modules: '108',
  napi: '8',
  nghttp2: '1.52.0',
  nghttp3: '0.7.0',
  ngtcp2: '0.8.1',
  openssl: '3.0.8+quic',
  simdutf: '3.2.2',
  tz: '2022g',
  undici: '5.21.0',
  unicode: '15.0',
  uv: '1.44.2',
  uvwasi: '0.0.15',
  v8: '10.2.154.26-node.26',
  zlib: '1.2.13'
}
```
After the nodejs is installed, following the rest of [ZAC Setup Guide](/docs/learn/quickstarts/zac/#cloning-from-github) to setup ZAC.

</TabItem>
</Tabs>

## 1.4 Firewall
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
Azure's default firewall is blocking all incoming access to the VM. You will need to open ports you specified for controller and ZAC (if you plan to use ZAC). Here is a example of the firewall ports if you used the default ports.

![Diagram](/img/AWS/firewall-sg-cr.jpg)

</TabItem>
</Tabs>
