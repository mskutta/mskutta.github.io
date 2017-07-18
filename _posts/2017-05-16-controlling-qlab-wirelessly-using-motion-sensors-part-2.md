---
layout: post
title: "Controlling QLab Wirelessly using Motion Sensors - Part 2"
date:   2017-05-16 00:00:00 -0500
categories: halloween
tags: qlab raspberry-pi osc pishield
author: Mike Skutta
---

* content
{:toc}

## Overview 
In the last blog post, [Controlling QLab Wirelessly using Motion Sensors](/2017/05/13/controlling-qlab-wirelessly-using-motion-sensors/), I talked about setting up the Raspberry Pi to send Motion data to QLab via OSC messages.
In this post, I will expand on that by building out support for multiple sensors as well as providing a dashboard to display the sensor values.




As part of our Halloween display we will be triggering multiple scenes using multiple motion sensors.
I just wanted to demonstrate how I setup Node-Red to handle multiple sensors.
I also wanted a way to display the status of the sensors on a dashboard.
Using the dashboard, I am able to walk through the scenes while viewing the motion sensor status.
This allows easier tuning of where to place and point the sensors.
I also discovered when sensors are not connected (in the case when you may be setting up a sensor), a bit of noise passes through the system, triggering cues unexpectedly.
I updated the scripts to prevent this.

This is a continuation of the previous post.
I will not go into as much detail.
Please reference the previous post if you need more detailed steps.

## Install Node-Red Dashboard

In order to use the dashboard, we need to install the Node-Red Dashboard.

### Step 1: Install Node-Red Dashboard

