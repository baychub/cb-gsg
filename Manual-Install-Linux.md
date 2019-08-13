## Installing Manually on Linux
The automatic installation script for Linux downloads, builds, and starts the the RSP Controller Application software. It also launches the web admin portal. 

You'll need know the specific commands for those tasks when you do the following:

 - Control details of the installation
 - Reboot the edge computer or close the terminal with the RSP Controller Application process
 - Close the browser running the web admin portal

This document gives the commands for manually downloading, building, and starting the software to use Intel RSP.

### Contents
- Download dependencies and code
- Build and deploy
- Run RSP Controller Application
- Launch web admin portal reference software
- Use web admin portal, CLI, or MQTT to manage sensors

### Download dependencies and code
1. Run these commands to get dependencies and clone the RSP project from GitHub:
```
#-- Install development dependencies
sudo apt-get install default-jdk git gradle

#-- Install runtime dependencies
sudo apt-get install mosquitto mosquitto-clients avahi-daemon ssh

#-- Install and configure network time server on local machine so no internet connection required
sudo apt-get install ntp
echo "server 127.127.1.0 prefer" | sudo tee -a /etc/ntp.conf
echo "fudge 127.127.22.1" | sudo tee -a /etc/ntp.conf
sudo systemctl status ntp

#-- Create base directory for use case examples and documentation
mkdir -p ~/projects

#-- Clone the reference source code
cd ~/projects
git clone https://github.com/intel/rsp-sw-toolkit-gw.git
```

### Build and deploy
The commands below will build the RSP Controller Application software and then create credentials so RSP sensors can connect to the edge computer. Without these credentials, you can't communicate with the sensors.
1. Build the application software with these commands in a terminal window:
```
cd ~/projects/rsp-sw-toolkit-gw

#-- The gradle build script will trigger a full build and
#-- a local deployment to the directory ~/deploy/rsp-sw-toolkit-gw 
gradle clean deploy
```
2. Generate credentials (needed just once per build) with these commands:
```
#-- !!!! CERTIFICATES ARE NEEDED !!!!
#-- RSP sensors connect securely to the RSP Controller using a self-signed certificate.
#-- A convenience script will generate these credentials in order to
#-- get the reference deployment up and running quickly.
mkdir -p ~/deploy/rsp-sw-toolkit-gw/cache
cd ~/deploy/rsp-sw-toolkit-gw/cache
~/deploy/rsp-sw-toolkit-gw/gen_keys.sh
```

### Run RSP Controller Application
The RSP Controller Application software provides the functionality for the RSP system and must be running. 
1. Run the following commands to start the application in the foreground:
```
cd ~/deploy/rsp-sw-toolkit-gw
~/deploy/rsp-sw-toolkit-gw/run.sh
```
2. Connect an Intel&reg; RSP sensor on the same network segment as the Intel&reg; RSP Controller Application. 

### Launch web admin portal reference software

The Intel&reg; RSP Controller Application provides a web-based administration interface for configuration and monitoring. Follow these steps to run the web admin portal.
1. Launch a web browser on the edge computer. 
2. Enter this URL in your browser:
	http://localhost:8080/web-admin/

### Use web admin portal, CLI, or MQTT to manage sensors
For details, go to the [Getting Started Guide](https://github.com/baychub/cb-gsg/blob/master/docs/Getting-Started.md).
<!--stackedit_data:
eyJoaXN0b3J5IjpbLTcxNDE2NzExOV19
-->
