Building ZigBee Device Handlers
===============================

There are four common ZigBee commands that you will use to integrate
SmartThings with your ZigBee Devices.

Read
----

Read gets the devices current state and is formatted like this:

.. code-block:: groovy

    def refresh() {
        "st rattr 0x${device.deviceNetworkId} 1 0xB04 0x50B"
    }

In this example, the device type (from the "CentraLite Switch" device
type) is calling the "refresh" function. It is sending a ZigBee Read
Attribute request to read the current state (the active power draw). The
cluster we are reading here is Electrical Measurement (0xB04) and
specifically the Active Power Attribute (0x50B).

+-------------------------------+-----------------------------+
| Component                     | Description                 |
+===============================+=============================+
|st rattr                       | SmartThings Read Attribute  |
+-------------------------------+-----------------------------+
|0x\$\{device.deviceNetworkId\}	| Device Network ID           |
+-------------------------------+-----------------------------+
|1                              | Endpoint Id                 |
+-------------------------------+-----------------------------+
|0xB04                          | Cluster                     |
+-------------------------------+-----------------------------+
|0x50B                          | Attribute                   |
+-------------------------------+-----------------------------+

Write
-----

Write sets an attribute of a ZigBee device and is formatted like this:

.. code-block:: groovy

    def configure() {
            "st wattr 0x${device.deviceNetworkId} 1 8 0x10 0x21 {1400}"
        }

In this example (from the "ZigBee Dimmer" device type) we are writing to
an attribute to set the amount of time it takes for a light to fully dim
on and off. Here we are using the Level Control Cluster (8) to write to
the attribute that defines on and off transition time (0x10). The value
we are using is formatted in an Unsigned 16-bit integer (0x21) with the
payload being in 1/10th of a second. In this case the payload ({1400})
translates to 2 seconds.

Each attribute possesses a specific data type. The corresponding value for
this data type can be found in table 2.16 of the
`ZigBee Cluster Library <http://www.zigbee.org/download/standards-zigbee-cluster-library/>`__.

.. note:: How does a payload of 1400 translate to 2 seconds? The value 1400 is actually a little endian hex string. It translates into an integer value of 0x0014, which is decimal value 20. Finally, 20 * 1/10 of a second equals 2 seconds.

+-------------------------------+-----------------------------+
| Component                     | Description                 |
+===============================+=============================+
|st wattr                       | SmartThings Write Attribute |
+-------------------------------+-----------------------------+
|0x${device.deviceNetworkId}    |Device Network ID            |
+-------------------------------+-----------------------------+
|1                              |Endpoint Id                  |
+-------------------------------+-----------------------------+
|8                              |Cluster                      |
+-------------------------------+-----------------------------+
|0x10                           |Attribute Set                |
+-------------------------------+-----------------------------+
|0x21                           |Data Type                    |
+-------------------------------+-----------------------------+
|{1400}                         |Payload                      |
+-------------------------------+-----------------------------+

Command
-------

Command invokes a command on a ZigBee device and is formatted like this:

.. code-block:: groovy

    def on() {
        "st cmd 0x${device.deviceNetworkId} 1 6 1 {}"
    }

In this example (from the "ZigBee Dimmer" device type) we are sending a
ZigBee Command to turn the device on. We use the On/Off Cluster (6) and
send the command to turn on (1). This commands has no payload, so there
is nothing within the payload brackets. Even though there is no payload,
the empty brackets are still required.

+-------------------------------+-----------------------------+
| Component                     | Description                 |
+===============================+=============================+
|st cmd                         |SmartThings Command          |
+-------------------------------+-----------------------------+
|0x${device.deviceNetworkId}    |Device Network ID            |
+-------------------------------+-----------------------------+
|1                              |Endpoint Id                  |
+-------------------------------+-----------------------------+
|6                              |Cluster                      |
+-------------------------------+-----------------------------+
|1                              |Command                      |
+-------------------------------+-----------------------------+
|{}                             |Payload                      |
+-------------------------------+-----------------------------+

Zdo Bind
--------

Bind instructs a device to notify us when an attribute changes and is
formatted like this:

.. code-block:: groovy

    def configure() {
        "zdo bind 0x${device.deviceNetworkId} 1 1 6 {${device.zigbeeId}} {}"
    }

In this example (using the "CentraLite Switch" device type), the bind
command is sent to the device using its Network ID which can be
determined using 0x${device.deviceNetworkId}. Then using source and
destination endpoints for the device and hub (1 1), we bind to the
On/Off Clusters (6) to get events from the device. The last part of the
message contains the hub's ZigBee id which is set as the location for
the device to send callback messages to. Note that not at all devices
support binding for events.

+-------------------------------+-----------------------------+
| Component                     | Description                 |
+===============================+=============================+
|zdo bind                       |SmartThings Command          |
+-------------------------------+-----------------------------+
|0x${device.deviceNetworkId}    |Device Network ID            |
+-------------------------------+-----------------------------+
|1                              |Source Endpoint              |
+-------------------------------+-----------------------------+
|1                              |Destination Endpoint         |
+-------------------------------+-----------------------------+
|0x0006                         |Cluster                      |
+-------------------------------+-----------------------------+
|{${device.zigbeeId}}{}         |ZigBee ID ("IEEE Id")        |
+-------------------------------+-----------------------------+

ZigBee Utilities
----------------

In order to work with ZigBee you will need to use the ZigBee Cluster
Library extensively to look up the proper values to send back and forth
to your device. You can download this document
`here <http://www.zigbee.org/download/standards-zigbee-cluster-library/>`__.
