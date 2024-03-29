# Getting Started with Intel® RFID Sensor Platform (RSP) on Linux

The instructions in this getting started guide will rapidly get an Intel RSP configuration up and running in your lab to prepare for solution development. The steps in this guide apply whether you provide your own hardware or buy the Intel RSP Developer Kit (RDK), available from the [atlasRFIDstore](https://www.atlasrfidstore.com/intel-rsp-h3000-integrated-rfid-reader-development-kit/).

Intel RSP Controller application software is open source, so you can easily build out your own solution. The web-based portal included in the distribution demonstrates the capabilities of Intel RSP by building on the API to collect and process RFID tag information.

*The web admin portal shown in this guide is not intended to be a complete end-to-end inventory management solution.* It is a demonstration of what the platform can do.* All other software installed in this getting started guide is production software.

## Contents
  * [System Overview](#system-overview)
  * [System Requirements](#system-requirements)
  * [Setting up the Hardware](#setting-up-the-hardware)
  * [Installing the RSP Controller](#installing-the-rsp-controller)
  * [Running the RSP Controller Application](#running-the-rsp-controller-application)
  * [Using the Web Portal](#using-the-web-portal)
    + [Open the web portal](#open-the-web-portal)
    + [Understand dashboard information](#understand-dashboard-information)
    + [See live RFID tag data](#see-live-rfid-tag-data)
    + [See digested inventory data](#see-digested-inventory-data)
    + [Turn scheduling on or off](#turn-scheduling-on-or-off)
  * [Viewing RFID Data with MQTT](#viewing-rfid-data-in-other-ways)
  * [Getting data over MQTT](#getting-data-over-mqtt)
    + [Subscribe to an MQTT data topic](#subscribe-to-an-mqtt-data-topic)
  * [Configuring RSP Controller for Your Use Case](#configuring-rsp-controller-for-your-use-case)
    + [Assign a personality to a sensor](#assign-a-personality-to-a-sensor)
  * [Next Steps](#next-steps)

## System Overview

### Solution Structure

The image below is an example of a robust inventory management system built on Intel RSP. The RSP reader activates the RFID tags within its range and passes tag data, along with information from other on-board sensors, to the RSP Controller application running on an on-premises computer. The computer and RSP Controller together are known as the *edge computer*, which functions as a data gateway for connected RSP sensors.

The RSP Controller aggregates the high volume of raw data and generates meaningful inventory events (e.g., "item exited"), alerts, and system status notifications. It publishes them to MQTT topics on an upstream channel, available to applications running in a customer’s cloud infrastructure.

The PC doesn't need to be dedicated to running RSP Controller software. It can also run other workloads and be available for future expansion.

![](https://baychub.github.io/cb-gsg/images/solution-map.png)

### Getting Started Components
This getting started tutorial uses a simple network configuration. Key points about the components:

 - **RSP sensor** units are powered by a PoE+ switch and get their IP address from a DHCP-enabled router that the switch is connected to.
 - The **RSP Controller application** runs on the edge computer (an on-premises computer that controls and processes data from connected sensors), which must be on the same network segment. This application automatically pairs with RSP sensors. 
 - An **MQTT interface** on the edge computer, provided by the RSP Controller application, enables standards-based access to data from RFID.
 - (Demo only, not meant for production) A **web-based admin** portal displays data gathered by the RSP Controller software over an MQTT channel. The web admin portal allows you to make configuration changes as well. 

## System Requirements
In the items below, *RDK* refers to the Intel RSP Developer Kit. You can buy the kit, or you can provide your own hardware.

### Hardware
 - Any computer that can run the JRE and operating system *(if you have the RDK, an Intel NUC)*
 - Ethernet router with DHCP enabled *(if you have the RDK, model xxx)*
 - Ethernet switch that supports PoE+ *(if you have the RDK, model yyy)*
 - At least one Intel RSP sensor unit, preferably two, from models H1000 and H3000 *(RDK includes either one H1000 or two H3000 sensors)*
 - Sample RFID tags to test with *(included with RDK)*
 - Ethernet cables

### Software
 - Ubuntu 18.04 *(preinstalled if you have the RDK)*, Debian 9, or Windows 10
 - Other operating systems that can run JRE v.8+ will likely work but have not been tested

## Setting up the Hardware

Connections are slightly different depending on whether you have the RSP H1000 sensor or the RSP H3000 sensor.

**RSP H1000 Sensor**
![Network Map](https://baychub.github.io/cb-gsg/images/demo-map-h1000.png)

**RSP H3000 Sensor**
![Network Map](https://baychub.github.io/cb-gsg/images/demo-map-h3000.png)
Whether you're using the RSP Developer Kit or provide your own components, the steps for setting up your environment are the same. The network map above shows the arrangement, but **the connection order is important**, so follow these steps in order:

1. If your RSP sensor is the H1000 model, attach two external antennas using antenna cables or adapters, and point them away from each other. (If you have the RDK, antennas are preassembled.)

	This step is not needed for the H3000 model, which has two built-in antennas. 
	
2. Connect the router's Internet/WAN/Uplink port to your internet connection with an Ethernet cable.
3. Power on the router before connecting other devices. 
	
	RSP sensor units need a DHCP server when they start up to get an IP address and communicate with the RSP Controller application.
	
4. Connect the PoE+ switch to the router and power on the switch.

	RSP sensors will get their IP addresses from the DHCP server on the router.

5. Connect each RSP sensor to a PoE+ port on the switch. This must happen **after** the router and PoE+ switch are powered on.

6. Check the LEDs on the RSP sensors.

	The LEDs will go through a series of patterns, and after 1-2 minutes they should be flashing pinkish-white. The LEDs will continue flashing pinkish-white until you install and run the RSP Controller application, which will connect to the sensors and enable them to begin reading tags.
	
	If the LEDs are not flashing pinkish-white as expected in this step, disconnect and reconnect the sensors. If that doesn't work, repeat this whole section. Next, try the suggestions in the LED section of [this FAQ](https://01.org/rsp-sw-toolkit/faq-0). 
	
7. Connect the edge computer to the router and power it on.

	This is the computer where you will install the RSP Controller application next.
	
8. Attach a monitor to the edge computer and power it on.

	In the meantime, LED patterns on the RSP sensors show startup progress.

	![LED Reference](https://baychub.github.io/cb-gsg/images/led-reference.png)

## Installing the RSP Controller
This document assumes an edge computer running Ubuntu 18.04, which is preinstalled on the RDK, but other Linux distributions compatible with JRE 8+ should also be compatible with RSP. The steps here enable a quick installation with the RSP Installer software. (Alternatively, you can [install manually](https://github.com/baychub/cb-gsg/blob/master/Manual-Install-Linux.md) or [install with Docker](https://github.com/intel/rsp-sw-toolkit-installer/).)

### Clone RSP Installer repository
1. Open a terminal window on the edge computer.
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
#-- The build script will trigger a full build and deployment
#-- to the directory ~/deploy/rsp-sw-toolkit-gw 
cd ~/projects/rsp-sw-toolkit-installer/native/
./build.sh
```
When complete, the web portal will open in the default browser. If you close the browser window, you can reach the portal with this URL: http://localhost:8080/web-admin/

## Running the RSP Controller Application
The RSP Controller application must be running in order to gather, process, and report data form RFID tags. You'll build your own solution that communicates with the RSP Controller over MQTT, but the web portal gives you an idea of what kind of data the RSP Controller application makes available.

The installer above doesn't just install. It also runs the RSP Controller application software and launches the web portal. If you reboot or close the terminal window, you will need to run the next steps to start the RSP Controller.

### Start the RSP Controller application
A shell script starts the RSP Controller in the foreground. 
1. If you just ran the installer, this task is already done and you should skip to the Using the Web Portal section.
2. Otherwise, run these commands to start the application: 
```
cd ~/deploy/rsp-sw-toolkit-gw
~/deploy/rsp-sw-toolkit-gw/run.sh
```
The RSP sensors listen for messages from the RSP Controller application and initiate a connection with it. As the sensors connect, by default the RSP Controller schedules them to activate in round-robin sequence, one at a time. You can watch the terminal output to see connections taking place and RFID tags being read.

Trying running the web portal (next section) a minute or two after running the RSP Controller application.

## Using the Web Portal
Along with the RSP Controller application, Intel provides as sample software a web-based administration interface for configuration and RFID monitoring. The home page is a dashboard showing connected sensors, tags being read, and other status information.

### Open the web portal
While the RSP Controller application software is running, you can use the web portal.
1. Open the web portal on the edge computer by going to http://localhost:8080/web-admin/ in a browser.

![Dashboard page](https://baychub.github.io/cb-gsg/images/dashboard.png)

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
2. 
	![](https://baychub.github.io/cb-gsg/images/menu.png)
	
3. Click Tag Statistics.

	![Tag Statistics page](https://baychub.github.io/cb-gsg/images/tag-statistics.png)
	List of tags is shown in the left column. The other columns show readings from each of the two antennas on each sensor.  The values are signal strength (green from strong, red for weak) and brightness or dimness for time since the last reading. The most recent and strongest reading is highlighted in gray as the most likely current location for each tag.
4. Click any data point or the circle-I button in the upper right of the dashboard to see what the readings and colors mean.

	![tag data](https://baychub.github.io/cb-gsg/images/tag-statistics-legend.png)
6. Experiment with moving RFID tags around the room to see how the sensor readings respond.

### See digested inventory data
Each RSP sensor reads each RFID tag that it can reach many times per second, generating a huge volume of raw data. The RSP Controller application processes that raw data stream into meaningful events and status. 

For example, thousands of consecutive readings of the same tag by the same sensor can be compressed into a state of PRESENT with that tag on that sensor. Or a tag that has no current reading but whose last reading was at the exit sensor would have a status of EXIT. The Inventory window shows these computed status values for each known tag.
1. Click the three-bar menu button to the left of the Tag Statistics page heading to open the navigation menu.
2. Click Inventory.

	![Inventory page](https://baychub.github.io/cb-gsg/images/inventory.png)

### Turn scheduling on or off
By default, the RSP Controller application directs each RSP sensor to read one at a time, from first sensor to last sensor and then back to the first again. This minimizes interference in case the antennas are relatively close together with significant overlaps.

You can change to have all sensors reading simultaneously, for example, if the antennas are farther apart. 
1. Click the three-bar menu button at the top-left of the web admin portal.
2. Click Scheduler.

	![](https://baychub.github.io/cb-gsg/images/scheduler.png)
3. In the blue header, click the ALL_ON option.
4. Observe the change in the LED pattern on the sensors.

**Troubleshooting:** If you click the FROM_CONFIG option but haven't set up a configuration, the sensor will not read any data. Click the ALL_ON or ALL_SEQUENCED option to read data again.

## Viewing RFID Data with MQTT
You've now used the RSP web portal to see a sample of some of the platform's capabilities. In this section, you'll use some RSP building blocks of data that can be part of your own RFID solution:
 - **MQTT communications protocol.** The RSP Controller uses a built-in MQTT broker to deliver data structured as JSON-RPC. Many libraries support the well-known MQTT and JSON-RPC protocols, so it's straightforward to implement data access in the code for your solution.

### MQTT overview
You can open a terminal window and subscribe to the RSP Controller events topic in order to monitor tag events produced by the RSP Controller, raw RFID tag data, and other RSP system information. 

The RSP web portal displays many of the available MQTT topics (specific sets of data you can subscribe to) in the Upstream MQTT and Downstream MQTT panes of the Dashboard page, e.g., "rfid/controller/alerts". The RSP Controller divides its MQTT topics into two groups:
- Downstream: data the RSP Controller pulls from or sends to the sensors
- Upstream: processed data the RSP Controller makes available out on the network to go into a database, management app, etc.

### Subscribe to an MQTT data topic
To monitor RSP data over MQTT from a terminal window:
1. Run the mosquitto_sub command and specify the MQTT topic for the data you want (in this example, controller events).
```
#-- See data from the RSP Controller events topic
mosquitto_sub -t x`
```

The console displays in JSON-RPC format any high-level events processed by the RSP Controller (e.g., an RFID tag has appeared for the first time) while you're monitoring this topic.

2. If you don't see any events initially, try introducing or moving some tags to generate an event.
	![Console login](https://baychub.github.io/cb-gsg/images/mosquitto_sub_events.jpg)
	
4. Press Ctrl-C to stop monitoring MQTT data on this topic.
5. Try another MQTT topic to see other types of data the RSP Controller publishes:
```
#-- See raw data from all tag reads
mosquitto_sub -t rfid/rsp/data/#
```
Full documentation of the MQTT topics and data definitions can be found in the *[Intel® RSP Controller - Edge Computer Software Application Programming Interface (API)](https://github.com/intel/rsp-sw-toolkit-gw/blob/master/docs/Intel-RSP-Controller-App_API_Guide.pdf)* guide, chapters 2 and 3.

## Configuring RSP Controller for Your Use Case
Between MQTT topics and the demonstration web admin portal, there are several ways to configure your sensors and RSP implementation for scale deployments. This section will show one example, and there are others in the Next Steps section.

### Assign a personality to a sensor
Sensor can have an optional *personality*, or category that tells RSP Controller application to process RFID data from that sensor in special ways that vastly simplify your code for processing the raw RFID data. The RSP Controller supports these personalities:
* The EXIT personality is for a sensor near the exit of a facility and generates a DEPARTED event when this sensor was the last to detect a tag and a certain amount of time has elapsed.
* The POS (point of sale) personality is intended for a retail checkout counter. A sensor with this personality generates a DEPARTED event immediately upon detection and allows re-entry into inventory after a certain amount of time.

You can learn how to assign a personality to a sensor by carrying out one of the retail or QSR tutorials under Next Steps, below. You'll have options for whichever Intel&reg; RSP hardware you have, either the H1000 or H3000 sensor models.

## Next Steps
The links below contain practical information for getting ready to do implement Intel&reg; RSP sensors and software.
- [Retail tutorial](./docs/usecases/use_cases.md): Implementation guidance for batch configuration of sensors on retail floors
- [QSR (quick-service restaurant) tutorial](./docs/usecases/use_cases.md): Implementation guidance for batch configuration of sensors for QSR environments
- [RSP Controller User Guide](https://github.com/intel/rsp-sw-toolkit-gw/blob/master/docs/Intel-RSP-Controller-App_Installation_User_Guide.pdf): Reference for the RSP Controller application that the system is built on
- [RSP Controller API](https://github.com/intel/rsp-sw-toolkit-gw/blob/master/docs/Intel-RSP-Controller-App_API_Guide.pdf): Reference for getting data from and configuration commands to the RSP Controller software
- [Other RSP documentation](https://01.org/rsp-sw-toolkit/downloads/installation-user-guides): Guides for RSP hardware and an Android-based NFC application
- [Updating sensor software](https://github.com/baychub/cb-gsg/blob/master/update_sensor.md): Instructions for installing updates to RSP sensor software, also found in "Sensor Software Update," chapter 11 of the RSP Controller User Guide.

<!--stackedit_data:
eyJoaXN0b3J5IjpbLTEwOTc2NjA2NDgsNzA0NDM1NjMzLDEzNT
c3NDcxNjgsLTc1MjQ3ODAyMSwtNTU1NjU1MTAxLDQ2NzUwMjc4
NiwtMjA1OTI2NzI1NSwtODYyNzk2NDgwLC0xMzQ2NzU0MTg0LD
YzMzEzMTYsMTMwMzQyNDY1MSwtMTc2ODYwMzAwMSwtMzc3NDEx
NzMxLDkyOTgyOTA2MSw4MjY3MzkwNCwtMTY0ODYwMTEwNSwxNz
QyOTQzODMwLDE0MzA2MDIzMzIsLTEwNzIwMjM0NjAsLTE4NjI4
NzMxODFdfQ==
-->