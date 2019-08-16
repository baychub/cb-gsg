![QSR H1000 Use Case](../usecases/QSR_H1000.png)

This use case demonstrates configuring the Intel&reg; RSP H1000 Devkit Sensor and Intel&reg; RSP 
Controller Application as deployed in a typical quick serve restaurant (qsr) environment.

## Goals  
- Manage a deployment with two separate cold storage rooms, using one H1000 sensor and two antennae
- Know when tagged items come into either cold room
- Determine the location of a tagged item (sensor and facility)
- Know when items potentially move between the cold rooms
- Know when items leave either of the cold rooms
  
By the end of the example, you will be able to track tags as they move in and out of the different 
cold rooms.
  
## Prerequsites  
1. It is assumed that the controller is already running and the sensor has its antennae attached, is 
running, and is connected to the controller.

2. In [DevkitQsrCluster.json](./DevkitQsrCluster.json), edit the sensor device id in the 
sensor_groups to match the sensor included with the Devkit. 
This cluster configuration file is an example that establishes the one facility (ColdArea).  It also 
creates aliases for the two antennae (Freezer and Refridgerator) in order to generate more natural 
locations names, as opposed to RSP-150004-0 and RSP-150004-1.  The sensor is configured with an EXIT 
personality in order to detect when an item leaves either of the cold rooms.  It also will assign 
the appropriate behaviors for reading the RFID tags.

3. Hide the Tags  
Make sure no tags are visible to the antennae attached to the sensor in order to see a complete use case 
scenario.

## Configure / Control the Intel&reg; RSP Controller Application
After the prerequisites have been met, choose one of the following methods to configure and control the 
application. Each method accomplishes the same configuration tasks.
- Using the Web Admin
- Using the Command Line Interface (CLI)
- Using the MQTT Messaging API

Note: In the following instructions, the term YOUR_PROJECT_DIRECTORY will refer to the directory where the 
cloned rsp-sw-toolkit-gw repo contents reside, and the term YOUR_DEPLOY_DIRECTORY will refer to the directory 
where the Intel&reg; RSP Controller Application was deployed.

___

