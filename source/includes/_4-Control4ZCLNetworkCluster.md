
# Control4 Zigbee Cluster for End Devices

## Scope and Purpose

This content specifies the Control4 Networking Cluster, which is an application level interface between ZigBee devices and a Control4 System Controller. In general, use of clusters from the ZigBee Cluster Library promotes interoperability between ZigBee devices.  Use of this cluster promotes interoperability between devices and a Control4 System Controller by defining a standard set of attributes and commands that Control4 has found useful for a typical Control4 ZigBee network topology
This document should be used in conjunction with the ZigBee Cluster Library.


## References

The following standards and specifications contain provisions, which through reference in this document constitute provisions of this specification. All the standards and specifications listed are normative references.  At the time of publication, the editions indicated were valid. All standards and specifications are subject to revision, and parties to agreements based on this specification are encouraged to investigate the possibility of applying the most recent editions of the standards and specifications indicated below.

**Zigbee Alliance Documents:**

075366r02ZB_AFG-ZigBee_Cluster_Library_Public_download_version.pdf:  This document describes the ZigBee Cluster Library framework and it is essential that it be understood in order to use this cluster definition document.


## Definitions

| Term | Definition |
| --- | --- |
| Controller | Control4 System Controller running the central processes that manage drivers, communications, and networking. | | There can be multiple Controllers within a system. Each Controller can act as a ZigBee Access Point. |
| ZigBee Access Point | A process running on a Control4 Controller that behaves as a ZigBee message aggregator in a                     | |Control4 ZigBee network.  All ZigBee messaging on Control4 ZigBee networks go through one or more ZigBee Access 
| |Points. There can be multiple access points on a single mesh. Also referred to as a ZAP. |
| ZServer | The process running on a Control4 Controller that manages one or more ZigBee Access Points, and provides an IP 
| | based API to clients that wish to send and receive messages in the Control4 ZigBee network. There is only one ZServer per 
| | instance of mesh, but there can be multiple meshes in a single installation.
| | Director A central process running on a designated Control4 Controller that manages all other Control4 Controllers, 
| | networking traffic, and drivers within a Control4 system. A single Director can manage one or more ZigBee meshes through 
| | one or more ZServers. These can reside on different Controllers within the Control4 system. |


## Conformance Levels

| Value | Definition |
| --- | --- |
| 3.1.1 expected | A key word used to describe the behavior of the hardware or software in the design models assumed by this 
| | Draft. Other hardware and software design models may also be implemented. |
| 3.1.2 may | A key word that indicates flexibility of choice with no implied preference. |
| 3.1.3 shall | A key word indicating a mandatory requirement. Designers are required to implement all such mandatory 
| | requirements. |
| 3.1.4 should | A key word indicating flexibility of choice with a strongly preferred alternative. Equivalent to the phrase, “is 
| | recommended”. |


## Acronyms and Abbreviations

| Acronym | Meaning |
| --- | --- |
| ZAP | ZigBee Access Point |
| SSCP | Special Sauce Control Protocol (Control4 internal name for Control4 Networking Cluster) |
| MTORR | Many-To-One Route Request |


## Introduction

The Control4 ZCL Networking Cluster described by this document encapsulates a standard set of attributes and commands that make it easy for a ZigBee device to interoperate with a Control4 System Controller. This is implemented using a Control4 specific profile. By standardizing a set of attributes and commands, along with a set of prescribed behaviors in a Control4 specific ZCL cluster, a Control4 System Controller can be agnostic to the stack vendor and device hardware architecture of the ZigBee device that it wishes to communicate with.

A Control4 ZigBee network can be described as a command and control network, where all ZigBee commands originate from, or are sent to, a central location (a Control4 System Controller). There can be more than one Control4 Controller providing this role in a single Control4 ZigBee network. These are known as ZigBee Access Points, or ZAPs. A Control4 ZigBee network is not a peer to peer network, i.e. a keypad controlling a light switch does not send the on/off command directly to the light switch. Instead, it sends a keypad button press event to a Control4 Controller, and the Controller then routes the message to any other devices in the ZigBee network that are interested when this event happens. The advantage of this design is that other system events can be triggered simultaneously with a single message, such as updating a UI, sending an email, or notifying an IP audio receiver to switch inputs, etc. 

Control4 ZigBee networks are often large, with 200-300 nodes. Because of this, Control4 routers utilize ZigBee Many-To-One routing.  Many-To-One routing is a feature of the ZigBee-Pro stack profile that allows a central access point to aggregate all messages back to itself. Control4 networks also have an enhanced feature in that there can be multiple aggregating access points (ZAPs). Each of these will send MTORRs to allow Control4 router nodes to aggregate the best routes back to each separate ZAP, and eventually reach a single central controller within the System. 

