
# Control4 Zigbee Cluster for End Devices

## Scope and Purpose

This content specifies the Control4 Networking Cluster, which is an application level interface between ZigBee devices and a Control4 System Controller. In general, use of clusters from the ZigBee Cluster Library promotes interoperability between ZigBee devices.  Use of this cluster promotes interoperability between devices and a Control4 System Controller by defining a standard set of attributes and commands that Control4 has found useful for a typical Control4 ZigBee network topology
This document should be used in conjunction with the ZigBee Cluster Library.


## References

The following standards and specifications contain provisions, which through reference in this document constitute provisions of this specification. All the standards and specifications listed are normative references.  At the time of publication, the editions indicated were valid. All standards and specifications are subject to revision, and parties to agreements based on this specification are encouraged to investigate the possibility of applying the most recent editions of the standards and specifications indicated below.

**Zigbee Alliance Documents:**

_075366r02ZB\_AFG-ZigBee\_Cluster\_Library\_Public\_download\_version.pdf:_  This document describes the ZigBee Cluster Library framework and it is essential that it be understood in order to use this cluster definition document.


## Definitions

| Term | Definition |
| --- | --- |
| Controller | Control4 System Controller running the central processes that manage drivers, communications, and networking. | There can be multiple Controllers within a system. Each Controller can act as a ZigBee Access Point. |
| ZigBee Access Point | A process running on a Control4 Controller that behaves as a ZigBee message aggregator in a Control4 | | ZigBee network.  All ZigBee messaging on Control4 ZigBee networks go through one or more ZigBee Access Points. There 
| | can be multiple access points on a single mesh. Also referred to as a ZAP. |
| ZServer | The process running on a Control4 Controller that manages one or more ZigBee Access Points, and provides an IP 
| | based API to clients that wish to send and receive messages in the Control4 ZigBee network. There is only one ZServer per 
| | instance of mesh, but there can be multiple meshes in a single installation. |
| Director | A central process running on a designated Control4 Controller that manages all other Control4 Controllers, |
| | networking traffic, and drivers within a Control4 system. A single Director can manage one or more ZigBee meshes through |
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


<img src="images/4_ZCL.png"/> 



## Control4 Network Cluster

- Server and Client
- Dependencies: None


### Attributes

The currently defined attributes for this cluster are listed in Table 3. It may be observed that Control4 Controllers and routers send other attributes on this same profile and cluster. These are not required as they are extensions implemented as new capabilities become available. They can safely be ignored if not part of the following list. 

| Identifier | Name | Type | Range | Access | Default | Mandatory/Optional | Mandatory Reporting to ZServer |
| --- | --- | --- | --- | --- | --- | --- | --- |
| 0x0000 | DEVICE\_TYPE | Unsigned 8-bit integer | 0x03 – 0x04 | Read only | 0x03 | M | M |
| 0x0001 | ANNOUNCE\_WINDOW | Unsigned 16-bit integer | 0x0f – 0xffff seconds |Read/Write | 0x12c | O |O |		
| 0x0002 | MTORR\_PERIOD | Unsigned 16-bit integer |0x000f-0xffff seconds | Read/Write | 0x12C | O | O |
| 0x0004 | FIRMWARE\_VERSION | Char String | - | Read only | Null String | M M |
| 0x0005 | REFLASH\_VERSION | Unsigned 8-bit integer | 0xff (vendor specific) | Read only | 0xff | M M |
| 0x0006 | BOOT\_COUNT | Unsigned 16-bit integer | 0-0xffff | Read only | 0 | M | M |
| 0x0007 | PRODUCT\_STRING | Char String | - | Read only | Null String | M | M |
| 0x0008 | ACCESS\_POINT\_NODE\_ID | Unsigned | 16-bit integer | 0-0xffff | Read/Write | 0xffff | M |O |
| 0x0009 | ACCESS\_POINT\_LONG\_ID | IEEE Address | - | Read/Write | 0xffffffffffffffff | M | O |
| 0x000A | ACCESS\_POINT\_COST | Unsigned 8-bit integer | 0-0xff | Read/Write | 0xff | M | O |
| 0x000C | MESH\_CHANNEL | Unsigned 8-bit integer | 0x0b – 0x19 | Read/Write | 0x0b | M | M |
| 0x0013 | AVG\_RSSI | Signed 8-bit integer | -128-0 | Read Only | 0 |O | O |
| 0x0014 | AVG\_LQI | Unsigned 8-bit integer | 0-0xff | Read Only | 0 |	O | O |
| 0x0015 | BATTERY\_LEVEL | Signed 8-bit integer | 0-100 (percent) | Read Only | 0 | O | O |
| 0x0016 | RADIO\_4\_BARS | Unsigned 8-bit integer | 0-4 | Read Only | 0 | O | O |


