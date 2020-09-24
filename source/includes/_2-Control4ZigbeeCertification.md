
# Control4 Zigbee Certification

## Overview

This content is to be used as a guide for implementation and certification of the Control4 Networking Cluster on an OEM product so that it can interoperate optimally with a Control4 System Controller. 

The content provides certification test details of behavior of ZigBee devices that implement the Control4 Large Network Cluster. During development, a Control4 controller with ZigBee capability can be utilized to test many of these functions using the ‘zman’command line interface present on the controller. This application allows direct control over the ZigBee server running on that controller. A ZigBee sniffer is also required. Finally, a method of measuring radio performance, such as error vector magnitude (EVM) and receive sensitivity.  

Preliminary Setup:

1. ssh to the controller at: root@\<controller ip\> password: t0talc0ntr0l4!
2. Type: sysman disable director
3. Type: sysman enable zap
4. Type: sysman enable zserver
5. Type: zman

The process of certifying a device with Control4 is to send two units running validation firmware to Control4 for certification testing, as well as providing instructions on how to enter a FCC or manufacturing test mode for streaming. Optionally, an additional unit may be sent with manufacturing test firmware for this purpose. 

The following areas of functionality will be tested based upon device type:

## End-Device (polling and non-polling)

1. Implements Control4 or equivalent joining algorithm to ensure reliable joining behavior in the presence of more than one PAN. 
2. Support for ZDO leave request.
3. Support for ZDO channel change request.
4. Uses secure rejoin only - Device does not expose the mesh network keys during rejoin behavior
5. Adheres to end-device announcement behavior.
6. Implements Control4 lost node behavior equivalent. Adheres to clustering behavior when changing channels, and performs access point queries when acquiring correct channel to avoid clustering on incorrect channels.
7. Implements HA standard polling and fast polling intervals.
8. Implements access point requests to parent router to determine best access point.
9. Implemented on a tested baseline "compatible" stack version and vendor.
10. Implemented with a General Availability (GA) stack release. 
11. Uses Control4 compatible stack configuration and SAS attribute set.
12. EVM less than 15% - See EVM (Error Vector Magnitude)
	 
## Router

In addition to the above, router implementations also require the following:

1. Support for Distributed Trust Center mode.
2. Support for adjustable MTORR period and adjustable Announce Window.
3. Adheres to announcement jitter and back-off behavior.
4. Uses Control4 stack configurations for maximum hops, stack profile, security level, end-device poll timeout shift, end-device child count that is compatible with Control4 configuration.
5. Implements required SSCP as per our router specification. This includes end-device access point information. 
6. Implements access point tracking algorithm for 3 or more access points.
7. Implements access point requests from a child end device.

