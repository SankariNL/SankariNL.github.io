---
layout: post
title:  "Node-RED on the Turck IM18-CCM50"
date:   2021-11-15 13:00:00 +0100
categories: IM18-CCM50 Node-RED Modbus
---
# Node-RED on the Turck IM18-CCM50
This tutorial explains how to install and use Node-RED on the Turck IM18-CCM50. More information about the IM18-CCM50 can be found on [the Turck website](https://www.turck.de/en/product/100022405).

[Node-RED](https://nodered.org/) is a programming tool for wiring together hardware devices, APIs and online services in new and interesting ways. It provides a browser-based editor that makes it easy to wire together flows using the wide range of nodes in the palette that can be deployed to its runtime in a single-click.

![CCM50NodeRED](/assets/img/CCM50NodeRed.png)

## Connect to the IM18-CCM50
By default the ethernet interfaces are set the DHCP. The default IP addresses when no DHCP server is active are 192.168.1.20 for ETH0 and 192.168.1.10 for ETH1. After powering up the device and connecting via ethernet it is possible to access the device via SSH. A client like [PuTTY](https://www.putty.org/) can be used. The user is `sshu` and the default password is `P@ssw0rd12ssh!` It is recommended to change this password after the first login. 

## Installing Node-RED
The [Node-RED documenation](https://nodered.org/docs/getting-started/local) tells us that first Node.js needs to be installed. In the [FAQ](https://nodered.org/docs/faq/node-versions) the recommended and supported versions of Node.js can be found. I recommend to use the most recent supported LTS release of Node.js. Check the [Node.js website](https://nodejs.org/en/about/releases/) for information about the current LTS release. At the time of writing Node.js V16 is the most recent LTS release.

The [Node.js documentation](https://nodejs.org/en/download/package-manager/#debian-and-ubuntu-based-linux-distributions) explains that an installation package for Node.js is provided by [NodeSource](https://github.com/nodesource/distributions/blob/master/README.md#installation-instructions). Use the following commands to install Node.js on the IM18-CCM50:
```bash
sudo curl -fsSL https://deb.nodesource.com/setup_16.x | bash -
sudo apt install -y nodejs
```

To install Node-RED the `npm` command can be used. Use the following command to install Node-RED:
```bash
sudo npm install -g --unsafe-perm node-red
```

Node-RED can be started by executing the following command via SSH:
```
node-red
```

Now Node-RED is available to access via a webbrowser at port 1880. For example: `https://192.168.1.10:1880`

To start Node-RED on boot, a script needs to be added. [The script is available on github](https://gist.github.com/SankariNL/afe080d53b90ffe19081e720b569a319). It can be installed with 1 command:
```
sudo wget -O /tmp/download https://gist.github.com/SankariNL/afe080d53b90ffe19081e720b569a319/download && sudo tar -zxf /tmp/download --strip-components 1 -C /etc/init.d && sudo chmod +x /etc/init.d/node-red && sudo update-rc.d node-red defaults
```
The file can also be manually added to the `/etc/init.d` folder via SCP for example. Then the `chmod` and `update-rc.d` commands need to be executed manually.

## Using the available scripts
When Node-RED is installed and running, it is possible to use the scripts that are delivered with the IM18-CCM50. These scripts are located in the `/home/scripts` directory as mentioned in chapter 7.3 of [the manual](https://www.turck.nl/attachment/100023797.pdf). Scripts can be executed by Node-RED by using the default `exec` node. To execute the `ambient_read.sh` script input the following as the command: `/home/scripts/ambient_read.sh`. 

There is a flow [available](https://flows.nodered.org/flow/64631bb920110a0fb6db3e0c8c765735) where the `ambient_read.sh` script is executed every 30 seconds (this can be changed after importing) and the result is added to the `msg` object. The flow looks like this:

![Ambient](/assets/img/Ambient_read-flow.png)

When observing de debug output it shows the payload object with the humidity and temperature:

![Payload object](/assets/img/PayloadObject.png)

## Installing additional Node-RED submodule for Modbus
To use Modbus in Node-RED an additional package needs to be installed. There are several packages available, in this tutorial we will use the [node-red-contrib-modbus](https://flows.nodered.org/node/node-red-contrib-modbus) package. Normally packages can be added by using the `manage palette` menu option. However, when trying this the following error shows up in the installation log: 
# TO DO

This is because the [SerialPort](https://www.npmjs.com/package/serialport) Node.js package could not be installed. The IM18-CCM50 uses a Texas instruments Sitara processor that is based on ARM v7. According to the [documentation of SerialPort](https://serialport.io/docs/guide-platform-support) this is not a supported platform, but will probably work. To get it to work we need to compile the SerialPort package from scratch. For this we need the `build-essential` and `python 2.7` packages.

Install the `build-essential` package to enable compiling of Node.js packages on Debian:
```
sudo apt install build-essential
```

Install the `python2.7` package because by default only the minimal python package is installed. SerialPort uses the `ast` package that is not included in the minimal python installation.
```
sudo apt install python2.7
```

Now compile and install the Node.js package with the following command:
```
Sudo npm install serialport --unsafe-perm --build-from-source
```

Now it is possible to install the `node-red-contrib-modbus` package via the manage palette function in Node-RED. After installing the new modbus related nodes show up on the left side.

## Using modbus in Node-RED
To use the modbus connection, first a modbus device needs to be connected. 
 To use the modbus serial connection use the `Modbus - Read` node to read a register from a connected modbus device.

 # TO DO