### USING THE WEB ADMIN
1. Open the [web admin](http://localhost:8080/web-admin) page and confirm that the sensor included in the 
dev kit is connected. This can be seen on the [dashboard](http://localhost:8080/web-admin/dashboard.html) 
page or the [sensors](http://localhost:8080/web-admin/sensors-main.html) page.

2. On the [scheduler](http://localhost:8080/web-admin/scheduler.html) page, stop the sensor from reading 
by selecting the INACTIVE run state.

3. On the [inventory](http://localhost:8080/web-admin/inventory-main.html) page, press the Unload button 
to clear out all previous tag history to start a clean session.

4. On the [behaviors](http://localhost:8080/web-admin/behaviors.html) page, use the Upload From File
button to upload the use case behavior to the controller. The behavior file can be found at 
YOUR_PROJECT_DIRECTORY/rsp-sw-toolkit-gw/examples/use-cases/qsr/h1000/DevkitQsrBehaviorExit_PORTS_2.json.  

    This __MUST__ be loaded to the controller __BEFORE__ the cluster configuration because the cluster file 
    references that behavior id, and that behavior must already be known by the controller. Otherwise the 
    loading of the cluster configuration file will fail validation.

5. Upload the __edited__ cluster configuration file (see Prerequistes) using the 
[cluster config](http://localhost:8080/web-admin/cluster-config.html) page.

6. On the [scheduler](http://localhost:8080/web-admin/scheduler.html) page, start the sensor reading 
according to the cluster configuration by selecting the FROM_CONFIG run state. The clusters that the 
scheduler is using will be displayed on the page.

7. On the [sensors](http://localhost:8080/web-admin/sensors-main.html) page, confirm that the sensor has 
been configured as expected and is reading tags according to the cluster configuration file.

8. Navigate to the [inventory](http://localhost:8080/web-admin/inventory-main.html) page which can be used 
to monitor tag reads and states.

Continue to the Observe Tag Events section.
___
  
### USING THE CLI
1. Open a terminal window and copy the use case behavior to the deployed controller so it is available for 
use.
    ```bash
    cd YOUR_PROJECT_DIRECTORY/rsp-sw-toolkit-gw/examples/use-cases/qsr/h1000
    cp DevkitQsrBehaviorExit_PORTS_2.json YOUR_DEPLOY_DIRECTORY/rsp-sw-toolkit-gw/config/behaviors/
    ```

2. Connect to the controller's command line interface and execute the following series of commands.
    ```bash
    ssh -p5222 console@localhost
    password: console
        
    #-- stop the scheduler
    cli> scheduler set.run.state INACTIVE 
    ------------------------------------------
    completed
    ------------------------------------------
    
    #-- unload the current inventory
    cli> inventory unload 
    ------------------------------------------
    unload complete
    ------------------------------------------
    
    #-- load the cluster configuration
    cli> clusters load.file YOUR_PROJECT_DIRECTORY/rsp-sw-toolkit-gw/examples/use-cases/qsr/h1000/DevkitQsrCluster.json
    ------------------------------------------
    completed
    ------------------------------------------
    
    #-- activate the scheduler in custom configuration mode
    cli> scheduler set.run.state FROM_CONFIG 
    ------------------------------------------
    completed
    ------------------------------------------

    #-- confirm the configuration is active 
    cli> scheduler show 
    ------------------------------------------
    runState: FROM_CONFIG
    ------------------------------------------
    clusters:
          id: ColdAreaCluster
    behavior: DevkitQsrBehaviorExit_PORTS_2
    sensors: [RSP-150004 ]
    
    ------------------------------------------
    ```

Continue to the Observe Tag Events section.
___

### USING MQTT:
1. Edit [cluster_set_config_request_use_case_qsr.json](./cluster_set_config_request_use_case_qsr.json) 
replacing "CONTENTS_OF_CLUSTER_CONFIG_GO_HERE" with the contents of the edited DevkitQsrCluster.json file. 

2. Open a terminal window and subscribe to the controller's command response topic in order to monitor the 
command responses.
    ```bash
    #-- monitor the rpc command responses
    mosquitto_sub -t rfid/controller/response
    ```

3. Open another terminal to send JsonRPC commands over MQTT to configure and control the controller.
    ```bash
    #-- change directory to the examples folder 
    #-- so the example commands work correctly
    cd YOUR_PROJECT_DIRECTORY/rsp-sw-toolkit-gw/examples
    
    #-- stop the scheduler
    mosquitto_pub -t rfid/controller/command -f api/upstream/scheduler_set_run_state_request_INACTIVE.json
    
    #-- unload the current inventory
    mosquitto_pub -t rfid/controller/command -f api/upstream/inventory_unload_request.json
    
    #-- load the behavior specific to this exercise
    #-- (lowered power level as antennae are likely interferring)
    mosquitto_pub -t rfid/controller/command -f use-cases/qsr/h1000/behavior_put_request_Exit.json
    
    #-- load (set) the cluster configuration
    mosquitto_pub -t rfid/controller/command -f use-cases/qsr/h1000/cluster_set_config_request_use_case_qsr.json
    
    #-- activate the scheduler in custom configuration mode
    mosquitto_pub -t rfid/controller/command -f api/upstream/scheduler_set_run_state_request_FROM_CONFIG.json
    ```

Continue to the Observe Tag Events section.
___

## Observe Tag Events
Check that the antennae are not pointed in conflicting directions; keep the antennae separate and pointing 
away from each other as much as possible.

Open a terminal window and subscribe to the controller events topic in order to monitor tag events as 
produced by the controller.

```bash
#-- monitor the upstream events topic
mosquitto_sub -t rfid/controller/events
```

1. ##### Tag arrival in the first cold room
    At this point, remove a tag from hiding and place it nearby one of the two antennae. When the tag is  
    read initially, an arrival event will be generated on the rfid/controller/events topic. 
    Verify from the Web Admin 
    [inventory](http://localhost:8080/web-admin/inventory-main.html) page that the tags are now in the 
    EXITING state and the location is at the first antenna's alias (either Freezer or Refridgerator).  
    Verify the receipt of the MQTT event message.
    ```json
    {
      "jsonrpc": "2.0",
      "method": "inventory_event",
      "params": {
        "sent_on": 1559867406651,
        "device_id": "intel-acetest",
        "data": [
          {
            "facility_id": "ColdArea",
            "epc_code": "303530C29C000000F0006B12",
            "tid": null,
            "epc_encode_format": "tbd",
            "event_type": "arrival",
            "timestamp": 1559867406524,
            "location": "Freezer"
          }
        ]
      }
    }
    ```

2. ##### Tag moved to the other cold room
    Now take the tag and move it to the other antenna. Since these antennae are in the same facility, a 
    moved event will be generated. It may take a few moments for the event to be generated as the 
    algorithm uses time-weighted RSSI averages to determine tag location. From the 
    [inventory](http://localhost:8080/web-admin/inventory-main.html) page, confirm that the tag's location 
    has changed to the other cold room.  
    Verify the receipt of the MQTT event message.
    ```json
    {
      "jsonrpc": "2.0",
      "method": "inventory_event",
      "params": {
        "sent_on": 1559867488229,
        "device_id": "intel-acetest",
        "data": [
          {
            "facility_id": "ColdArea",
            "epc_code": "303530C29C000000F0006B12",
            "tid": null,
            "epc_encode_format": "tbd",
            "event_type": "moved",
            "timestamp": 1559867487834,
            "location": "Refridgerator"
          }
        ]
      }
    }
    ```

3. ##### Tag departure from the second cold room
    Now take that tag and hide it such that it can't be seen by either antenna.  After about 
    30 seconds, a departed event should be generated for the tag that was removed. From the 
    [inventory](http://localhost:8080/web-admin/inventory-main.html) page, confirm that the tag 
    state of the removed tag has changed to DEPARTED_EXIT.  
    Verify the receipt of the MQTT event message.
    ```json  
    {
      "jsonrpc": "2.0",
      "method": "inventory_event",
      "params": {
        "sent_on": 1559867527713,
        "device_id": "intel-acetest",
        "data": [
          {
            "facility_id": "ColdArea",
            "epc_code": "303530C29C000000F0006B12",
            "tid": null,
            "epc_encode_format": "tbd",
            "event_type": "departed",
            "timestamp": 1559867494569,
            "location": "Refridgerator"
          }
        ]
      }
    }
    ```
