The Ziti Administration Console (ZAC) is a web UI provided by the OpenZiti project which will allow you to configure and explore a Ziti Network.

# Prerequisites
It's expected that you're using bash for these commands. If you're using Windows we strongly recommend that you install and use Windows Subsystem for Linux (WSL). Other operating systems it's recommended you use bash unless you are able to translate to your shell accordingly.

You will need node and npm executables from Node.js v16+.

NOTE
When running Ziti Administration Console, you should also prefer using https over http. In order to do this you will need to either create, or copy the certificates needed. Each section below tries to show you how to accomplish this on your own.

# Cloning From GitHub
These steps are applicable to both the local, no docker as well as the hosted yourself deployments. Do note, these steps expect you have the necessary environment variables established in your shell. If you used the default parameters, you can establish these variables using the file at ${HOME}/.ziti/quickstart/$(hostname)/$(hostname).env. To deploy ZAC after following one of those guides, you can perform the following steps.

Clone the ziti-console repo from github:
```
git clone https://github.com/openziti/ziti-console.git "${ZITI_HOME}/ziti-console"
```
Install Node modules:
```
cd "${ZITI_HOME}/ziti-console"
npm install
```
Use the ziti-controller certificates for the Ziti Console:

Link a server certificate into the ziti-console directory. Your web browser won't recognize it, but it's sufficient for this exercise to have server TLS for your ZAC session.
```
ln -s "${ZITI_PKI}/${ZITI_EDGE_CONTROLLER_HOSTNAME}-intermediate/certs/${ZITI_EDGE_CONTROLLER_HOSTNAME}-server.chain.pem" "${ZITI_HOME}/ziti-console/server.chain.pem"
ln -s "${ZITI_PKI}/${ZITI_EDGE_CONTROLLER_HOSTNAME}-intermediate/keys/${ZITI_EDGE_CONTROLLER_HOSTNAME}-server.key" "${ZITI_HOME}/ziti-console/server.key"
```
[Optional] Edit the Ziti Console systemd file and update systemd to start the Ziti Console. If you have not sourced the Ziti helper script, you need to in order to get the necessary function.
```
createZacSystemdFile
sudo cp "${ZITI_HOME}/ziti-console.service" /etc/systemd/system
sudo systemctl daemon-reload
sudo systemctl enable --now ziti-console
```
If you do not have systemd installed or if you just wish to start ZAC you can simply issue:
```
node "${ZITI_HOME}/ziti-console/server.js"
```
Output:
```
Initializing TLS
TLS initialized on port: 8443
Ziti Server running on port 1408
```
[Optional] If using systemd - verify the Ziti Console is running by running the systemctl command 
```
$ sudo systemctl status ziti-console --lines=0 --no-pager
```
Output: 
```
ziti-console.service - Ziti-Console
 Loaded: loaded (/etc/systemd/system/ziti-console.service; disabled; vendor preset: enabled)
 Active: active (running) since Wed 2021-05-19 22:04:44 UTC; 13h ago
 Main PID: 13458 (node)
 Tasks: 11 (limit: 1160)
 Memory: 33.4M
 CGroup: /system.slice/ziti-console.service
 └─13458 /usr/bin/node /home/ubuntu/.ziti/quickstart/ip-172-31-22-212/ziti-console/server.js
```
Verify Nodes and ports:
```
$ sudo ss -lntp | grep node
LISTEN 0      511                *:8443             :    users:(("node",pid=26013,fd=19))           
LISTEN 0      511                *:1408             :    users:(("node",pid=26013,fd=18))
```
 

Login and use ZAC
Now we can access from anywhere

https://${ZITI_EDGE_CONTROLLER_HOSTNAME}:8443   E.g https://157.245.203.171:8443/

At this point you should be able to navigate to both: https://${ZITI_EDGE_CONTROLLER_HOSTNAME}:8443and see the ZAC login screen. (The TLS warnings your browser will show you are normal - it's because these steps use a self-signed certificate generated in the install process)

Set the controller as shown (use the correct URL):

set edge controller name as the hostname of the controller

set url https://hostname:8441  click the setcontroller

login the username and password
![Diagram](imag/console.png)