### DEVICE\_TYPE Attribute

The DEVICE\_TYPE attribute indicates what type of ZigBee Device the device is configured to use, i.e. non-polling end device, polling (sleepy) end device. Partner devices are only supported in an end device configuration (polling END\_DEVICE, or non-polling SLEEPY\_END\_DEVICE). Note that the non-polling END\_DEVICE configuration allow the same communications performance as a router device, with the benefit of a smaller stack size. Control4 routers implement a number of features not described by this document, and typically installation scenarios provide more than enough surround routers, so if the planned configuration is to use a router with a non-Control4 network, it should instead be configured as END\_DEVICE when used within a Control4 network. 

The possible types of devices are as follows (others are reserved):

| Name | Description | Value | 
| --- | --- | --- |
| END\_DEVICE | Communicates only with parent, will not route messages.  Parent does not impede or cache messages destined for this node. Receives messages as quickly as a router. Node only has to poll parent occasionally to make sure it still has a parent and to remain in the parents child table. | 0x03 |
| SLEEPY\_END\_DEVICE | End device that can turn off radio, must poll parent for messages. | 0x04 |

### ANNOUNCE\_WINDOW Attribute

In a Control4 network, the system controllers periodically send out ZigBee Many-To-One Route Request (MTORR) broadcasts.  A system controller that has a ZigBee radio, and is able to send an MTORR, is called a Control4 ZigBee Access Point (ZAP). This broadcast establishes routes back to the ZAP from each Control4 router within the network. Note that end devices always send route records to the destination node, and therefore do not utilize the MTORR itself. 

Upon hearing an MTORR, a Control4 router will adjust and internal tracking of multiple best access points. The Control4 routers pick a single destination from this information. To establish the route from the Control4 router back to the ZAP, a ZigBee route record message must be generated from the router back to the ZAP.  In order to avoid a flood of route records, Control4 routers jitter their responses back to the ZAP.  The time they jitter the responses is known as the ANNOUNCE\_WINDOW. The ANNOUNCE\_WINDOW indicates what period of time (in seconds) after hearing a MTORR that a Control4 router must send in a route record message, followed by a report attributes of some of the mandatory Control4 Network Cluster attributes. The minimum jitter time for this message is 15 seconds, and maximum is 65535. By default, 300 seconds is used.

Since all of the Control4 Network Cluster attributes cannot fit in a typical ZigBee packet, the Control4 devices rotate between sets of attributes they report each time it they hear an MTORR.

A Control4 automation system may contain more than one ZAP.  Control4 routers gather statistics about the best access points within the system, and provide this information through an attribute to end devices interested in discovery the best access point to send messages to. An end device is free to send events to any ZAP in the network, but since Control4 routers gather statistics about the best access points within the system, it is recommended that end devices query the access point information from their parent Control4 router whenever they join or rejoin the network.
 
Control4 routers can be queried by end devices, and asked for the following attributes: ACCESS\_POINT\_NODE\_ID, ACCESS\_POINT\_LONG\_ID, and ACCESS\_POINT\_COST. 

They can also periodically poll the parent router for updated access point information, though this should not be performed more often that the MTORR\_PERIOD default period of 300 seconds (5 minutes) to preserve network bandwidth. In the future, they may also expect to have this information changed at any time from a parent router. \_

In a Control4 system, a router would typically not hear another MTORR until after a ANNOUNCE\_WINDOW time period has passed.  Exceptions are made by a ZAP when it deems routes are in need of repair. End devices never hear MTORR’s and instead automatically send in route records every time the send a message to a destination. 


