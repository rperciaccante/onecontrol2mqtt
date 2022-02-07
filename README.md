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