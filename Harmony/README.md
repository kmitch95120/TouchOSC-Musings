# Infrared Remote UI for Logitech Harmony Hub

This is a proof-of-concept UI for a [Logitech Harmony Hub](https://support.myharmony.com/en-us/hub). Even though Logitech has officially discontinued the sale of this product, it remains one of the most popular universal remote control solutions, particularly well suited for integration with home automation systems. For security reasons, the XMPP remote feature on the hub is disabled by default and therefore must be enabled for remote control to work. This is be done from the Harmony app. Go to: **Menu > Harmony Setup > Add/Edit Devices & Activities > Remote & Hub > Enable XMPP**.

The TouchOSC template sends custom [OSC, Open Sound Control](https://ccrma.stanford.edu/groups/osc/index.html) messages to a message handler running in [Node-RED](https://nodered.org/) on a [Raspberry Pi 3B](https://www.raspberrypi.org/computers). The handler then sends command requests to the hub. The command requests are sent using the [Harmony Command node](https://flows.nodered.org/node/node-red-contrib-harmony-extra).

## The TouchOSC Template ([ir_remote.tosc](ir_remote.tosc))

![image](ir_remote.png)

The template is meant to be a simple layout that is easy to use, particularly in low-light situations.

Note that the entire remote is a group. This is because it is a sub-project for a much larger 4K wall-mounted control panel. I also make heavy use of element grouping to keep things aligned and easier to move around without having to be too concerned with messing up alignment.

Each button is a group made up of three objects; a background, a button, and a label. Each button has a single OSC message that is sent when the button is pressed. There is very little scripting in this template with the only scripts being on the buttons for visual confirmation of a press. If the device supports it, this would be a good place to put a call to the vibrate function for a more tactile confirmation.

The format of the custom OSC messages is as follows:

```lua
/<instance>/<location>/<device>/<command>
```

For this template the address is:

```lua
/hs4b/backoffice/tivo/<command>
```

## The Node-RED Message Handler Flow ([ir_remote_osc_handler.json](ir_remote_osc_handler.json))

![image](flow.png)

The message handler is straight-forward. A UDP listener node listens for TouchOSC OSC messages on port 8000. Of course, this port can be changed simply by editing the node.

The UDP mesage is sent to an [OSC decode node](https://flows.nodered.org/node/node-red-contrib-osc) where it is decoded into a Node-RED message containing topic and payload. The message is then passed to a switch node that acts as a filter. A regex match of the start of the OSC address is done and the message is passed along. Messages not starting with the proper address are simply discarded, after being logged, of course.

A message with a validated starting address is passed to another switch node that looks for a regex match at the end of the OSC address, which is the actual command.  Each of those matches directs the message to the corresponding Harmony Command node, which triggers a Hub command request. Any command not matched by the switch node is logged and discarded.