### MTORR\_PERIOD Attribute

The MTORR\_PERIOD attribute indicates how often a ZAP will send an MTORR, and therefore the frequency a parent router could be expected to change to a new access point.  ZServer will typically set this to 300 seconds by default, but may use a slower value to preserve network bandwidth. The value is also multiplied by the number of ZAPs within the system (i.e. 2 zaps would produce an MTORR by each zap every 600 seconds). An end device may query this value, and use it to determine how often to poll a parent for new access point information. 300 seconds is always a safe value as a default. An end device will not benefit from polling a parent any faster than this value, since the routes will not change any faster than this. \_


### FIRMWARE\_VERSION Attribute

This is an arbitrary string that indicates firmware version.  The format of this string is optional. Control4 typically implements firmware strings as follows: “01.02.03” where 01 is the major version (all values in hex), 02 is the minor version, and 03 is the build version. 
Note that the string is not limited to a particular number of characters or format, but it is recommended that length be limited for payload reasons. Excessive length of this string, or other variable length attributes, such as the product string, may result in failure on the part of the stack to be able to send the packet due to excessive length. 


### REFLASH\_VERSION Attribute

This attribute indicates the over the air reflash algorithm the device supports.  This field is reserved for Control4 devices and reflash mechanisms. Other vendors should always use the vendor specific option of 0xFF unless implicitly implementing a Control4 specific reflash protocol. A reflash API is provided within the DriverWorks SDK for updating vendor devices. This can be utilized by vendors reporting VENDOR\_SPECIFIC as their reflash profile. 


| Reflash Version | Description | Value | 
| --- | --- | --- |
| VENDOR\_SPECIFIC | Over the air reflash is handled between the device driver and the device. This is the DriverWorks SDK method for handling reflash within the driver | 0xFF |


### BOOT\_COUNT Attribute

This attribute indicates how many times the device has rebooted. Typically, this is stored in NVS, incremented each time the device reboots, and only reset upon a factory reset procedure. It may be helpful to not roll this value at 0xFFFF, since this typically indicates a problem. This can be helpful in determining some hardware failure modes that may be causing a devices to reboot repeatedly. 


### PRODUCT\_STRING Attribute

This attribute is a string that uniquely describes the device. It must be unique to the device SKU, and match the DriverWorks driver “search type”.  It is the string that a Control4 Controller uses to match the device with the appropriate Control4 system driver.

A vendor namespace convention is typically used for this string of “vendor:product type:model number”.  Here is an example of a Control4 Product string: 

“c4:light\_ip\_control4:ldz-101-w:”

Here would be an example of a third party product string: 

“acme:mouse\_trap:amt-11-22-33:”

The vendor unique string (i.e. “acme:” in the example) is treated as a namespace, and should always come first. The rest of the string should differentiate between this specific vendors devices.  This allows multiple vendors making different products with the same product type string, such as “light switch”, and allows drivers to uniquely identify devices.


### ACCESS\_POINT\_NODE\_ID Attribute

This attribute contains the short ID of a Control4 router best ZigBee Access Point. An end device within the Control4 system should query this attribute from its parent when joining for the first time, and when rejoining a parent, in order to determine the destination for its own messages. This is selected statistically based upon cost, failure rate, and other criteria, and reflects the destination the end device parent will to use for messages. End devices do not hear MTORRs, and therefore cannot determine the best access points to use. They should therefore utilize the parent Control4 router access point as their own destination as well.

Note in some cases, an end device will join directly to a ZigBee Access Point as a parent. The ZAP responds to the same queries as Control4 router, and will still notify the end device of the proper access point to use (i.e. itself). 
This attribute may be set by parent Control4 router or ZAP if the network topology requires it.  If an end device has this attribute set by a parent, it should not override the new setting unless it needs to change due to a normal network management event (i.e. such as a rejoin attempt).  

Note: when setting the access point information within the stack, the long ID should be set prior to setting the short ID. In some stacks, performing the opposite (setting the short ID first, and then the long ID) will invalidate the short ID with 0xFFFD. 


