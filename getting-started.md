﻿# Getting Started with Intel® RFID Sensor Platform (RSP)

The instructions in this getting started guide will rapidly get an Intel RSP configuration up and running in your lab to prepare for solution development. The steps in this guide apply whether you provide your own hardware or buy the Intel RSP Developer Kit (RDK), available from the [atlasRFIDstore](https://www.atlasrfidstore.com/intel-rsp-h3000-integrated-rfid-reader-development-kit/).

Intel RSP Controller Application software is open source, so you can easily build out your own solution. The web-based application included in the distribution demonstrates the capabilities of Intel RSP by building on the API to collect and process RFID tag information.

##### *The web admin portal shown in this guide is not intended to be a complete end-to-end inventory management solution. It is a demonstration of what the platform can do.*

All other software installed in this getting started guide is core to the platform.

## Contents
- System Overview
	- Solution Structure
	- Network Map for Getting Started
- System Requirements
	- Hardware
	- Software
- Setting up the Hardware
- Installing and Running the RSP Controller on Windows
- Installing the RSP Controller on Linux (Recommended)
	- Clone RSP Controller repository
	- Build and deploy
- Running the RSP Controller Application on Linux
	- Start the RSP Controller Application
- Using the Web Portal
	- Open the web portal
	- ((Other web portal tasks here))
- Viewing RFID data in other ways
	- Subscribe to MQTT topics
	- Use the command-line interface
- Next Steps

## System Overview

### Solution Structure

The image below is an example of a robust inventory management system built on Intel RSP. The RSP reader activates the RFID tags within its range and passes tag data, along with information from other on-board sensors, to the RSP Controller gateway software running on a PC. The RSP Controller aggregates the high volume of raw data and generates meaningful inventory events (e.g., "item exited"), alerts, and system status notifications. It publishes them to MQTT topics on an upstream channel, available to applications running in a customer’s cloud infrastructure.

The PC doesn't need to be dedicated to running RSP Controller software. It can also run other workloads and be available for future expansion.

![](https://baychub.github.io/cb-gsg/solution-map.png)

### Network Map for Getting Started
This getting started tutorial uses a simple network configuration, shown in the image below. Key points about the setup:

 - RSP sensor units are powered by a PoE+ switch and get their IP address from a DHCP-enabled router that the switch is connected to.
 - RSP Controller gateway software runs on a PC, which must be on the same network segment.
 - RSP Controller software automatically pairs with the RSP sensors. 
 - A simple command-line interface on the edge computer enables troubleshooting and low-level data monitoring.
 - (Demo only, not meant for production) A web-based portal displays data gathered by the RSP Controller software over an MQTT channel. The web portal allows you to make configuration changes as well. 

![](https://baychub.github.io/cb-gsg/demo-map.png)


## System Requirements
In the items below, *RDK* refers to the Intel RSP Developer Kit. You can buy the kit, or you can provide your own hardware.

### Hardware
 - Any PC that can run the JRE and operating system *(if you have the RDK, an Intel NUC)*
 - Ethernet router with DHCP enabled *(if you have the RDK, model xxx)*
 - Ethernet switch that supports PoE+ *(if you have the RDK, model yyy)*
 - At least one Intel RSP sensor unit, preferably two, from models H1000 and H3000 *(RDK includes either one H1000 or two H3000 sensors)*
 - Sample RFID tags to test with
 - Ethernet cables

### Software
 - Java Runtime Environment (JRE), version 8 or later *(provided as part of RSP installation)*
 - Any operating system that can run the JRE *(if you have the RDK, Ubuntu 18.04 preinstalled)*

## Setting up the Hardware

Whether you're using the RSP Developer Kit or provided your own components, the steps for setting up your environment are the same.
1. Connect the router to the internet.
2. Power on the router before connecting other devices. 
	(Each RSP sensor unit needs a DHCP server when it starts up.)
3. Connect the PoE+ switch to the router and power it on.
4. Connect the RSP Controller PC to the router and power it on.
5. If your RSP sensor model is the H1000, attach the two external antennas (using antenna cables or adapters).
6. Connect each RSP sensor to a PoE+ port on the switch.
	If **[insert problem symptom]**, **[insert troubleshooting solution]**.
7. Attach a monitor to the RSP Controller PC and power it on.

## Installing and Running the RSP Controller on Windows

To build and install in a Windows environment:

1. Install the dependencies in section 2.2 of the [RSP Controller App Installation & User's Guide](https://github.com/intel/rsp-sw-toolkit-gw/blob/master/docs/Intel-RSP-Controller-App_Installation_User_Guide.pdf).
2. Run the [build-win10.sh](https://github.com/intel/rsp-sw-toolkit-installer/blob/master/native/build-win10.sh) script from the RSP Toolkit Installer project on GitHub.
3. After installing, go to the "Open the web portal" section below.

The remainder of this document assumes a Linux installation.

## Installing the RSP Controller on Linux (Recommended)
This document assumes an Edge Computer running Ubuntu 18.04, which is preinstalled on the RDK, but other Linux distributions compatible with JRE 8+ should also be compatible with RSP. The steps here enable a quick installation with the RSP Installer software. (You can also [install manually](manual-install-link) or [install with Docker](docker-install-link).)

### Clone RSP Installer repository
1. Open a terminal window on the Edge Computer.
2. Run these commands to clone the RSP Installer repository in a new directory:
```
#-- create expected directories for the use case examples and documentation
mkdir -p ~/projects
#-- clone the reference source code
cd ~/projects
sudo apt –y install git
git clone https://github.com/intel/rsp-sw-toolkit-installer.git
```
This guide assumes you've used the ~/projects directory, but you can change this path to your own projects directory in later steps.

### Build and deploy
1. Open a terminal window.
2. Run these commands to build and run the Controller software:
```
#-- The RSP installer builds and deploys the RSP Controller software on your edge computer 
The build script will trigger a full build and deployment
#-- to the directory ~/deploy/rsp-sw-toolkit-gw 
cd ~/projects/rsp-sw-toolkit-installer/native/
./build.sh
```
When complete, the web portal will open in the default browser. If you close the browser window, you can reach the portal with this URL: http://localhost:8080/web-admin/

## <a id="run_controller_linux"></a>Running the RSP Controller Application on Linux
The RSP Controller Application must be running in order to gather, process, and report data form RFID tags. You'll build your own solution that communicates with the RSP Controller over MQTT, but the web portal gives you an idea of what kind of data the RSP Controller Application makes available.

The installer above runs the RSP Controller Application software and launches the web portal. If you reboot or close the terminal window, you will need to run these steps in this section.

### Start the RSP Controller Application
A shell script starts the RSP Controller Application in the foreground. 
1. Run these commands to start the application: 
```
cd ~/deploy/rsp-sw-toolkit-gw
~/deploy/rsp-sw-toolkit-gw/run.sh
```
The RSP sensors listen for messages from the RSP Controller Application and initiate a connection with it. As the sensors connect, by default the RSP Controller Application schedules them to activate in round-robin sequence, one at a time. You can watch the terminal output to see connections taking place and RFID tags being read.

You can run the web portal (next section) at any time after starting the RSP Controller software because the dashboard refreshes automatically. The web portal dashboard will be ready to display RFID data when you see a screen like the following:

![enter image description here](add-screenshot-URL-here)


## Using the Web Portal
The RSP Controller Application provides, as sample software, a web-based administration interface for configuration and RFID monitoring. The home page is a dashboard showing connected sensors, tags being read, and other status information.

### Open the web portal
While the RSP Controller Application software is running, you can use the web portal.
1. If the web portal is not already open, open it from the Edge Computer by going to http://localhost:8080/web-admin/ in a browser.
2. Navigate through the portal by choosing a page from the three-bar menu on the upper left of the display.

![enter image description here](add-screenshot-URL-here)

### Other web portal tasks here
While the RSP Controller Application software is running, you can use the web portal.

## Viewing RFID data in Other Ways
You've now used the RSP web portal to see a sample of some of the platform's capabilities. In this section, you'll use some RSP building blocks of data that you'll use for your own RFID solution:
 - **The MQTT communications protocol.** The RSP Controller uses a built-in MQTT broker to deliver data structured as JSON-RPC. Many libraries support the well-known MQTT and JSON-RPC protocols, so it's straightforward to implement data access in the code for your solution.
 - **The RSP Controller CLI.** The RSP Controller includes a simple, local command-line interface for accessing data and also for making sensor configuration changes. 

### Subscribe to MQTT topics
You can open a terminal window and subscribe to the RSP Controller events topic in order to monitor 
tag events produced by the RSP Controller, raw RFID tag data, and other RSP system information. 

The RSP web portal displays many of the available MQTT topics (specific sets of data you can subscribe to) in the Upstream and Downstream panes of the Dashboard page, e.g., "rfid/gw/alerts". The RSP Controller divides its MQTT topics into two groups:
- Downstream: data the RSP Controller pulls from or sends to the sensors
- Upstream: processed data the RSP Controller makes available out on the network to go into a database, management app, etc.

1. To see RSP data over MQTT from a terminal window:
```
#-- See data from the RSP Controller events topic
mosquitto_sub -t rfid/gw/events
```
The console displays in JSON-RPC format any high-level events processed by the RSP Controller (e.g., an RFID tag has appeared for the first time) while you're monitoring this topic. If you don't see any events initially, try introducing or moving some tags.
![enter image description here](add-screenshot-URL-here)

2. Press Ctrl-C to stop monitoring MQTT data on this topic.
3. Try some of the other MQTT topics to see other types of data the RSP Controller publishes:
```
#-- See raw data from all tag reads
mosquitto_sub -t rfid/rsp/data/#
#-- See changes in RSP Controller status, e.g., RSP Controller app starts or shuts down
mosquitto_sub -t rfid/rsp/data/#
#-- See the status of all connected RSP sensors
mosquitto_sub -t rfid/rsp/rsp_status/#
```
Full documentation of the MQTT topics and data definitions can be found in the *[Intel® RSP Controller - Edge Computer Software Application Programming Interface (API)](https://github.com/intel/rsp-sw-toolkit-gw/blob/master/docs/Intel-RSP-Controller-App_API_Guide.pdf)* guide, chapters 2 and 3.

### Use the command-line interface
The RSP Controller provides a command line interface (CLI) for configuration and monitoring, available locally on the RSP Controller PC. The CLI is ideal for troubleshooting, simple configuration changes, or a quick check on live data.

1. To connect to the RSP Controller CLI:
```
ssh -p5222 gwconsole@localhost
password: gwconsole
```
![enter image description here](add-screenshot-URL-here)

2. To view summary information about sensors connected to this RSP Controller, enter `view sensor information`:

```
#-- view sensor information 
rfid-gw> sensor show
------------------------------------------
device     connect      reading    behavior  facility    personality  aliases

RSP-150000 CONNECTED    STOPPED    Default   SalesFloor               [RSP-150000-0, RSP-150000-1, RSP-150000-2, RSP-150000-3]
RSP-150004 CONNECTED    STOPPED    Default   SalesFloor  EXIT         [RSP-150004-0, RSP-150004-1, RSP-150004-2, RSP-150004-3]
RSP-150005 CONNECTED    STOPPED    Default   BackStock                [RSP-150005-0, RSP-150005-1, RSP-150005-2, RSP-150005-3]
------------------------------------------
```
Nearly all RFID tag readings are redundant because, for example, retail items sat on the shelf most of the time. The RSP Controller software processes the high volume of readings into useful data, e.g., this one entered, this one moved from here to here, this one exited. 

3. To see a processed summary of inventory (that is, tags read very recently), enter `view inventory summary`.
```
#-- view inventory information
rfid-gw> inventory summary 
------------------------------------------
- Total Tags: 26

- State
      26 PRESENT

- Last Seen
      26 within_last_01_min
------------------------------------------
```
4. To see a list of available commands, at the RSP Controller CLI prompt press Enter with no text.
```
[Show command help terminal output]
```
5. Commands may have several layers of parameters, so the tab completion feature helps. At the RSP Controller CLI prompt type `inventory` and a space character, then press the Tab key.
```
[Show command help terminal output]
```
6. After the CLI displays the available parameters, type `stats` and a space character, then press the Tab key.
```
[Show command help terminal output]
```
7. After the CLI displays the available parameters, type `show` and press Enter.
 ```
[Show command help terminal output]
```
In this way you can see the syntax options progressively without referring to documentation. To view command help, type the command and press Enter.

Full documentation of the CLI is in the *[Intel® RSP Controller Application - Edge Computer Software Installation & User Guide](https://github.com/intel/rsp-sw-toolkit-gw/blob/master/docs/Intel-RSP-Controller-App_Installation_User_Guide.pdf)*, chapter 9.


## Next Steps
- [Batch configuration tutorial](examples/use-cases/retail): Walkthrough of creating cluster files to configure behaviors and settings for multiple sets of RSP sensors
- [Use Cases](examples/use-cases/retail): Possible implementations for common use cases in areas like retail and factory floors
- [RSP Controller API](examples/use-cases/retail): Reference for getting data from and configuration commands to the RSP Controller software
- [Etc. (TBD)](examples/use-cases/retail): More to come
- 



<!--stackedit_data:
eyJoaXN0b3J5IjpbMTMzMjA3NjY1MSwxNjk0MjU4MjAzXX0=
-->