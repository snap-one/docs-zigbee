
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
12. Error Vector Magnitude less than 15%.
	 

## Router

In addition to the above, router implementations also require the following:

1. Support for Distributed Trust Center mode.
2. Support for adjustable MTORR period and adjustable Announce Window.
3. Adheres to announcement jitter and back-off behavior.
4. Uses Control4 stack configurations for maximum hops, stack profile, security level, end-device poll timeout shift, end-device child count that is compatible with Control4 configuration.
5. Implements required SSCP as per our router specification. This includes end-device access point information. 
6. Implements access point tracking algorithm for 3 or more access points.
7. Implements access point requests from a child end device.


## Details

### Implements Control4 or equivalent joining algorithm

802.15.4 provides a very basic mechanism for discovering and joining a network. The method is to send a single one-hop 802.15.4 message called a beacon request, requesting that all neighboring nodes within one hop send a beacon response. This response indicates if they are in permit joining mode. The requesting device stack implementation can only process a subset of these responses. In the presence of many responses, some will be lost. After receiving beacon responses, the requester then must prioritize responses and select a PAN to join. The goal is to obtain a list of all PAN's with nodes allowing joining, and then successfully to join the "best" one. This criteria for selection of a PAN is up to the implementer. 

Control4 recommends the following joining algorithm to ensure reliable joining. Since this is application code, this will be vendor specific and may vary from this recommended implementation:

1. Set a random channel.
2. Send out a beacon request.
3. Handle incoming beacons responses for the ZigBee-Pro specified active scan duration of 3, or 138 milliseconds (note that nodes must match the node they are joining, meaning zserver, parent routers, and all end devices must use the same scan duration). Store up to 16 PAN candidates, each being stored if all of the following criteria are true: 

- Beacon response has the "permit joining" flag set to true.
- The stack profile matches the ZigBee-Pro stack profile (2 for secure ZigBee-Pro devices, 0 for non-secure).
- The pan is unique, and has not already been stored.

4. If joinable PAN(s) found, and if multiple PANs are found, attempt to join the PAN with the best (highest) LQI (or equivalent metric) during the beacon scan process. Otherwise attempt to join the single PAN found. 
5. If joinable PAN(s) not found, perform steps 2-5 on the next channel. Continue until all channels are attempted.
6. If a join is attempted and fails for the following reasons: mac transmit failure or no beacon responses, then repeat steps 1-5.
7. In case of join failures, retry steps 1-6 up to 5 times. 


**Notes:** 
Silicon Labs stacks have included a simplified version of the above algorithm for some time. The included method of joining was found deficient in a number of areas. Silicon Labs has noted that the joining code provided as part of a GA stack release is for demonstration purposes, and vendors are recommended to implement their own method to harden the implementation. Some areas where the Silicon Labs joining code exhibited problems when: joining in presence of multiple PANs, joining first pan on lowest channel, not handling scan error status or retries. In the case of Silicon Labs EM357, there was a defect in some stack variants wherein the code relied on byte alignment for a memory copy of duplicate PAN information. This caused a situation where PAN information was being overwritten on EM357 platforms in the presence of more than one PAN. It is expected there is a limitation on number of discoverable PANs. The device should support at least 16 PAN’s not permitting joining. If there are more than 16 PAN's not permitting joining, PANs that are permitting joining may not be discovered within all retry attempts. The PAN limit is often determined by free RAM and the size of message buffers provided by the stack, and is therefore vendor specific. This can be an issue in hospitality and MDU environments. It can also be a problem in dense QA, engineering development, or manufacturing environments, where many PANs are present in a condensed area. For this reason, it may also be useful to provide code to join a specific PAN.


## Support for ZDO leave request

Device should leave the network when sent a ZDO leave request. 

Validation: 

1. Add the device to a mesh.
2. Send a ping request to confirm the device is communicating on the mesh.
3. Send a ZDO leave using zman→leave.
4. Send a ping request to confirm the device is not responding to ping.
5. Confirm with a sniffer the device is no longer sending messages on the mesh (link status exchange or ping ack/response).


## Support for ZDO channel change request

Device should change channels upon reception of a ZDO channel change request. 

Validation: 