### ACCESS\_POINT\_LONG\_ID Attribute

This attribute contains the EUI64 long ID of a Control4 router primary ZigBee Access Point.  An end device within the Control4 system should always query this attribute from its parent in order to determine the destination for its own messages. It is recommended this occurs whenever a parent for the end device may change, such as during a new join, or a rejoin. It can also be done periodically to refresh the end device access point in case the parent ZAP has changed. If periodically polled, this should not occur any faster than the default MTORR\_PERIOD of 300 seconds. 

This attribute may be set by a parent to an end device if the network topology requires it.  If an end device has this attribute set by a parent, it should not override the new setting unless it needs to change due to a normal network management event (i.e. such as a rejoin attempt).

Note: when setting the access point information within the stack, the long ID should be set prior to setting the short ID. In some stacks, performing the opposite (setting the short ID first, and then the long ID) will invalidate the short ID with 0xFFFD. 


### ACCESS\_POINT\_COST Attribute
This attribute contains the cost of a reaching a ZigBee Access Point.  Parent devices track multiple Access Points and select them partially based upon cost. Since end devices only track a single Access Point, they have less use for this parameter, but can query it none-the-less for diagnostic purposes.


### MESH\_CHANNEL\_ Attribute

The MESH\_CHANNEL attribute indicates the ZigBee channel that the node is operating on, and is reported to Access Points to make sure it matches.  It is expected this would be the same channel a Control4 ZAP is using, but this isn’t always the case. Depending on hardware design, it is possible for a node to “hear” and join a ZigBee network on a “ghosted” channel. The ZigBee protocol itself does nothing to prevent this. In these cases, the node may appear to function, but encounter significantly poor radio performance. To prevent this, Control4 Controllers actively ensure all the nodes in a mesh are on a correct channel. A Control4 Controller will set this attribute on nodes if it is found to be different than the channel the Controller is expecting. If a node receives a request to set this attribute, it should honor this request and change its ZigBee stack channel to match the request.\_


### AVG\_RSSI Attribute

This value can be used to report the RSSI value of packets received by an end device from a parent. Usually a running average is maintained on each incoming packet. The values can help when diagnosing systems with sparse or weak links between nodes. 


### AVG\_LQI Attribute

This value can be used to report the LQI value of packets received by an end device from a parent. Usually a running average is maintained on each incoming packet. The values can help when diagnosing systems with sparse or weak links between nodes.


### BATTERY\_LEVEL Attribute

This value can be used to report the battery level of a battery powered end device. The value is reported as a percentage, from 0 to 100%. The value can be ignored if the end device is not battery powered. It will help the system passively track battery status on nodes, and can be used for diagnostics purposes as well. 


### RADIO\_4\_BARS Attribute

This attribute provides a pseudo “4-bar” radio signal quality indicator for each node in the network. This can be used to help diagnose communications problems to devices, and is displayed in the installer UI tools such as Composer. The value is typically based upon a combination of LQI and RSSI. LQI indicates link quality, and RSSI indicates receive signal strength. RSSI is a less reliable indicator of radio performance than LQI, and the 4 bar value is typically weighted accordingly. 

The algorithm for determining the strength of bars is primarily based upon average LQI values for incoming packets, and is only adjusted downward in case of exceptionally poor RSSI. The values used by Control4 devices are as follows:

| LQI | 4 Bar Signal Strength | 
| --- | --- | 
| 235-255 | 4 |
| 215-234 | 3 |
| 195-214 | 2 |
| 175-194 | 1 |
| Below 175 | 0 |

If average RSSI value is below -70dB, then the 4 bar signal strength is adjusted downward by one (i.e. an average LQI of 215 with an average RSSI value of -80dB would result in a signal strength of 3-1=2).

This can be dependent on hardware design, since an LNA/PA can boost transport and receive performance. It is also only a unidirectional value, since only the receive from a neighbor is being evaluated. These values are only shown as an example for reference, and vendor hardware can estimate this using some other criteria if desired. 


## Commands Sent and Received

The table below lists cluster specific commands are sent or received by the server.

| Command Identifier Field Value | Description | Mandatory/Optional |
| --- | --- | --- | 
| 0x00 | IMMEDIATE\_ANNOUNCE\_COMMAND\_ID | O |