A Control4 network, including one with multiple ZigBee Access Points, is controlled by a single central controller running a process called Director. Director is a central message routing hub for all of the devices in a Control4 system. This process ensures that all incoming and outgoing messages from one or more access points will reach the proper destinations. Because of this, the concept of a ZigBee binding is not necessary in a Control4 system. There is still a requirement to be able to associate a particular node to the central controller in an easy and intuitive way. Control4 refers to this as the Commissioning process, and the messages involved as an “Identify”. An Identify can actually be more than one message due to ZigBee payload limitation, but essentially describes a single set of information about the device. 

This document describes a set of attributes by Control4 Controllers to uniquely “Identify” devices within a Control4 system, and the associated commands these devices must support. These attributes and commands are multi-purposed. In addition to providing some basic information about the device when it joins a Control4 ZigBee network, these attributes are also used to diagnose network health, online/offline status, and help correct network failure modes. Some of these attributes are mandatory within a Control4 network.


## Cluster List

_The clusters defined in this document are listed in the following table._

| Cluster Name | Description |
| --- | --- |
| Control4 Networking Cluster | Attributes and commands for a Control4 ZigBee network. |


_The following image represents the typical usage of this cluster._

![]()ZCL_01


## Control4 Network Cluster

- Server and Client
- Dependencies: None


### Attributes

The currently defined attributes for this cluster are listed in Table 3. It may be observed that Control4 Controllers and routers send other attributes on this same profile and cluster. These are not required as they are extensions implemented as new capabilities become available. They can safely be ignored if not part of the following list. 

| Identifier | Name | Type | Range | Access | Default | Mandatory/Optional | Mandatory Reporting to ZServer |
| --- | --- | --- | --- | --- | --- | --- | --- |
| 0x0000 | DEVICE_TYPE | Unsigned 8-bit integer | 0x03 – 0x04 | Read only | 0x03 | M | M |
| 0x0001 | ANNOUNCE_WINDOW | Unsigned 16-bit integer | 0x0f – 0xffff seconds |Read/Write | 0x12c | O |O |		
| 0x0002 | MTORR_PERIOD | Unsigned 16-bit integer |0x000f-0xffff seconds | Read/Write | 0x12C | O | O |
| 0x0004 | FIRMWARE_VERSION | Char String | - | Read only | Null String | M M |
| 0x0005 | REFLASH_VERSION | Unsigned 8-bit integer | 0xff (vendor specific) | Read only | 0xff | M M |
| 0x0006 | BOOT_COUNT | Unsigned 16-bit integer | 0-0xffff | Read only | 0 | M | M |
| 0x0007 | PRODUCT_STRING | Char String | - | Read only | Null String | M | M |
| 0x0008 | ACCESS_POINT_NODE_ID | Unsigned | 16-bit integer | 0-0xffff | Read/Write | 0xffff | M |O |
| 0x0009 | ACCESS_POINT_LONG_ID | IEEE Address | - | Read/Write |	0xffffffffffffffff | M | O |
| 0x000A | ACCESS_POINT_COST | Unsigned 8-bit integer | 0-0xff | Read/Write | 0xff | M | O |
| 0x000C | MESH_CHANNEL | Unsigned 8-bit integer | 0x0b – 0x19 | Read/Write | 0x0b | M | M |
| 0x0013 | AVG_RSSI | Signed 8-bit integer | -128-0 | Read Only | 0 |O | O |
| 0x0014 | AVG_LQI | Unsigned 8-bit integer | 0-0xff | Read Only | 0 |	O | O |
| 0x0015 | BATTERY_LEVEL | Signed 8-bit integer | 0-100 (percent) | Read Only | 0 |	O | O |
| 0x0016 | RADIO_4_BARS | Unsigned 8-bit integer | 0-4 | Read Only | 0 | O | O |


### DEVICE_TYPE Attribute_

The DEVICE_TYPE attribute indicates what type of ZigBee Device the device is configured to use, i.e. non-polling end device, polling (sleepy) end device. Partner devices are only supported in an end device configuration (polling END_DEVICE, or non-polling SLEEPY_END_DEVICE). Note that the non-polling END_DEVICE configuration allow the same communications performance as a router device, with the benefit of a smaller stack size. Control4 routers implement a number of features not described by this document, and typically installation scenarios provide more than enough surround routers, so if the planned configuration is to use a router with a non-Control4 network, it should instead be configured as END_DEVICE when used withing a Control4 network. 

The possible types of devices are as follows (others are reserved):

| Name | Description | Value | 
| --- | --- | --- |
| END_DEVICE | Communicates only with parent, will not route messages.  Parent does not impede or cache messages destined for this node. Receives messages as quickly as a router. Node only has to poll parent occasionally to make sure it still has a parent and to remain in the parents child table. | 0x03 |_
| SLEEPY_END_DEVICE | End device that can turn off radio, must poll parent for messages. | 0x04 |

