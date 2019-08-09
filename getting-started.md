# Getting Started with Intel® RFID Sensor Platform (RSP)

The instructions in this getting started guide will rapidly get an Intel RSP configuration up and running in your lab to prepare for solution development. The steps in this guide apply whether you provide your own hardware or buy the Intel RSP Developer Kit (RDK), available from the [atlasRFIDstore](https://www.atlasrfidstore.com/intel-rsp-h3000-integrated-rfid-reader-development-kit/).

Intel RSP Controller application software is open source, so you can easily build out your own solution. The web-based portal included in the distribution demonstrates the capabilities of Intel RSP by building on the API to collect and process RFID tag information.

*The web admin portal shown in this guide is not intended to be a complete end-to-end inventory management solution.* It is a demonstration of what the platform can do.* All other software installed in this getting started guide is production software.

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
- Running the RSP Controller application on Linux
	- Start the RSP Controller application
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
 - Sample RFID tags to test with *(included with RDK)*
 - Ethernet cables

### Software
 - Java Runtime Environment (JRE), version 8 or later *(provided as part of RSP installation)*
 - Ubuntu 18.04 *(preinstalled if you have the RDK)*, Debian 9, or Windows 10
 - Other operating systems that can run JRE v.8+ will likely work but have not been tested

## Setting up the Hardware

Whether you're using the RSP Developer Kit or provided your own components, the steps for setting up your environment are the same. The network map above shows the arrangement, but **the connection order is important**, so follow these steps.
1. If your RSP sensor is the H1000 model, attach two external antennas (included if you have the RDK) using antenna cables or adapters, and point them away from each other.

	This step is not needed for the H3000 model, which has two built-in antennas. 
	
2. Connect the router to the internet.
3. Power on the router before connecting other devices. 
	
	RSP sensor units need a DHCP server when they start up.
	
4. Connect each RSP sensor to a PoE+ port on the switch.
5. Connect the PoE+ switch to the router and power on the switch.

6. Check the LEDs on the RSP sensors.

	The LEDs will go through a series of patterns, and after 1-2 minutes they should be flashing pinkish-white. After the RSP Controller application starts and connects, the LEDs will be yellow (idle) or blue (reading tags). If the LEDs are not flashing as expected, repeat these connection steps and try these suggestions in the LED section of [this FAQ](https://01.org/rsp-sw-toolkit/faq-0). 
	
6. Connect the edge computer to the router and power it on.

	This is the PC where you will install the RSP Controller application.
	
7. Attach a monitor to the edge computer and power it on.

## Installing and Running the RSP Controller on Windows

To build and install in a Windows environment:

1. Install the dependencies in section 2.2 of the [RSP Controller App Installation & User's Guide](https://github.com/intel/rsp-sw-toolkit-gw/blob/master/docs/Intel-RSP-Controller-App_Installation_User_Guide.pdf).
2. Run the [build-win10.sh](https://github.com/intel/rsp-sw-toolkit-installer/blob/master/native/build-win10.sh) script from the RSP Toolkit Installer project on GitHub.
3. After installing, go to the "Open the web portal" section below.

The remainder of this document assumes a Linux installation.

## Installing the RSP Controller on Linux (Recommended)
This document assumes an edge computer running Ubuntu 18.04, which is preinstalled on the RDK, but other Linux distributions compatible with JRE 8+ should also be compatible with RSP. The steps here enable a quick installation with the RSP Installer software. (You can also [install manually](manual-install-link) or [install with Docker](docker-install-link).)

### Clone RSP Installer repository
1. Open a terminal window on the edge eomputer.
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
The RSP Controller application must be running in order to gather, process, and report data form RFID tags. You'll build your own solution that communicates with the RSP Controller over MQTT, but the web portal gives you an idea of what kind of data the RSP Controller application makes available.

The installer above doesn't just install. It also runs the RSP Controller application software and launches the web portal. If you reboot or close the terminal window, you will need to run the next steps to start the RSP Controller.

### Start the RSP Controller application
A shell script starts the RSP Controller in the foreground. 
1. If you just ran the installer, this task is already done and you can skip to the Using the Web Portal section.
2. Otherwise, run these commands to start the application: 
```
cd ~/deploy/rsp-sw-toolkit-gw
~/deploy/rsp-sw-toolkit-gw/run.sh
```
The RSP sensors listen for messages from the RSP Controller application and initiate a connection with it. As the sensors connect, by default the RSP Controller schedules them to activate in round-robin sequence, one at a time. You can watch the terminal output to see connections taking place and RFID tags being read.

You can run the web portal (next section) at any time after starting the RSP Controller software because the dashboard refreshes automatically. The web portal dashboard will be ready to display RFID data when you see a screen like the following:

![enter image description here](add-screenshot-URL-here)


## Using the Web Portal
Along with the RSP Controller application, Intel provides as sample software a web-based administration interface for configuration and RFID monitoring. The home page is a dashboard showing connected sensors, tags being read, and other status information.

### Open the web portal
While the RSP Controller application software is running, you can use the web portal.
1. Open the web portal on the edge computer by going to http://localhost:8080/web-admin/ in a browser.

![Dashboard page](https://baychub.github.io/cb-gsg/dashboard.png)

### Understand dashboard information
The dashboard page shows summary information about tags and sensors connected to this edge computer, along with the names of MQTT topics you can use to stream data or send configuration changes using an MQTT client:
* **Sensors.** Gives number of known RSP sensors and their connection status.
* **Tag States.** Gives counts of inventory status for recently read RFID tags.
* **Tag Reads.** Gives counts of RFID tag readings, grouped chronologically.
* **Upstream MQTT.** Provides *upstream* MQTT topics (i.e., management messages processed by the RSP Controller application).
* **Downstream MQTT.** Provides *downstream* MQTT topics (i.e., data gathered directly from the RSP sensors).
* **Scheduler.** Shows scheduling configuration. By default, RFID readings are done one sensor at a time, cycling through each connected sensor.

### See live RFID tag data
The Tag Statistics window shows low-level detail about each individual tag the RSP sensors communicate with, including signal strength and when it was last read.
1. Click the three-bar menu button to the left of the Dashboard page heading to open the navigation menu.
	[insert zoomed pic]
2. Click Tag Statistics.
![Tag Statistics page](https://baychub.github.io/cb-gsg/tag-statistics.png)
	List of tags is shown in the left column. The other columns show readings from each of the two antennas on each sensor.  The values are signal strength (green from strong, red for weak) and brightness or dimness for time since the last reading. The most recent and strongest reading is highlighted in gray as the most likely current location for each tag.
3. Click any data point to see the legend for these readings and colors.
4. Experiment with moving RFID tags around the room to see how the sensor readings respond.

### See digested inventory data
Each RSP sensor reads each reachable RFID tag many times a second, generating a huge volume of raw data. The RSP Controller application processes that raw data stream into meaningful status and events. 

For example, thousands of consecutive readings of the same tag by the same sensor can be compressed into a state of PRESENT with that sensor. Or a tag that has no current reading but whose last reading was at the exit sensor would have a status of EXIT. The Inventory window shows these computed status values for each known tag.
1. Click the three-bar menu button to the left of the Tag Statistics page heading to open the navigation menu.
2. Click Inventory.
![Inventory page](https://baychub.github.io/cb-gsg/inventory.png)
3. Try placing a tag some distance away or under a thick piece of metal to simulate exiting, and see how the display changes for that tag's EPC number.


## Viewing RFID data in Other Ways
You've now used the RSP web portal to see a sample of some of the platform's capabilities. In this section, you'll use some RSP building blocks of data that can be part of your own RFID solution:
 - **The MQTT communications protocol.** The RSP Controller uses a built-in MQTT broker to deliver data structured as JSON-RPC. Many libraries support the well-known MQTT and JSON-RPC protocols, so it's straightforward to implement data access in the code for your solution.
 - **The RSP Controller CLI.** The RSP Controller includes a simple, local command-line interface for accessing data and also for making sensor configuration changes. 

### Get data over MQTT
You can open a terminal window and subscribe to the RSP Controller events topic in order to monitor tag events produced by the RSP Controller, raw RFID tag data, and other RSP system information. 

The RSP web portal displays many of the available MQTT topics (specific sets of data you can subscribe to) in the Upstream MQTT and Downstream MQTT panes of the Dashboard page, e.g., "rfid/controller/alerts". The RSP Controller divides its MQTT topics into two groups:
- Downstream: data the RSP Controller pulls from or sends to the sensors
- Upstream: processed data the RSP Controller makes available out on the network to go into a database, management app, etc.

To monitor RSP data over MQTT from a terminal window:
1. Run the mosquitto_sub command and specify the MQTT topic for the data you want (in this example, controller events).
```
#-- See data from the RSP Controller events topic
mosquitto_sub -t rfid/controller/events
```
The console displays in JSON-RPC format any high-level events processed by the RSP Controller (e.g., an RFID tag has appeared for the first time) while you're monitoring this topic. If you don't see any events initially, try introducing or moving some tags to generate an event.
![Dashboard page](terminal screenshot for MQTT topic)

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

3. Sensors are referred to be their ID and antenna port (e.g., RSP150944-1). To assign a human-readable name as an alias for the first antenna (`PORT_0`) of a sensor, find the sensor's ID from the web admin Dashboard page and enter this command at the CLI prompt:
```
sensor set.alias <sensorID> PORT_0 "newAlias"
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
eyJoaXN0b3J5IjpbODg3MTQ0Njk5LC0yMTE4NjY1ODc5LC0xMj
cxMDc1MjA1LC01MTcwNTYxMjIsMTkwOTE1MzExOSwyODY2NDE0
MDEsLTEwNjUyOTM0MjUsMTIzMjI2MDY2OCwxMjU5MzUxMzAzLC
0xNjQ4MjA3MDIzLDI1MDEzMTU1LC02NzY3NzQ1NiwxNDg4MTAw
OTAyLDE2OTQyNTgyMDNdfQ==
-->