### IMMEDIATE\_ANNOUNCE\_COMMAND\_ID

This command can be sent by Control4 Controller (received by a node) to request that an Announcement be sent immediately. An Announcement is simply the predetermined set of mandatory attributes defined by this document (see section 7 for details). There may be more than one “Announcement” required to report all of the required information (in this case send them back to back). 

The request for an immediate announce may be unicast, or it may be broadcast to the entire network. In the case of a broadcast, the list of nodes in the payload of the message must be used as a filter for the target nodes being requested to send in an Announcement.  This target node list is a little endian list of short ids.

This command is usually used to correct communications problems where a Control4 Controller does not have enough information about a particular node. In these cases, it is typically sent broadcast in order to generate a new Announcement back to itself. In that case, the target node id list must be used to limit the number of responders to those the Control4 Controller is interested in. 


## Announcements

### Mandatory Attribute Reporting (Announcements)

Many of the Control4 Network Cluster attributes should be reported to the ZServer process on a Control4 Controller when a device reboots, joins, or rejoins the network.  This is referred to as “Announcing”. The attributes being reported allow the Control4 Controller to update local state information about the device in the network, establish proper routes to reach the device, and correct any error conditions should they occur. The device should send in as many report attribute messages is required to fit the mandatory attributes. A minimal announcement can be used (as described below) to allow them all to be sent in a single packet, if desired. This can help preserve network bandwidth on a whole system restart, such as might occur after a brief power outage. 

On a regular interval, the device should send in an announcement unicast. If all the attributes do not fit within a single announcement, the device may send some attributes on one announcement, and the rest on another.  The frequency of sending each announcement is defined by the ANNOUNCE\_WINDOW for end devices. The only side effect of not send them frequently enough is that the device may show up as “offline” after several ANNOUNCE\_WINDOW intervals have expired without the system Controller hearing from the device. The device will appear online again on the next incoming message, however. Devices that sleep for extended periods may therefore show up as offline, but this doesn’t negatively affect communications in any way. 


### Control4 Identification Process (Identifying)

When a node reboots or joins a network, it should send in all of its Control4 Network attributes via one or more ZigBee broadcasts (destination = 0xFFFC).  This broadcast is identical to the Control4 System Controller in the commissioning process, and is referred to as “Control4 Identification”, or “Identifying”.  It also required that the device be able to send in the attribute report broadcast with a physical interaction, such as with a button push, in order to be able to re-identify a device that has already been joined the network. On Control4 devices, this is typically triggered by 4-tapping a button. This identification process binds individual nodes to the appropriate Control4 driver within the Control4 Controller.

An “Identify” and an “Announcement” contain the same attributes and packet format. The former is simply sent broadcast on a button tap sequence, when rebooting, or when joining the first time (cases when there may not be a route to a Controller). The latter is sent when rejoining, and periodically to stay “online” with the Control4 Controller. Since excessive broadcasts on a ZigBee network can degrade network performance, broadcasts are only used for this purpose in limited situations. 


### Minimal Announcement or Identify

While the above sections describe the full implementation of Control4 network handling used by Control4, this is not required for a device to be part of a Control4 network. A single message containing a subset of the attributes described above can be used to both “identify” a device, and to “announce” to stay online within the network.  These are referred to as minimal “Announcement” or “Identify”, depending upon the use case, and are described as follows: 

An “Announcement” is a unicast sent to a coordinator every few minutes to keep a node “Online”.  While an Announcement is intended to contain a dynamic set of attributes, the “Announcement” containing hardcoded values provided below will also serve the same function in a simple static form. This can be useful when first developing a new device to be used within the Control4 system. 

An “Identify” is the exact same packet as an “Announcement”, but is sent broadcast to short address 0xFFFC instead of unicast. This still uses the same profile and cluster as the Announcement.  An Identify is used to notify Director and drivers of a node joining the network for the first time, and to populate the Identify window within Composer. It is also often used when a device reboots as it may not have a valid Control4 access point to send to yet, so a broadcast is used to reach any within range. Broadcasts should not be used regularly to keep a node “Online” however, as hundreds of devices implementing this behavior could degrade network bandwidth considerably. 