### ANNOUCE_WINDOW Attribute_

In a Control4 network, the system controllers periodically send out ZigBee Many-To-One Route Request (MTORR) broadcasts.  A system controller that has a ZigBee radio, and is able to send an MTORR, is called a Control4 ZigBee Access Point (ZAP). This broadcast establishes routes back to the ZAP from each Control4 router within the network. Note that end devices always send route records to the destination node, and therefore do not utilize the MTORR itself. 

Upon hearing an MTORR, a Control4 router will adjust and internal tracking of multiple best access points. The Control4 routers pick a single destination from this information. To establish the route from the Control4 router back to the ZAP, a ZigBee route record message must be generated from the router back to the ZAP.  In order to avoid a flood of route records, Control4 routers jitter their responses back to the ZAP.  The time they jitter the responses is known as the ANNOUNCE_WINDOW. The ANNOUNCE_WINDOW indicates what period of time (in seconds) after hearing a MTORR that a Control4 router must send in a route record message, followed by a report attributes of some of the mandatory Control4 Network Cluster attributes. The minimum jitter time for this message is 15 seconds, and maximum is 65535. By default, 300 seconds is used.

Since all of the Control4 Network Cluster attributes cannot fit in a typical ZigBee packet, the Control4 devices rotate between sets of attributes they report each time it they hear an MTORR.

A Control4 automation system may contain more than one ZAP.  Control4 routers gather statistics about the best access points within the system, and provide this information through an attribute to end devices interested in discovery the best access point to send messages to. An end device is free to send events to any ZAP in the network, but since Control4 routers gather statistics about the best access points within the system, it is recommended that end devices query the access point information from their parent Control4 router whenever they join or rejoin the network.
 
Control4 routers can be queried by end devices, and asked for the following attributes: ACCESS_POINT_NODE_ID, ACCESS_POINT_LONG_ID, and ACCESS_POINT_COST. They can also periodically poll the parent router for updated access point information, though this should not be performed more often that the MTORR_PERIOD default period of 300 seconds (5 minutes) to preserve network bandwidth. In the future, they may also expect to have this information changed at any time from a parent router. _

In a Control4 system, a router would typically not hear another MTORR until after a ANNOUNCE_WINDOW time period has passed.  Exceptions are made by a ZAP when it deems routes are in need of repair. End devices never hear MTORR’s and instead automatically send in route records every time the send a message to a destination. 


### MTORR_PERIOD Attribute_

The MTORR_PERIOD attribute indicates how often a ZAP will send an MTORR, and therefore the frequency a parent router could be expected to change to a new access point.  ZServer will typically set this to 300 seconds by default, but may use a slower value to preserve network bandwidth. The value is also multiplied by the number of ZAPs within the system (i.e. 2 zaps would produce an MTORR by each zap every 600 seconds). An end device may query this value, and use it to determine how often to poll a parent for new access point information. 300 seconds is always a safe value as a default. An end device will not benefit from polling a parent any faster than this value, since the routes will not change any faster than this. _


### FIRMWARE_VERSION Attribute_

This is an arbitrary string that indicates firmware version.  The format of this string is optional. Control4 typically implements firmware strings as follows: “01.02.03” where 01 is the major version (all values in hex), 02 is the minor version, and 03 is the build version. 
Note that the string is not limited to a particular number of characters or format, but it is recommended that length be limited for payload reasons. Excessive length of this string, or other variable length attributes, such as the product string, may result in failure on the part of the stack to be able to send the packet due to excessive length. 


### REFLASH_VERSION Attribute_

This attribute indicates the over the air reflash algorithm the device supports.  This field is reserved for Control4 devices and reflash mechanisms. Other vendors should always use the vendor specific option of 0xFF unless implicitly implementing a Control4 specific reflash protocol. A reflash API is provided within the DriverWorks SDK for updating vendor devices. This can be utilized by vendors reporting VENDOR_SPECIFIC as their reflash profile.   _


| Reflash Version | Description | Value | 
| --- | --- | --- |
| VENDOR_SPECIFIC | Over the air reflash is handled between the device driver and the device. This is the DriverWorks SDK method for handling reflash within the driver | 0xFF |_


### BOOT_COUNT Attribute_

This attribute indicates how many times the device has rebooted. Typically, this is stored in NVS, incremented each time the device reboots, and only reset upon a factory reset procedure. It may be helpful to not roll this value at 0xFFFF, since this typically indicates a problem. This can be helpful in determining some hardware failure modes that may be causing a devices to reboot repeatedly. 



