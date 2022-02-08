onecontrol2mqtt - bring OneControl events into MQTT
===============

onecontrol2mqtt

### About

This is a Node Red flow that will connect to your Lippert OneControl cloud connector (you need the device in place, but it does not need to be connected to the internet and you do not have to pay for their service).

To check if you are able to use this flow, when on the RV's wireless network, open a browser to http://192.168.1.4:8080.  If you are able to connect to this device, then you are good to go.

To install, copy the contents of "flows.json" into the "Import" section of Node Red.  When the flow is deployed, it will attempt to do its thing.

**Important** if you find connections are failing, check the contents of the "Set Variables" node to make sure that the target is correct, and also ensure your MQTT node configuration (host and credentials) is correct.

Requires the following nodes:
- node-red-contrib-moment
- node-red-contrib-sse-client
- node-red-node-rbe

### Some technical details on how this works:

First, many of the options are configurable - check out the "System Variables" node when it is loaded into Node Red.
The base mqtt topic is configurable, so you can publish and subscribe pretty much anywhere.

Via the SSE node, I connect to the OneControl bridge hia API and pull the event stream into Node Red for processing.

The underlying technology in the OneControl bridge is OpenHab, an open-source automation platform.  It is currently running version 2.3.0.201807201454 as of the time of this writing (2/8/2022).

The incoming payloads look like this:
```
{
  "topic": "smarthome/items/idsmyrv_hvac_thing_0000000DA7271002_inside_temperature/state",
  "payload": {
    "type": "Decimal",
    "value": "54.75"
  },
  "type": "ItemStateEvent",
  "_msgid": "6186b7643da1846c",
}
```
This is the naming convention I have adopted:
```
                         |---------------------channel---------------|
                         |----------------device ------------|       |-attr-|
   topic: smarthome/items/idsmyrv_light_thing_0000000DA6BB1407_switch/state
          |- base topic -|                                    |entity|
                                              |----deviceID--|
```
Based on that naming convention, I break the info out like so:

```
topicOrig =      smarthome/items/idsmyrv_light_thing_0000000DA6BB1407_switch/state
baseTopic =      smarthome/items
channelName =    idsmyrv_light_thing_0000000DA6BB1407_switch
deviceName =     light_0000000DA6BB1407
deviceID =       0000000DA6BB1407
entityType =     switch
attribute =      state
```

Since I get the feed from OneControl(OpenHab), I have to convert to to something more generic
```
new topic: (example) integration/onecontrol/device/switch_0000000DA7251E09/percent/
```
There are 4 different kinds of messages: 
- state
  - original value: ItemStateEvent
- stateChange
  - original value: ItemStateChangedEvent
- deviceState
  - original value: ThingStatusInfoEvent
- command
  - original value: ItemCommandEvent

I split those up into topics off above:
```
light_0000000DA7251E09/switch/state
light_0000000DA7251E09/switch/stateChange
light_0000000DA7251E09/switch/deviceState
light_0000000DA7251E09/switch/command
```
(deviceState is the host device.  Think 3 relays on a board, if the board (device) goes down, the relays go offline)

Since the payloads of the different event types are different, I standardize them:
```
type: "onoff"               (decimal, string, onoff, percent, etc)
value: "off"
typeOrig: "OnOff"           (if during the standardization of the data it is changed, the old value is here)
valueOrig: "OFF"            (if during the standardization of the data it is changed, the old value is here)
details: ""                 (some values have additional details)
label: "Water Pump Relay Current Draw"  (this is the human readable name of the entity)
timeStamp: "Mon Feb 07, 2022 4:24:23pm"
```

Now that the inbound info is saved/published, an additional topic is created:
```
light_0000000DA7251E09/switch/attributes
```
While this is currently in progress, this will be used to hold information that is needed for other platforms, as well as info that is useful, but not necessarily worth a dedicated entity.  This is the info I will use for HA discovery and are retained.  When the flows are deployed, they pick up the attributes compare that info to the discovery host list.  If there is a device that is new, it adds it to the discovery topic to generate a new HA entity, if the device is not found, it removes it from discovery so the entity is deleted.