The normal sequence for sending these messages is as follows: 

1. Join network using the stack provided method.
2. Send out an “Identify” after joining to populate the identify window.
3. Then send out “Announcements” every few minutes to keep the device online.

In the case of a unicast Announcements, sending this message every 5-10 minutes is sufficient to keep a node online as determined by the Control4 Controller. To preserve bandwidth with many devices, it is recommend that a random jitter be introduced when sending this message (say once every 15-300 seconds), and re-randomized after each announcement. This prevents blocks of nodes from repeatedly sending their Announcements in sync, such as might occur after a brief power outage affects an entire home. 

The “Announce” or “Identify” packets are sent using the following parameters:

`ZCL message type: report attributes (0x0a)`
`ProfileId: 0xc25d`
`ClusterId: 0x0001`
`Endpoint: anything but 0x00 or 0xff`

Here are the minimal ZCL attributes you’ll need to populate the “Announce” or “Identify” packet.

`AttributeId 0x0007
``AttributeType 0x42
``AttributeValue = length+"PROD\_STRING\_MAKE\_IT\_LESS\_THAN\_8\_CHARS"
`  
`AttributeId 0x0004`
`AttributeType 0x42`
`AttributeValue = length+"01.01.01"`
  
`AttributeId 0x0005`
`AttributeType 0x20`
`AttributeValue 0x03`
  
`AttributeId 0x0006`
`AttributeType 0x21`
`AttributeValue = 0x0130`
  
`AttributeId 0x0000`
`AttributeType 0x20`
`AttributeValue = 0x02`
  
`AttributeId 0x0001`
`AttributeType 0x21`
`AttributeValue 0x0123`
  
`AttributeId 0x0002`
`AttributeType 0x21`
`AttributeValue 0x012c`
  
`AttributeId 0x0003`
`AttributeType 0x20`
`AttributeValue 0x01`
  
`AttributeId 0x000b`
`AttributeType 0x21`
`AttributeValue 0x012c`
  
`AttributeId 0x000c`
`AttributeType 0x20`
`AttributeValue 0x0b`
  
  
When sending these attributes, be sure to implement the correct endian-ness according to the ZCL specification (i.e. little endian byte order). Sending the incorrect byte order is the most common error encountered for “Identify” or “Announcement” packets.  As an example, an attribute record 0x0001 containing an unsigned 16-bit value (datatype 0x21) with a value of 0x1234:

`AttributeId 0x0001`
`AttributeType 0x21`
`AttributeValue 0x1234`

Would be sent over the air in little endian byte order as 0100213412


### Additional End Device Attributes

In the case of Control4 networks, Control4 routers track a best access point based upon many-to-one-route-requests being received by the Control4 parent routers. Since end devices do not hear these messages, they are required to ask their parent for the relevant access point information to learn where to send messages. 

When a Control4 end node joins a router, it will query the following information from the router via read attributes requests using the following parameters. 

ZCL message type: read attributes request
ProfileId: 0xc25d
ClusterId: 0x0001
Endpoint: anything but 0x00 or 0xff

Control4 routers respond by providing the following read attributes responses. The end device can then use this information for the best Control4 access point. For example, the following values communicate to an end device that 0x0000 is the preferred access point for this end device.

AttributeId: 0x0008
AttributeType: ZCL\_INT16U\_ATTRIBUTE\_TYPE
AttributeValue: 0x0000

AttributeId: 0x0009
AttributeType: ZCL\_IEEE\_ADDRESS\_ATTRIBUTE\_TYPE
AttributeValue:  EUI64

AttributeId: 0x000a
AttributeType: ZCL\_INT8U\_ATTRIBUTE\_TYPE
AttributeValue: 0x00

Control4 systems support spanning a mesh across multiple access points through IP, allowing devices out of radio range of each other to communicate. The way an end device discovers which access point is within range is by querying the parent Control4 router or ZAP for the best access point within range. It is therefore important that end devices adhere to the access point information provided by the router/ZAP using these attributes. Simply sending to a predetermined coordinator (0x0000), for example, is not guaranteed to work in some network topologies, such as a pool house out of radio range of a main house. 


