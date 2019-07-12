# Getting Started with Intel® RFID Sensor Platform (RSP)

The instructions in this getting started guide will rapidly get an Intel RSP configuration up and running in your lab to prepare for solution development. The steps in this guide apply whether you provide your own hardware or buy the Intel RSP Developer Kit *(UPDATE NAME)*, available from the [atlasRFIDstore](https://www.atlasrfidstore.com/intel-rsp-h3000-integrated-rfid-reader-development-kit/).

The Intel RSP gateway software is open source and fully functional, so you can easily build your own solution. The web-based application included in the distribution demonstrates the capabilities of Intel RSP by building on the API to collect and process RFID tag information.

##### *The web portal shown in this guide is intended to be a complete end-to-end inventory management solution. It is a demonstration of what the platform can do.*

Other than the web portal, all the software pieces installed in this getting started guide, including the MQTT and CLI functionality and the underlying gateway application, are core components of the platform.

## Contents
- System Overview
	- Solution Structure
	- Network Map for Getting Started
- System Requirements
	- Hardware
	- Software
- Setting up the Hardware
- Installing and Running the Gateway on Windows
- Installing the Gateway on Linux
	- Install prerequisites (not needed for RDK)
	- Clone gateway repository (not needed for RDK)
	- Build and deploy (not needed for RDK) 
- Running the Gateway on Linux
	- Start gateway software
	- Open the web portal
- View RFID data in other ways
	- Subscribe to MQTT topics
	- Explore the command-line interface
- Next Steps

## System Overview

### Solution Structure

The image below is an example of a robust inventory management system built on Intel RSP. The RSP reader activates the RFID tags within its range and passes tag data, along with information from other on-board sensors, to the gateway software running on a PC. The gateway aggregates the high volume of raw data and generates meaningful inventory events (e.g., "item exited"), alerts, and system status notifications. It publishes them to MQTT topics on an upstream channel, available to applications running in a customer’s cloud infrastructure.

![](https://baychub.github.io/cb-gsg/solution-map.png)

### Network Map for Getting Started
This getting started tutorial uses a simple network configuration, shown in the image below. Key points about the setup:

 - The RSP sensor units are powered by a PoE+ switch and get their IP address from a DHCP-enabled router that the switch is connected to.
 - The RSP gateway software runs on a PC, which must be on the same network segment.
 - The gateway software automatically pairs with the RSP sensors. 
 - A simple command-line interface on the gateway PC enables troubleshooting and low-level data monitoring.
 - (Demo only, not meant for production) A web-based portal displays data gathered by the gateway software over an MQTT channel. The web portal allows you to make configuration changes as well. 

![](https://baychub.github.io/cb-gsg/demo-map.png)


## System Requirements
In the items below, *RDK* refers to the Intel RSP Developer Kit. You can buy the kit, or you can provide your own hardware.

### Hardware
 - Any PC that can run the JRE and operating system (if you have the RDK, an Intel NUC)
 - Ethernet router with DHCP enabled (if you have the RDK, model xxx)
 - Ethernet switch that supports PoE+ (if you have the RDK, model yyy)
 - At least one Intel RSP sensor unit, preferably two, from models H1000, H3000, or H4000 (if you have the RDK, one H1000 and one H3000)
 - Sample RFID tags to test with
 - Ethernet cables

### Software
 - Java Runtime Environment (JRE), version 8 or later (if you have the RDK, preinstalled)
 - Any  operating system that can run the JRE (if you have the RDK, Ubuntu 18.04)

## Setting up the Hardware

Whether you're using the RSP Developer Kit or provided your own components, the steps for setting up your environment are the same.
1. Power on the router first. 
(Each RSP sensor unit needs a DHCP server when it starts up.)
2. Connect the PoE+ switch to the router and power it on.
3. Connect the router to the internet.
4. Connect each RSP sensor to a PoE+ port on the switch.
5. Connect the gateway PC to the router and power it on.
6. Attach a monitor to the gateway PC and power it on.

## Installing and Running the Gateway on Windows

Instructions for build and installation in a Windows environment are in the [Gateway Installation & User's Guide](https://github.com/intel/rsp-sw-toolkit-gw/blob/master/docs/338088-002_Intel-RSP_User_Guide.pdf), section 2.2.

## Installing the Gateway on Linux (Recommended)
This section assumes an Ubuntu 18.04 installation, preinstalled on the RDK, but but other Linux distributions compatible with JRE 8+ should also be compatible.
1. If you have the Intel RSP Developer Kit (RDK), skip ahead to "[Run gateway software](#run_gateway_linux)" because the software is preinstalled. 
2. If you provided your own gateway PC, carry out all of the following steps.

### Install prerequisites (not needed for RDK)
1. Run these commands to get software required for building the gateway application. 
```
#-- install development dependencies
sudo apt-get install default-jdk git gradle
```
2. Run these commands to get software required for the gateway to communicate. 
```
#-- install runtime dependencies
sudo apt-get install mosquitto mosquitto-clients avahi-daemon ntp ssh
```
### Clone gateway repository (not needed for RDK)
1. Run these commands to clone the RSP gateway repository in a new directory. This guide assume you've used the ~/projects directory, but you can change the path to wherever you want to clone the directory where you see the projects directory in later steps.
```
#-- create expected directories for the use case examples and documentation
mkdir -p ~/projects
#-- clone the reference source code
cd ~/projects
git clone https://github.com/intel/rsp-sw-toolkit-gw.git
```

### Build and deploy (not needed for RDK)
These components are preinstalled if you have the RSP Developer Kit. If you don't, go to the Linux PC you're using for the RSP gateway software and follow the steps in tthis section.
1. Run these commands to build the gateway software, using the gradle utility.

```
cd ~/projects/rsp-sw-toolkit-gw

#-- The gradle build script will trigger a full build
#-- and a local deployment to the directory ~/deploy/rsp-sw-toolkit-gw 
gradle clean deploy
```
2. Run these commands to generate certificates for authentication between gateway and sensors for a secure connection.
```
#-- !!!! CERTIFICATES ARE NEEDED !!!!
#-- The sensors connect securely to the gateway using a self-signed certificate.
#-- A convenience script generates these credentials in order to get the
#-- reference deployment up and running quickly.
mkdir -p ~/deploy/rsp-sw-toolkit-gw/cache
cd ~/deploy/rsp-sw-toolkit-gw/cache
~/deploy/rsp-sw-toolkit-gw/gen_keys.sh
```

## <a id="run_gateway_linux"></a>Running the gateway on Linux
The gateway software must be running in order to gather, process, and report data form RFID tags. You'll build your own solution that communicates with the gateway over MQTT, but the web portal in this section will give you an idea of what kind of data the gateway makes available.

### Start gateway software
A shell script starts the gateway application in the foreground. 

1. Run these commands to start the application: 
```
cd ~/deploy/rsp-sw-toolkit-gw
~/deploy/rsp-sw-toolkit-gw/run.sh
```
The RSP sensors listen for messages from the gateway and initiate a connection with it. As the sensors connect, by default the gateway schedules them to activate in round-robin sequence, one at a time. You can watch the terminal output to see connections taking place and RFID tags being read.

You can run the web portal (next section) at any time after starting the gateway software because the dashboard refreshes automatically. The web portal dashboard will be ready to display RFID data when you see a screen like the following:
![enter image description here](add-screenshot-URL-here)

### Open the web portal
The gateway provides, as sample software, a web-based administration interface for configuration and RFID monitoring. The home page is a dashboard showing connected sensors, tags being read, and other status information.

1. Open the web portal from the gateway PC by going to http://localhost:8080/web-admin/ in a browser.
2. Navigate through the portal by choosing a page from the three-bar menu on the upper left of the display.

![enter image description here](add-screenshot-URL-here)

## Viewing RFID data in other ways
You've now used the RSP web portal to see a sample of some of the platform's capabilities. In this section, you'll use some RSP building blocks of data that you'll use for your own RFID solution:
 - **The MQTT communications protocol.** The RSP gateway uses a built-in MQTT broker to deliver data structured as JSON-RPC. Many libraries support the well-known MQTT and JSON-RPC protocols, so it's straightforward to implement data access in the code for your solution.
 - **The RSP gateway CLI.** The RSP gateway includes a simple, local command-line interface for accessing data and also for making sensor configuration changes. 

### Subscribe to MQTT topics
You can open a terminal window and subscribe to the gateway events topic in order to monitor 
tag events produced by the gateway, raw RFID tag data, and other RSP system information. 

The RSP web portal displays many of the available MQTT topics (specific sets of data you can subscribe to) in the Upstream and Downstream panes of the Dashboard page, e.g., "rfid/gw/alerts". The RSP gateway divides its MQTT topics into two groups:
- Downstream: data the gateway pulls from or sends to the sensors
- Upstream (processed data the gateway makes available out on the network to go into a database, management app, etc.)

1. To see RSP data over MQTT from a terminal window:
```
#-- See data from the gateway events topic
mosquitto_sub -t rfid/gw/events
```
The console displays in JSON-RPC format any high-level events processed by the gateway (e.g., an RFID tag has appeared for the first time) while you're monitoring this topic. If you don't see any events initially, try introducing or moving some tags.
![enter image description here](add-screenshot-URL-here)

2. Press Ctrl-C to stop monitoring MQTT data on this topic.
3. Try some of the other MQTT topics to see other types of data the gateway publishes:
```
#-- See raw data from all tag reads
mosquitto_sub -t rfid/rsp/data/#
#-- See changes in gateway status, e.g., gateway app starts or shuts down
mosquitto_sub -t rfid/rsp/data/#
#-- See the status of all connected RSP sensors
mosquitto_sub -t rfid/rsp/rsp_status/#
```
Full documentation of the MQTT topics and data definitions can be found in the *[Intel® RSP SW Toolkit – Gateway Application Interface (API)](https://github.com/intel/rsp-sw-toolkit-gw/blob/master/docs/338971-0001_Intel-RSP-SW-Toolkit-Gateway-API.pdf)* guide, chapters 2 and 3.

### Explore the command-line interface
The gateway provides a command line interface (CLI) for configuration and monitoring, available locally on the gateway PC. The CLI is ideal for troubleshooting, simple configuration changes, or a quick check on live data.

1. To connect to the gateway CLI:
```
ssh -p5222 gwconsole@localhost
password: gwconsole
```
![enter image description here](add-screenshot-URL-here)

2. To view summary information about sensors connected to this gateway, enter `view sensor information`:
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
Nearly all RFID tag readings are redundant because, for example, retail items sat on the shelf most of the time. The gateway software processes the high volume of readings into useful data, e.g., this one entered, this one moved from here to here, this one exited. 

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
4. To see a list of available commands, at the gateway CLI prompt press Enter with no text.
```
[Show command help terminal output]
```
5. Commands may have several layers of parameters, so the tab completion feature helps. At the gateway CLI prompt type `inventory` and a space character, then press the Tab key.
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
In this way you can see the syntax options progressively without referring to documentation. Full documentation of the CLI is in the *[Intel® RSP SW Toolkit – Gateway Installation & User's Guide](https://github.com/intel/rsp-sw-toolkit-gw/blob/master/docs/338443-002_Intel-RSP-SW-Toolkit-Gateway.pdf)*, chapter 9.
To view command help, type the command and press Enter.

## Next Steps
### See sample use cases
Explore the reference use cases that are included.  
[Retail](examples/use-cases/retail)