Install [Node-Red Dashboard](https://flows.nodered.org/node/node-red-dashboard).

```
cd ~/.node-red
npm install node-red-dashboard
```

### Step 2: Restart Node-Red
Restart Node-Red so the changes take effect.

```
node-red-stop
node-red-start&
```

## Configure Node-Red

Now, configure Node-Red to support multiple sensors and the Dashboard.

### Step 1: Multiple Sensor Flow

Expand the Node-Red flow based on the prior [post](/2017/05/13/controlling-qlab-wirelessly-using-motion-sensors/).

![Node-Red Flow](/images/controlling-qlab-wirelessly-using-motion-sensors-part-2/node-red-flow.png)


The **switch** nodes are same as the prior **motion switch** nodes.
I shortened the name so everything fits better in the diagram.
I did have to update the script to handle the noise when a sensor is disconnected.
The following is the updated script:

``` javascript
// Determine the state of the sensor. 1 = motion
var state = (msg.payload < 64) ? 0 : 1;

// Get the last state
var last = context.get('last')||0;

// Only process changes in state
if (state === last) {
    return [ null, null ];
}

// Store the state for next time
context.set('last',state);

// Based on state, return output
if (state === 1) {
    return [ msg, null ];
} else {
    return [ null, msg ];
}
```

The **CH#'s** are ![Gauge Node](/images/controlling-qlab-wirelessly-using-motion-sensors-part-2/gauge-node.png) nodes.
Follow the [documentation](https://infusionsystems.com/pishield/node-red-sensor-dashboard/) provided by I-CubeX to set this up. 

The ![Change Node](/images/controlling-qlab-wirelessly-using-motion-sensors-part-2/change-node.png) nodes should be updated with the associated OSC message for each sensor.

### Step 2: Dashboard

If you followed the [instructions](https://infusionsystems.com/pishield/node-red-sensor-dashboard/) provided by I-CubeX, your dashboard should resemble the following:

![Node-Red Flow](/images/controlling-qlab-wirelessly-using-motion-sensors-part-2/node-red-dashboard.png)

You can navigate to the dashboard using **http://[Raspberry Pi IP Address]:1880/ui/**

### Complete Node-Red Configuration

The complete Node-Red settings are as follows:

```javascript
[
    {
        "id": "8868aa19.83a248",
        "type": "tab",
        "label": "Flow 1"
    },
    {
        "id": "163476c3.8a9749",
        "type": "ui_base",
        "theme": {
            "name": "theme-light",
            "lightTheme": {
                "default": "#0094CE",
                "baseColor": "#0094CE",
                "baseFont": "Helvetica Neue",
                "edited": true,
                "reset": false
            },
            "darkTheme": {
                "default": "#097479",
                "baseColor": "#097479",
                "baseFont": "Helvetica Neue",
                "edited": false
            },
            "customTheme": {
                "name": "Untitled Theme 1",
                "default": "#4B7930",
                "baseColor": "#4B7930",
                "baseFont": "Helvetica Neue"
            },
            "themeState": {
                "base-color": {
                    "default": "#0094CE",
                    "value": "#0094CE",
                    "edited": false
                },
                "page-titlebar-backgroundColor": {
                    "value": "#0094CE",
                    "edited": false
                },
                "page-backgroundColor": {
                    "value": "#fafafa",
                    "edited": false
                },
                "page-sidebar-backgroundColor": {
                    "value": "#ffffff",
                    "edited": false
                },
                "group-textColor": {
                    "value": "#1bbfff",
                    "edited": false
                },
                "group-borderColor": {
                    "value": "#ffffff",
                    "edited": false
                },
                "group-backgroundColor": {
                    "value": "#ffffff",
                    "edited": false
                },
                "widget-textColor": {
                    "value": "#111111",
                    "edited": false
                },
                "widget-backgroundColor": {
                    "value": "#0094ce",
                    "edited": false
                },
                "widget-borderColor": {
                    "value": "#ffffff",
                    "edited": false
                }
            }
        },
        "site": {
            "name": "Node-RED Dashboard",
            "hideToolbar": "false",
            "allowSwipe": "false",
            "dateFormat": "DD/MM/YYYY",
            "sizes": {
                "sx": 48,
                "sy": 48,
                "gx": 6,
                "gy": 6,
                "cx": 6,
                "cy": 6,
                "px": 0,
                "py": 0
            }
        }
    },
    {
        "id": "a2227267.5ae3e",
        "type": "ui_tab",
        "z": "",
        "name": "Motion Sensors",
        "icon": "dashboard",
        "order": 1
    },
    {
        "id": "d3cd702.a4f079",
        "type": "ui_group",
        "z": "",
        "name": "Group 1",
        "tab": "a2227267.5ae3e",
        "order": 1,
        "disp": false,
        "width": "3"
    },
    {
        "id": "20378d8c.019922",
        "type": "ui_group",
        "z": "",
        "name": "Group 2",
        "tab": "a2227267.5ae3e",
        "order": 2,
        "disp": false,
        "width": "3"
    },
    {
        "id": "a2b77e56.ec95f",
        "type": "mcp3008",
        "z": "8868aa19.83a248",
        "name": "PiShield-CH0",
        "device": "/dev/spidev0.0",
        "mode": "0x80",
        "interval": "100",
        "x": 289.1000061035156,
        "y": 40.0000114440918,
        "wires": [
            [
                "19ecaf33.969551",
                "69c66069.e7674"
            ]
        ]
    },
    {
        "id": "125a1636.bf6a1a",
        "type": "inject",
        "z": "8868aa19.83a248",
        "name": "start",
        "topic": "",
        "payload": "start",
        "payloadType": "str",
        "repeat": "",
        "crontab": "",
        "once": false,
        "x": 79.10001373291016,
        "y": 262.8000087738037,
        "wires": [
            [
                "a2b77e56.ec95f",
                "f8b068f8.c0c498",
                "43a4e6da.bfb578",
                "55d477d5.210cf8",
                "2e63e25.2db651e",
                "1aed5bef.36d564",
                "7bcc233e.33a7ac",
                "18834ca0.2a82a3"
            ]
        ]
    },
    {
        "id": "7158a004.84109",
        "type": "inject",
        "z": "8868aa19.83a248",
        "name": "stop",
        "topic": "",
        "payload": "stop",
        "payloadType": "str",
        "repeat": "",
        "crontab": "",
        "once": false,
        "x": 80.10001373291016,
        "y": 378.8000030517578,
        "wires": [
            [
                "a2b77e56.ec95f",
                "f8b068f8.c0c498",
                "43a4e6da.bfb578",
                "55d477d5.210cf8",
                "2e63e25.2db651e",
                "1aed5bef.36d564",
                "7bcc233e.33a7ac",
                "18834ca0.2a82a3"
            ]
        ]
    },
    {
        "id": "8706555e.921028",
        "type": "osc",
        "z": "8868aa19.83a248",
        "name": "",
        "path": "",
        "metadata": false,
        "x": 811.1001205444336,
        "y": 313.00000953674316,
        "wires": [
            [
                "57b9f7c4.cd8a08",
                "c5d32123.e0512"
            ]
        ]
    },
    {
        "id": "57b9f7c4.cd8a08",
        "type": "udp out",
        "z": "8868aa19.83a248",
        "name": "QLab",
        "addr": "10.81.95.117",
        "iface": "",
        "port": "53000",
        "ipv": "udp4",
        "outport": "",
        "base64": false,
        "multicast": "false",
        "x": 959.1000289916992,
        "y": 272.8000431060791,
        "wires": []
    },
    {
        "id": "c5d32123.e0512",
        "type": "debug",
        "z": "8868aa19.83a248",
        "name": "",
        "active": true,
        "console": "false",
        "complete": "true",
        "x": 956.1000289916992,
        "y": 357.6000156402588,
        "wires": []
    },
    {
        "id": "150291d7.947bde",
        "type": "change",
        "z": "8868aa19.83a248",
        "name": "/cue/1/start",
        "rules": [
            {
                "t": "set",
                "p": "topic",
                "pt": "msg",
                "to": "/cue/1/start",
                "tot": "str"
            },
            {
                "t": "delete",
                "p": "payload",
                "pt": "msg"
            }
        ],
        "action": "",
        "property": "",
        "from": "",
        "to": "",
        "reg": false,
        "x": 633.1001167297363,
        "y": 43.000000953674316,
        "wires": [
            [
                "8706555e.921028"
            ]
        ]
    },
    {
        "id": "19ecaf33.969551",
        "type": "function",
        "z": "8868aa19.83a248",
        "name": "switch",
        "func": "// Determine the state of the sensor. 1 = motion\nvar state = (msg.payload < 64) ? 0 : 1;\n\n// Get the last state\nvar last = context.get('last')||0;\n\n// Only process changes in state\nif (state === last) {\n    return [ null, null ];\n}\n\n// Store the state for next time\ncontext.set('last',state);\n\n// Based on state, return output\nif (state === 1) {\n    return [ msg, null ];\n} else {\n    return [ null, msg ];\n}\n",
        "outputs": "2",
        "noerr": 0,
        "x": 465.10005950927734,
        "y": 41.199984550476074,
        "wires": [
            [
                "150291d7.947bde"
            ],
            []
        ]
    },
    {
        "id": "f8b068f8.c0c498",
        "type": "mcp3008",
        "z": "8868aa19.83a248",
        "name": "PiShield-CH1",
        "device": "/dev/spidev0.0",
        "mode": "0x90",
        "interval": "100",
        "x": 285.1000061035156,
        "y": 121,
        "wires": [
            [
                "a2cde090.da35d",
                "e0555eb1.b183a"
            ]
        ]
    },
    {
        "id": "43a4e6da.bfb578",
        "type": "mcp3008",
        "z": "8868aa19.83a248",
        "name": "PiShield-CH2",
        "device": "/dev/spidev0.0",
        "mode": "0xA0",
        "interval": "100",
        "x": 290.1000061035156,
        "y": 197.00000667572021,
        "wires": [
            [
                "1d539b1a.0c5a55",
                "6c73852e.2c763c"
            ]
        ]
    },
    {
        "id": "55d477d5.210cf8",
        "type": "mcp3008",
        "z": "8868aa19.83a248",
        "name": "PiShield-CH3",
        "device": "/dev/spidev0.0",
        "mode": "0xB0",
        "interval": "100",
        "x": 291.1000061035156,
        "y": 274.0000333786011,
        "wires": [
            [
                "79fece06.7a2b7",
                "ea5ce738.4915e8"
            ]
        ]
    },
    {
        "id": "2e63e25.2db651e",
        "type": "mcp3008",
        "z": "8868aa19.83a248",
        "name": "PiShield-CH4",
        "device": "/dev/spidev0.0",
        "mode": "0xC0",
        "interval": "100",
        "x": 289.1000556945801,
        "y": 350.0000247955322,
        "wires": [
            [
                "e48906ea.eb5fc8",
                "3ef27281.e82d7e"
            ]
        ]
    },
    {
        "id": "1aed5bef.36d564",
        "type": "mcp3008",
        "z": "8868aa19.83a248",
        "name": "PiShield-CH5",
        "device": "/dev/spidev0.0",
        "mode": "0xD0",
        "interval": "100",
        "x": 286.1000061035156,
        "y": 427.0000123977661,
        "wires": [
            [
                "202f6ced.d6c3c4",
                "5a1229ef.637c18"
            ]
        ]
    },
    {
        "id": "7bcc233e.33a7ac",
        "type": "mcp3008",
        "z": "8868aa19.83a248",
        "name": "PiShield-CH6",
        "device": "/dev/spidev0.0",
        "mode": "0xE0",
        "interval": "100",
        "x": 280.1000328063965,
        "y": 505.0000286102295,
        "wires": [
            [
                "e557c2a6.b866e",
                "a42b8494.958228"
            ]
        ]
    },
    {
        "id": "18834ca0.2a82a3",
        "type": "mcp3008",
        "z": "8868aa19.83a248",
        "name": "PiShield-CH7",
        "device": "/dev/spidev0.0",
        "mode": "0xF0",
        "interval": "100",
        "x": 284.1000556945801,
        "y": 581.0000019073486,
        "wires": [
            [
                "e0790032.8dff3",
                "b5d60eb7.e296e"
            ]
        ]
    },
    {
        "id": "a2cde090.da35d",
        "type": "function",
        "z": "8868aa19.83a248",
        "name": "switch",
        "func": "// Determine the state of the sensor. 1 = motion\nvar state = (msg.payload < 64) ? 0 : 1;\n\n// Get the last state\nvar last = context.get('last')||0;\n\n// Only process changes in state\nif (state === last) {\n    return [ null, null ];\n}\n\n// Store the state for next time\ncontext.set('last',state);\n\n// Based on state, return output\nif (state === 1) {\n    return [ msg, null ];\n} else {\n    return [ null, msg ];\n}\n",
        "outputs": "2",
        "noerr": 0,
        "x": 466.1000633239746,
        "y": 120.20000457763672,
        "wires": [
            [
                "96a54092.309c7"
            ],
            []
        ]
    },
    {
        "id": "1d539b1a.0c5a55",
        "type": "function",
        "z": "8868aa19.83a248",
        "name": "switch",
        "func": "// Determine the state of the sensor. 1 = motion\nvar state = (msg.payload < 64) ? 0 : 1;\n\n// Get the last state\nvar last = context.get('last')||0;\n\n// Only process changes in state\nif (state === last) {\n    return [ null, null ];\n}\n\n// Store the state for next time\ncontext.set('last',state);\n\n// Based on state, return output\nif (state === 1) {\n    return [ msg, null ];\n} else {\n    return [ null, msg ];\n}\n",
        "outputs": "2",
        "noerr": 0,
        "x": 468.10003662109375,
        "y": 197.20001220703125,
        "wires": [
            [
                "4ce4c0af.29f62"
            ],
            []
        ]
    },
    {
        "id": "79fece06.7a2b7",
        "type": "function",
        "z": "8868aa19.83a248",
        "name": "switch",
        "func": "// Determine the state of the sensor. 1 = motion\nvar state = (msg.payload < 64) ? 0 : 1;\n\n// Get the last state\nvar last = context.get('last')||0;\n\n// Only process changes in state\nif (state === last) {\n    return [ null, null ];\n}\n\n// Store the state for next time\ncontext.set('last',state);\n\n// Based on state, return output\nif (state === 1) {\n    return [ msg, null ];\n} else {\n    return [ null, msg ];\n}\n",
        "outputs": "2",
        "noerr": 0,
        "x": 467.10003662109375,
        "y": 273.2000370025635,
        "wires": [
            [
                "6800b762.bf2d58"
            ],
            []
        ]
    },
    {
        "id": "e48906ea.eb5fc8",
        "type": "function",
        "z": "8868aa19.83a248",
        "name": "switch",
        "func": "// Determine the state of the sensor. 1 = motion\nvar state = (msg.payload < 64) ? 0 : 1;\n\n// Get the last state\nvar last = context.get('last')||0;\n\n// Only process changes in state\nif (state === last) {\n    return [ null, null ];\n}\n\n// Store the state for next time\ncontext.set('last',state);\n\n// Based on state, return output\nif (state === 1) {\n    return [ msg, null ];\n} else {\n    return [ null, msg ];\n}\n",
        "outputs": "2",
        "noerr": 0,
        "x": 467.1000862121582,
        "y": 349.2000370025635,
        "wires": [
            [
                "a5beea8d.b0b1a8"
            ],
            []
        ]
    },
    {
        "id": "202f6ced.d6c3c4",
        "type": "function",
        "z": "8868aa19.83a248",
        "name": "switch",
        "func": "// Determine the state of the sensor. 1 = motion\nvar state = (msg.payload < 64) ? 0 : 1;\n\n// Get the last state\nvar last = context.get('last')||0;\n\n// Only process changes in state\nif (state === last) {\n    return [ null, null ];\n}\n\n// Store the state for next time\ncontext.set('last',state);\n\n// Based on state, return output\nif (state === 1) {\n    return [ msg, null ];\n} else {\n    return [ null, msg ];\n}\n",
        "outputs": "2",
        "noerr": 0,
        "x": 467.10003662109375,
        "y": 426.20001220703125,
        "wires": [
            [
                "f1a0355c.0baf58"
            ],
            []
        ]
    },
    {
        "id": "e557c2a6.b866e",
        "type": "function",
        "z": "8868aa19.83a248",
        "name": "switch",
        "func": "// Determine the state of the sensor. 1 = motion\nvar state = (msg.payload < 64) ? 0 : 1;\n\n// Get the last state\nvar last = context.get('last')||0;\n\n// Only process changes in state\nif (state === last) {\n    return [ null, null ];\n}\n\n// Store the state for next time\ncontext.set('last',state);\n\n// Based on state, return output\nif (state === 1) {\n    return [ msg, null ];\n} else {\n    return [ null, msg ];\n}\n",
        "outputs": "2",
        "noerr": 0,
        "x": 464.1000633239746,
        "y": 504.20004081726074,
        "wires": [
            [
                "ba8326dc.1e1868"
            ],
            []
        ]
    },
    {
        "id": "e0790032.8dff3",
        "type": "function",
        "z": "8868aa19.83a248",
        "name": "switch",
        "func": "// Determine the state of the sensor. 1 = motion\nvar state = (msg.payload < 64) ? 0 : 1;\n\n// Get the last state\nvar last = context.get('last')||0;\n\n// Only process changes in state\nif (state === last) {\n    return [ null, null ];\n}\n\n// Store the state for next time\ncontext.set('last',state);\n\n// Based on state, return output\nif (state === 1) {\n    return [ msg, null ];\n} else {\n    return [ null, msg ];\n}\n",
        "outputs": "2",
        "noerr": 0,
        "x": 466.1000862121582,
        "y": 580.2000141143799,
        "wires": [
            [
                "91472f13.11fd2"
            ],
            []
        ]
    },
    {
        "id": "96a54092.309c7",
        "type": "change",
        "z": "8868aa19.83a248",
        "name": "/cue/2/start",
        "rules": [
            {
                "t": "set",
                "p": "topic",
                "pt": "msg",
                "to": "/cue/2/start",
                "tot": "str"
            },
            {
                "t": "delete",
                "p": "payload",
                "pt": "msg"
            }
        ],
        "action": "",
        "property": "",
        "from": "",
        "to": "",
        "reg": false,
        "x": 635.1000671386719,
        "y": 119.00000381469727,
        "wires": [
            [
                "8706555e.921028"
            ]
        ]
    },
    {
        "id": "4ce4c0af.29f62",
        "type": "change",
        "z": "8868aa19.83a248",
        "name": "/cue/3/start",
        "rules": [
            {
                "t": "set",
                "p": "topic",
                "pt": "msg",
                "to": "/cue/3/start",
                "tot": "str"
            },
            {
                "t": "delete",
                "p": "payload",
                "pt": "msg"
            }
        ],
        "action": "",
        "property": "",
        "from": "",
        "to": "",
        "reg": false,
        "x": 633.1000671386719,
        "y": 195.0000057220459,
        "wires": [
            [
                "8706555e.921028"
            ]
        ]
    },
    {
        "id": "6800b762.bf2d58",
        "type": "change",
        "z": "8868aa19.83a248",
        "name": "/cue/4/start",
        "rules": [
            {
                "t": "set",
                "p": "topic",
                "pt": "msg",
                "to": "/cue/4/start",
                "tot": "str"
            },
            {
                "t": "delete",
                "p": "payload",
                "pt": "msg"
            }
        ],
        "action": "",
        "property": "",
        "from": "",
        "to": "",
        "reg": false,
        "x": 635.1000671386719,
        "y": 273.00000858306885,
        "wires": [
            [
                "8706555e.921028"
            ]
        ]
    },
    {
        "id": "a5beea8d.b0b1a8",
        "type": "change",
        "z": "8868aa19.83a248",
        "name": "/cue/5/start",
        "rules": [
            {
                "t": "set",
                "p": "topic",
                "pt": "msg",
                "to": "/cue/5/start",
                "tot": "str"
            },
            {
                "t": "delete",
                "p": "payload",
                "pt": "msg"
            }
        ],
        "action": "",
        "property": "",
        "from": "",
        "to": "",
        "reg": false,
        "x": 633.1000671386719,
        "y": 350.0000104904175,
        "wires": [
            [
                "8706555e.921028"
            ]
        ]
    },
    {
        "id": "f1a0355c.0baf58",
        "type": "change",
        "z": "8868aa19.83a248",
        "name": "/cue/6/start",
        "rules": [
            {
                "t": "set",
                "p": "topic",
                "pt": "msg",
                "to": "/cue/6/start",
                "tot": "str"
            },
            {
                "t": "delete",
                "p": "payload",
                "pt": "msg"
            }
        ],
        "action": "",
        "property": "",
        "from": "",
        "to": "",
        "reg": false,
        "x": 632.1000671386719,
        "y": 424.0000123977661,
        "wires": [
            [
                "8706555e.921028"
            ]
        ]
    },
    {
        "id": "ba8326dc.1e1868",
        "type": "change",
        "z": "8868aa19.83a248",
        "name": "/cue/7/start",
        "rules": [
            {
                "t": "set",
                "p": "topic",
                "pt": "msg",
                "to": "/cue/7/start",
                "tot": "str"
            },
            {
                "t": "delete",
                "p": "payload",
                "pt": "msg"
            }
        ],
        "action": "",
        "property": "",
        "from": "",
        "to": "",
        "reg": false,
        "x": 632.1000671386719,
        "y": 503.00001525878906,
        "wires": [
            [
                "8706555e.921028"
            ]
        ]
    },
    {
        "id": "91472f13.11fd2",
        "type": "change",
        "z": "8868aa19.83a248",
        "name": "/cue/8/start",
        "rules": [
            {
                "t": "set",
                "p": "topic",
                "pt": "msg",
                "to": "/cue/8/start",
                "tot": "str"
            },
            {
                "t": "delete",
                "p": "payload",
                "pt": "msg"
            }
        ],
        "action": "",
        "property": "",
        "from": "",
        "to": "",
        "reg": false,
        "x": 628.1000671386719,
        "y": 580.0000171661377,
        "wires": [
            [
                "8706555e.921028"
            ]
        ]
    },
    {
        "id": "b5d60eb7.e296e",
        "type": "ui_gauge",
        "z": "8868aa19.83a248",
        "name": "",
        "group": "20378d8c.019922",
        "order": 4,
        "width": 0,
        "height": 0,
        "gtype": "gage",
        "title": "CH7",
        "label": "units",
        "format": "{{value}}",
        "min": 0,
        "max": "1023",
        "colors": [
            "#00b500",
            "#e6e600",
            "#ca3838"
        ],
        "seg1": "",
        "seg2": "",
        "x": 466.10011291503906,
        "y": 614.8000679016113,
        "wires": []
    },
    {
        "id": "a42b8494.958228",
        "type": "ui_gauge",
        "z": "8868aa19.83a248",
        "name": "",
        "group": "20378d8c.019922",
        "order": 3,
        "width": 0,
        "height": 0,
        "gtype": "gage",
        "title": "CH6",
        "label": "units",
        "format": "{{value}}",
        "min": 0,
        "max": "1023",
        "colors": [
            "#00b500",
            "#e6e600",
            "#ca3838"
        ],
        "seg1": "",
        "seg2": "",
        "x": 466.1000633239746,
        "y": 538.8000164031982,
        "wires": []
    },
    {
        "id": "5a1229ef.637c18",
        "type": "ui_gauge",
        "z": "8868aa19.83a248",
        "name": "",
        "group": "20378d8c.019922",
        "order": 2,
        "width": 0,
        "height": 0,
        "gtype": "gage",
        "title": "CH5",
        "label": "units",
        "format": "{{value}}",
        "min": 0,
        "max": "1023",
        "colors": [
            "#00b500",
            "#e6e600",
            "#ca3838"
        ],
        "seg1": "",
        "seg2": "",
        "x": 467.1000633239746,
        "y": 462.8000144958496,
        "wires": []
    },
    {
        "id": "3ef27281.e82d7e",
        "type": "ui_gauge",
        "z": "8868aa19.83a248",
        "name": "",
        "group": "20378d8c.019922",
        "order": 1,
        "width": 0,
        "height": 0,
        "gtype": "gage",
        "title": "CH4",
        "label": "units",
        "format": "{{value}}",
        "min": 0,
        "max": "1023",
        "colors": [
            "#00b500",
            "#e6e600",
            "#ca3838"
        ],
        "seg1": "",
        "seg2": "",
        "x": 469.10011291503906,
        "y": 384.8000602722168,
        "wires": []
    },
    {
        "id": "ea5ce738.4915e8",
        "type": "ui_gauge",
        "z": "8868aa19.83a248",
        "name": "",
        "group": "d3cd702.a4f079",
        "order": 4,
        "width": 0,
        "height": 0,
        "gtype": "gage",
        "title": "CH3",
        "label": "units",
        "format": "{{value}}",
        "min": 0,
        "max": "1023",
        "colors": [
            "#00b500",
            "#e6e600",
            "#ca3838"
        ],
        "seg1": "",
        "seg2": "",
        "x": 469.1000633239746,
        "y": 308.80005836486816,
        "wires": []
    },
    {
        "id": "6c73852e.2c763c",
        "type": "ui_gauge",
        "z": "8868aa19.83a248",
        "name": "",
        "group": "d3cd702.a4f079",
        "order": 3,
        "width": 0,
        "height": 0,
        "gtype": "gage",
        "title": "CH2",
        "label": "units",
        "format": "{{value}}",
        "min": 0,
        "max": "1023",
        "colors": [
            "#00b500",
            "#e6e600",
            "#ca3838"
        ],
        "seg1": "",
        "seg2": "",
        "x": 469.1000633239746,
        "y": 232.8000316619873,
        "wires": []
    },
    {
        "id": "e0555eb1.b183a",
        "type": "ui_gauge",
        "z": "8868aa19.83a248",
        "name": "",
        "group": "d3cd702.a4f079",
        "order": 2,
        "width": 0,
        "height": 0,
        "gtype": "gage",
        "title": "CH1",
        "label": "units",
        "format": "{{value}}",
        "min": 0,
        "max": "1023",
        "colors": [
            "#00b500",
            "#e6e600",
            "#ca3838"
        ],
        "seg1": "",
        "seg2": "",
        "x": 468.1000633239746,
        "y": 156.80000495910645,
        "wires": []
    },
    {
        "id": "69c66069.e7674",
        "type": "ui_gauge",
        "z": "8868aa19.83a248",
        "name": "",
        "group": "d3cd702.a4f079",
        "order": 1,
        "width": 0,
        "height": 0,
        "gtype": "gage",
        "title": "CH0",
        "label": "units",
        "format": "{{value}}",
        "min": 0,
        "max": "1023",
        "colors": [
            "#00b500",
            "#e6e600",
            "#ca3838"
        ],
        "seg1": "",
        "seg2": "",
        "x": 468.1000633239746,
        "y": 78.40002059936523,
        "wires": []
    }
]

```

## Running

1. Make sure the sensor(s) are plugged in and the Raspberry Pi is powered on.
1. Navigate to **http://[RASPBERRY PI IP ADDRESS]:1880/**
1. Click the **Start** (inject node)
1. The Raspberry Pi will now send OSC commands to QLab.
1. View the dashboard at: **http://[Raspberry Pi IP Address]:1880/ui/**