## Control4 Network Practices

The preceding text describes the over the air protocol used for the Control4 Network Cluster.  There is also a set of behaviors that it is strongly recommended that a device implement in order act consistently on a Control4 Network.  Many of these behaviors are described in the ZigBee Home Automation Profile document.

_Scanning and Joining_ - A Control4 Network may use ZigBee channels 11 – 25 in a released image.  Channel 26 is avoided in release images in order to not violate FCC rules.  When it is desired that a device be added to a Control4 system, the installer typically presses a button on the device four times.  This is called “identifying the device”. This causes the device to scan all 15 channels, looking for a PAN to join.  When a PAN that will allow joining is detected, the device joins the network and receives the network and a link key. There is a possibility that a device may miss a beacon with permit joining true. This is especially true when devices are first being joined to a new network, since devices across multiple PANs can flood a requesting device with beacon responses from other PANs that are not permitting joining. For this reason, it is recommended to try multiple beacon requests and beacon responses on each given channel, to ensure that the beacons from networks that not permitting joining will be disregarded and the beacon that is permitting joining will be detected.  

Control4 controllers handle the network key uniquely. During association, Control4 controllers will alternately send the key encrypted with the pre-configured link key, and then in the clear. This is designed to allow legacy devices to join that do not implement encrypted network keys as subsequently required by various ZigBee profiles. The joining device must be configured to either allow both options (e.g. not setting the EMBER\_REQUIRE\_ENCRYPTED\_KEY option for EmberInitialSecurityState on SiliconLabs stack, or equivalent), or implementing the joining state machine such that it retries when a key is received in the clear so that the subsequent encrypted key will be used.  

After a device successfully joins a network it is essential that it report its entire Control4 network attributes set to ZServer via one or more broadcast ZCL Attribute Report messages.  This “identifies” the device to the controller and pairs the device with the appropriate Control4 system driver.

If a device is already joined to a network, pressing the identify button four times (or what ever user interaction “identifies” the device) should cause the device to broadcast its Control4 network cluster attribute report again. This can be used to generate an online status event through the system. 

_Leaving a network_ – All Control4 ZigBee devices have a button sequence that causes them to leave the Control4 network.  The device should send out a ZDO device leave broadcast as it leaves.  This functionality is essential so that a device can be joined to another network.  Since Control systems are installed by certified installers, the button sequence should not allow for accidental triggering in normal device use.  Additionally, all devices should adhere to the ZDO leave request and implement the ZDO leave handler in firmware such that they can be told to leave the network over the air if the device is “disconnected” using an installation tool, or found to not match the driver it is being “identified” against. 

_Restoring to factory defaults_ – All Control4 ZigBee devices have a button sequence that causes it to restore all of the application settings to factory defaults.  This is usually (but not necessarily) the same button sequence as the leaving a network button sequence so both actions are completed at once.

_APS frame options_ – Devices must set the EmberApsFrame option EMBER\_APS\_OPTION\_SOURCE\_EUI64 for outgoing packets whenever possible. This allows the receiving controller to associate an incoming frame with an appropriate long and short ID of the source. Optionally, the outgoing packet should also use the EMBER\_APS\_OPTION\_DESTINATION\_EUI64 option. Be aware that these options come at the expense of payload size.

_End Device versus Router_ – Control4 routers implement specific behaviors to ensure the network health is maintained, and all devices can communicate properly. This behavior is not described within this document, and may change in the future to improve network performance, scaling, route stability, etc. For this reason, Control4 only supports end devices for non-Control4 manufactured devices, as described within this document. Note that non-polling end device configurations benefit from the same communications performance as a router, with the added benefit of more application space due to a smaller stack size. 

_Lost Node Behavior for End Devices_ – If an end node of any type cannot contact its parent for a number of polls, it should attempt to find a new parent on the current channel through a rejoin. If that fails, it should fallback to all other channels.  The sequence should be tried using the existing network key first (secure rejoin), and then fall back to retrieving a new network key if necessary. 