1. Add the device to a mesh.
2. Send a ping request to confirm the device is communicating on the mesh and correct channel.
3. Confirm with a sniffer the device is on the expected channel.
4. Send a ZDO channel change using zman→channel.
5. Change the sniffer channel to the new channel.
6. Wait for 10 seconds.
7. Send a ping request to confirm the device is still responding to ping on the new channel.
8. Confirm with a sniffer the device is sending messages on the correct channel.


## Uses secure rejoin only

When certain failures occur, the prescribed method of resolving communications problems is for a ZigBee device to rejoin the network. ZigBee-Pro devices can implement two different methods of rejoining network, secure and un-secure. The method for un-secure rejoin is to attempt to rejoin securely using the current network key, and if this fails, fallback to obtaining a network key using the pre-configured link key. This is the HA standard method of rejoining, where the network key may have changed. Since the preconfigured link key ("ZigBeeAlliance09") is well known, this makes it very easy to setup a sniffer and watch for the network key exchange for a device rejoining un-securely. This only applies to centralized trust center mode, however. Control4 uses distributed trust center mode, where each device can allow another device to negotiate a network key while joining. Since there is no reliable method of distributing a network key change in distributed trust center, and since the un-secure method exposes the network key over the air, Control4 strictly disallows this method of rejoining. The primary area of concern is that a parent router does not allow an un-secure device to obtain the network key using the preconfigured link key. This is a policy decision for a router firmware application when running in centralized or distributed trust center mode. The secondary concern is that the device does not attempt to rejoin using an unsecure rejoin. This would create a situation where the network key could be captured by forcing nodes within an installation into a rejoining mode. 

Validation:

1. Join a device that performs unsecure rejoin to a routing device under test.
2. Confirm that the routing device under test does not respond to un-secure rejoin requests.
3. Trigger the device under test to rejoin the network (e.g. generate route failures).
4. Ensure the device under test does not perform an unsecure rejoin using the preconfigured link key "ZigBeeAlliance09".


**Test Case 1:** 
Join DUT to the network. Generate noise in network such that DUT attempts  to rejoin the network, make sure DUT never attempts unsecure rejoin. Require logging changes, TRAN-2645 to track this modification. 


**Test Case 2:** 
Test device under test always perform secure join. We can verify this through sniffer trace or need same additional logging traces to be added. For logging TRAN-2645 is created.


**Test case 3:**  
Test DUT allows only secure joining: Add DUT in network and try to add another unsecure device to the network. To simulate this initiate "allow join" from ZMAN, turn off the controller, click on unsecure device so that it initiates join and DUT should reject this unsecure join. Currently we are not having any ZigBee device that initiate unsecure join so require modification in template, TRAN-2645 to track this modification. 


## Implements required SSCP as per Control4 router specification

Control4 controllers expect a minimum set of attributes for a device to become marked "online". Online status in zserver is required for director and drivers to perform initial configuration, match system state visible in UI's, and display connectivity status in Composer, Composer Express, and other tools. 

Validation:

Confirm device under test sends a report attributes message on Profile 0xC25D Cluster 0x0001 containing the following minimal set of attributes:

- 0x0000 - Device Type (0x03 End Device, 0x02 Router)
- 0x0001 - Announce Window
- 0x0002 - MTORR Period
- 0x0003 - Number of Zaps (number of zaps in mesh)
- 0x0004 - Firmware version (any string)
- 0x0005 - Reflash version (must be 0xFF)
- 0x0006 - Boot count (number of reboots)
- 0x000B - Access point poll period (mandatory if end device)
- 0x000C - Mesh channel 

Also see "Adheres to compatible announcement behavior" below regarding how some of these attributes are used.


## Support for adjustable MTORR period and adjustable Announce Window

Control4 network cluster allow for a controller to configure the device as the network scales. The configuration determines how often the device expects to hear MTORR's, and how quickly the device sends announcement information. This has the combined effect of allowing the device to determine when it is "lost" and must seek the network, and when zserver determines a device is "offline" and no longer in communication with the access points.

Validation: 

Confirm device under test allows setting and getting the following attributes on Profile 0xC25D Cluster 0x0001:

1. Write announce window to arbitrary value using Profile 0xC25D Cluster 0x0001 Attribute 0x0001. Read back announce window and confirm it was set.
2. Write MTORR period to arbitrary value using Profile 0xC25D Cluster 0x0001 Attribute 0x0002. Read back MTORR period and confirm it was set.
3. Confirm the device performs announcements according to "Adheres to compatible announcement behavior" below