The period of time to wait before attempting to rejoin starts at no less than 10 seconds and grow on every unsuccessful rejoin attempt, doubling each time until the period is equal to 1 hour is reached between rejoin attempts. This doubling algorithm is intended to prevent large number of beacons storming a network of 50+ nodes, a conservative approach is adopted as required by the product.

If the node cannot find the network, it should wait for a period of time and try again.  During this wait period, the node should stay on the last known valid network channel in a state that is ready to receive messages. If any are received, the node should clear the lost state and resume normal operation until the next lost threshold is triggered. 

_End Node Parent Polling Time_ – Polling end devices must poll their parent at least every 7.68 seconds according to the specification and configuration. This is known as the EMBER\_INDIRECT\_TRANSMISSION\_TIMEOUT. Control4 recommends this actually be around every 7 seconds to ensure messaging is successful. A node may choose to poll its parent less frequently, but any client that sends a message to that device will experience datagram delivery failures if messages are sent while the node is asleep for more than 7.68 seconds. Some implementations may sleep longer intentionally to save battery life. In this case, it is often beneficial to implement an awake signal to the driver to negotiate when messages may be sent to the device, and when they should not. Be aware that if the device fails to poll for too long, it will be removed from the parent child table and be forced to rejoin the network as described above. Currently the child table timeout is set for over 4 hours, but this may be reduced to as low as 10-15 minutes in the future Control4 systems to avoid the long timeout on children that may have left the parent without its knowledge. 

Non-polling end devices do not need to poll the parent to retrieve messages, and therefore can poll much more infrequently. In this case, they are polling simply to stay in the parent child table, or to ensure the parent is there prior to triggering a lost/rejoin behavior. 

_Control4 Network Attribute Sets_ – ZServer will occasionally set some of the writeable Control4 Network Attributes on a node via broadcast or unicast.  It does so in order to adjust attributes such as the ANNOUNCE\_WINDOW appropriately for the size of the network.  A node on a Control4 network should have message handlers to handle the setting of these attributes.  Note that the list of attributes to be written in the ZCL write attributes command may come in any order and be of any length that will fit in a ZigBee packet.  

_Control4 Network Attribute Reporting_ – A node on a Control4 Network should report its network attributes periodically using a “ZCL Report Attributes” command.  An end device should send in a report once every ANNOUNCE\_WINDOW. The report should be sent at a random time ranging from 15 seconds to the ANNOUNCE\_WINDOW seconds, and sent unicast to the nodes best access point.  The random back off ensures that all the device on the network will not send a message at the same time, and the report ensures that the ZAP will hear a ZigBee route record from each node and that ZServer is kept up to date on the devices network attribute status.  Since all the attributes may not fit into a single message, it is not necessary to send all the attributes every time, but they should be sent in alternating sets.

_End Device Attribute Reporting Behavior_ – End device have to find a balance between battery life and performance.  In order to maximize battery life, they may send Control4 Network Attribute Reports at their own desired frequency. This however creates a problem when ZServer has rebooted and has yet to hear a Control4 Network Attribute report from the end device.  ZServer can not send any source routed data to the end device until it hears a route record from the end device’s parent, and can not do a very good job at sending a source routed datagram until it knows the device type of the end device.  This condition could have the effect of an end device being “offline” to ZServer after a ZServer reboot until the end device sends a regularly scheduled Control4 Network Cluster Attribute Report message.  To alleviate this problem, both ZServer and the end device may choose to implement some specific behaviors.  If ZServer hears a datagram from a node that it has not heard a Control4 Network Cluster attribute report,  it may send the command to request an IMMEDIATE\_ANNNOUNCE on the node via source routing (if a route record was heard), or via AODV routing. This should cause the end device to immediately send an attribute report message. Note that this technique will not always be effective if the end device goes to sleep, and does not poll its parent for a more than 7.68 seconds, as it may not hear the IMMEDIATE\_ANNOUNCE command.  Alternatively, if the end device sends a message to ZServer, expects to hear a reply at the APS level (from a Control4 driver), but fails to do so, then that should be a good indicator that ZServer is unable to send a message back due to routing issues and it may choose to send a Control4 Network Cluster attribute report to get “online” quickly.  


