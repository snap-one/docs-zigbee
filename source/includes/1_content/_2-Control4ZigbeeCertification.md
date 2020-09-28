
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
3. Handle incoming beacons responses for the ZigBee-Pro specified active scan duration of 3, or 138 milliseconds (note that nodes must match the node they are joining, meaning Zserver, parent routers, and all end devices must use the same scan duration). Store up to 16 PAN candidates, each being stored if all of the following criteria are true: 

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

When certain failures occur, the prescribed method of resolving communications problems is for a ZigBee device to rejoin the network. ZigBee-Pro devices can implement two different methods of rejoining network, secure and un-secure. The method for un-secure rejoin is to attempt to rejoin securely using the current network key, and if this fails, fallback to obtaining a network key using the pre-configured link key. This is the HA standard method of rejoining, where the network key may have changed. Since the preconfigured link key ("ZigBeeAlliance09") is well known, this makes it very easy to setup a sniffer and watch for the network key exchange for a device rejoining un-securely. This only applies to centralized trust center mode, however. Control4 uses distributed trust center mode, where each device can allow another device to negotiate a network key while joining. Since there is no reliable method of distributing a network key change in distributed trust center, and since the un-secure method exposes the network key over the air, Control4 strictly disallows this method of rejoining. The primary area of concern is that a parent router does not allow an un-secure device to obtain the network key using the preconfigured link key. This is a policy decision for a router firmware application when running in centralized or distributed trust center mode. The secondary concern is that the device does not attempt to rejoin using an unsecured rejoin. This would create a situation where the network key could be captured by forcing nodes within an installation into a rejoining mode. 

Validation:

1. Join a device that performs unsecured rejoin to a routing device under test.
2. Confirm that the routing device under test does not respond to un-secure rejoin requests.
3. Trigger the device under test to rejoin the network (e.g. generate route failures).
4. Ensure the device under test does not perform an unsecured rejoin using the preconfigured link key "ZigBeeAlliance09".


**Test Case 1:** 
Join DUT to the network. Generate noise in network such that DUT attempts  to rejoin the network, make sure DUT never attempts unsecure rejoin. Require logging changes, TRAN-2645 to track this modification. 


**Test Case 2:** 
Test device under test always perform secure join. We can verify this through sniffer trace or need same additional logging traces to be added. For logging TRAN-2645 is created.


**Test case 3:**
Test DUT allows only secure joining: Add DUT in network and try to add another unsecured device to the network. To simulate this initiate "allow join" from ZMAN, turn off the controller, click on unsecured device so that it initiates join and DUT should reject this unsecured join. Currently we are not having any ZigBee device that initiate unsecured join so require modification in template, TRAN-2645 to track this modification. 


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

Control4 network cluster allow for a controller to configure the device as the network scales. The configuration determines how often the device expects to hear MTORR's, and how quickly the device sends announcement information. This has the combined effect of allowing the device to determine when it is "lost" and must seek the network, and when Zserver determines a device is "offline" and no longer in communication with the access points.

Validation: 

Confirm device under test allows setting and getting the following attributes on Profile 0xC25D Cluster 0x0001:

1. Write announce window to arbitrary value using Profile 0xC25D Cluster 0x0001 Attribute 0x0001. Read back announce window and confirm it was set.
2. Write MTORR period to arbitrary value using Profile 0xC25D Cluster 0x0001 Attribute 0x0002. Read back MTORR period and confirm it was set.
3. Confirm the device performs announcements according to "Adheres to compatible announcement behavior" below


## Device performs compatible announcement behavior

End devices and routers both must periodically send announcement information for Zserver to discover the device and mark it online. The period is configurable and is based upon the period of sending Many-To-One-Route-Requests (MTORRs). For routers, this period is communicated to the device using Profile 0xC25D Cluster 0x0001 Attribute 0x0001 (ANNOUNCE\_WINDOW) and (0x0002 MTORR\_PERIOD). 

See [Control4 ZCL Network Cluster Definition Document v1.09 ][1]

Validation:

1. Send an MTORR.
2. Confirm device sends an announcement within the announce window specified. 
3. Confirm the announcement contains the source EUI64 as part of the APS frame.
4. For multiple MTORR requests, confirm device randomizes response intervals. Announcements should be sent on different intervals ranging from 1 to n seconds, where n is the announce window configured through attribute 0x0001.
5. Confirm routing devices respond to access point requests for attribute 0x0008, 0x0009, and 0x000A with a preferred access point. Add a secondary zap, force the parent router to prefer this zap, and confirm this access point communicated to an end device with attributes 0x0008, 0x0009, 0x000A changes accordingly.
6. Configure a network with 2 access points. Send MTORR's from each access point. Confirm the number of zaps attribute reflects the number of zaps that have sent out MTORR's on the network. 
7. Repeat step 5 with 3 access points.
8. Confirm boot count increments each time the device power cycles.


## Implements access point request to parent router to determine best access point

In a mesh network with more than one access point (Zap), end devices must request the access point information from the parent router prior to sending messages. For end devices, this request should occur after rejoining. The end device must send a request for attributes 0x0008 (access point node id), 0x0009 (access point long id), and 0x000A (access point cost) to a parent router. The requests should occur on Profile 0xC25D and Cluster 0x0001. The parent router must respond with the attributes matching the "best" access point. If the access point is artificially forced to a new node id, the parent router should discover this after 3 MTORR periods, and subsequent communication to a joining or rejoining end device should also match. The end device should use the discovered access point any time it sends asynchronous messages.

Validation:

1. For end devices, ensure the requests are being sent to the parent router for Profile 0xC25D Cluster 0x0001, Attributes 0x0008, 0x0009, 0x000A.
2. For end device, ensure no messages are sent to an access point until all of these attributes have been requests and responded to (e.g. device does not trigger node id conflict on parent router).
3. For routers, ensure all attributes requested are responded to.
4. For routers, ensure as the access point changes, the attributes describing the access point match.


## Implements Control4 lost node behavior or equivalent

In the event of a channel change or PAN ID conflict, it is possible for nodes to become abandoned without a viable communication path to an access point. In these events, the device must automatically trigger a lost behavior, where it will seek out the network on the correct channel and PAN. This typically occurs through a rejoin process, first on the channel the device was on, and subsequently on all channels until the network is found. To avoid flooding a network in large installations, the device must not attempt this process more rapidly than once every 10 seconds. After the initial attempt, the device must extend the rejoin interval. Doubling the rejoin interval is recommended, until once per hour is reached. The recommended interval for starting the discovery process is 3 times the MTORR period + 9 seconds for broadcast propagation (configured through Profile 0xC25D Cluster 0x0001 Attribute 0x0002). The rejoin interval must scale based upon the number of Zaps (configured through Profile 0xC25D Cluster 0x0001 Attribute 0x0003). For example, if a device goes lost and seeks a network after 3 MTORR period intervals in a single Zap environment, it must allow for 3x3=9 MTORR periods (+9 seconds) in a 3 Zap environment. This is to handle the case where 2 out of 3 Zaps are not reachable by the device under test, plus maximum broadcast propagation delay.  Upon rejoining a network, the device must attempt to discover the access point on the given channel. The recommended method is to broadcast a manufacturer specific SSCP\_REQUEST\_PARAMS\_COMMAND\_ID 0x06 on Profile 0xC25D Cluster 0x0001. Upon receipt of this command, Zserver will broadcast a set channel attribute request on Profile 0xC25D Cluster 0x0001 Attribute 0x000C. The function of the set channel attribute request is twofold: 

1. It ensures there is an access point on the PAN able to communicate to the lost device.
2. It ensures the lost device is on the correct channel in cases of ghosting behavior (receiving messages on the incorrect channel).  Upon receipt the set channel attribute request, the device under test should compare the current channel to the channel attribute, and if they differ, switch channels to match. Devices that hear this request, including a device under test, should pause sending out their own requests pending a broadcast Zserver set channel attribute request. The design is intended to allow multiple devices to receive the SSCP\_REQUEST\_PARAMS\_COMMAND\_ID request signaling a node is lost, and also receive the associated broadcast set channel attribute request that allows them to determine if an access point is found and on what channel.  The pause delay should be randomized between 18 - 30 seconds to allow the broadcast request to propagate the subsequent set channel request to be received, and to provide an extra window for random jitter from requests from multiple nodes.  

Validation:

1. Setup a mesh with a controller and device under test.
2. Ensure the device under test cannot receive messages from the controller.
3. Change channels on the controller. This should leave the device under test on the wrong channel.
4. Ensure the device correct returns to the mesh after 30 minutes.
5. Setup a mesh with a controller, a Control4 router, and the device under test. 
6. Send a message to the Control4 router configuring the MTORR period to 0. This configures Control4 routers to never enter lost mode and attempt rejoins. 
7. Ensure the device under test and the router cannot receive messages from the controller. 
8. Change the channel on the controller. This should leave the router and the device under test on the wrong channel.
9. Ensure the device under test migrates away from the router on the wrong channel, and back to the controller on the correct channel. This tests access point detection during lost behavior to prevent islands of routers on incorrect channel.


## Implements HA standard polling and fast polling intervals

In their normal operating state, ZigBee end devices shall poll no more frequently than once every 7.5 seconds, except where the cluster specification indicates.  ZigBee end devices may operate with a higher polling rate during commissioning, network maintenance, alarm states, and for short periods after transmitting a message to allow for acknowledgments and or responses to be received quickly, but they must return to the standard rate indicated previously during normal operation.  If a device under test implements fast polling, the device should also implement the poll control cluster. This cluster allows configuring how the device fast polls. Attribute 0x0001 is the long poll interval in quarter seconds. Attribute 0x0002 is the short poll interval in quarter seconds. Attribute 0x0003 is the fast poll timeout in quarter seconds. When configured for fast polling, the device under test should not send messages any faster than once per quarter second and should automatically revert to the long polling interval after the configured fast polling timeout in quarter seconds. See the following documents for details: 

- docs-11-5474-59-00ha-zigbee-home-automation-profile-1-2-revision-for-053520r29.pdf 
- 07-5123-06-zigbee-cluster-library-specification.pdf

Validation: 

1. Verify the device under test does not poll more frequently than once every 7.5 seconds under normal use cases.
2. Verify that the device does not poll faster than 4 times per second in fast polling use cases.
3. Verify that the device reverts to a long polling interval (\> 7.5 seconds) after short polling completes. 
4. If the device supports the poll control cluster, very that setting the fast polling timeout reverts from the short poll interval to the long poll interval.


## Implemented on a tested baseline "compatible" stack version and vendor

Stack variants, and silicon implementations, can have varying routing capabilities. Methods of calculating link cost are varied, and depend upon vendor, or sometimes even within the same vendor. This can create “hot” routes, where all nodes in an installation will migrate to routing through a single node or node type. This was highlighted recently with a particular vendor, which had a fundamental shift in cost calculation from bit-error rate calculation to RSSI based calculations on a silicon revision. This change introduced route selection which favored extremely poor links due to imbalance in the link cost calculation and subsequent link status exchanges. Similar problems may occur with silicon from different vendors. Unfortunately, the ZigBee specification allows for variance in this regard between stack vendors. As a baseline, ensuring the device under test uses a stack that has undergone interoperability testing at an approved ZigBee Alliance test house (NTS or equivalent). Secondary, link cost can be observed monitoring the link status exchanges between routing nodes. These should balance with surrounding nodes, based upon receive sensitivity and transmit power of the device under test. 

Validation:

1. Verify the stack has passed core stack interoperability testing and certification with one of the test houses. See "Certification Test Houses". Confirm the device stack is GA release, and has passed core stack interoperability testing and certification with one of the test houses. See "Certification Test Houses"
2. Verify routing behavior does not favor or avoid device under test due to link cost calculation.


## EVM less than 15%

EVM (Error Vector Magnitude) is a measure of transmitter performance compared to an ideal transmitter. EVM is measured in absolute or offset values. For the purposes of Control4 certification, offset EVM is the primary factor of interest. 802.15.4 allows for up to 35% offset EVM, but EVM is not a single metric. A number of transmitter characteristics can contribute to EVM.  Some silicon, Silicon Labs parts for example, are more stringent about some of the contributing factors that lead to high EVM. For this reason, Silicon Labs recommends limiting a transmitter to below 15% offset EVM to ensure reliable communications. Other vendors may have similar requirements. Control targets 15% offset EVM as the threshold for expected performance, which is more stringent than the ZigBee specification requires. Offset EVM above 15% will typically only be encountered in older external LNA/PA designs, and should not be an issue for newer silicon and LNA/PA designs.  

Validation:

Validation for offset EVM requires special hardware, test equipment, and environment. Usually an electrical engineer will be involved in this testing. The general requirements are as follows:

1. Obtain or modify hardware for coax connector. If device under test already has a coax connector, this step can be skipped. 
2. Obtain and load a manufacturing test image on the device under test, or enter into manufacturing test mode if this function is already provided existing firmware. 
3. Connect RF analyzer via coax connection (Control4 uses Agilent analyzer). 
4. Using manufacturer specific method for generating a transmit stream, measure offset EVM. Values above 15% offset EVM indicate a transmit problem that should be addressed in hardware.


**Notes:** 
The failure mode introduced by high offset EVM can be difficult to diagnose. The typical failure mode will be excessive retries at the 802.15.4 MAC layer to deliver messages and obtain associated MAC acks. This is usually unidirectional in nature, since only one side is having a transmit problem, and it can appear that the receiver is having a problem. It is not possible to directly see high offset EVM in a sniffer, but there are some indicators. Lower than expected LQI and cost in link status exchanges is one key indicator. Both transmit and receive LQI are reflected in link status exchange cost. The indicator would not be from the node having transmit problems, but instead from a neighboring node reflecting how well it is receiving messages from this device. Another indicator is that a sniffer may not "hear" the transmitting node, even in close proximity. This combined with excessive MAC layer retries to a particular node (indicating the MAC acks are also not being received by the transmitting node) can be an indicator of high offset EVM. Control4 has found some sniffers are able to receive packets with higher offset EVM than others. Sniffing from two independent sniffers, one based upon AVR/2420 architecture for example, and one based upon Em250/35x, can provide more evidence. The AVR sniffer will decode the packets whereas the Em250/35x sniffer will not. Note that this is not sufficient to identify high offset EVM. Other factors, like interrupt handling implementation in firmware, could also produce a failure to MAC ack incoming messages. This is why an EVM analysis is required to measure offset EVM value of a particular device. Also be aware that other changes can introduce (or resolve) EVM issues over the life of a product. For instance, a minor change in hardware (e.g. changing a capacitor), extreme temperature changes triggering radio recalibration, and transmit power settings clipping the PA of a transmitter have all been found to generate offset EVM in excess of 15%. Regular testing and diligence on the part of hardware manufacturers and firmware developers is important to ensure EVM is within specifications.


## Support for Distributed Trust Center mode

The entity on a ZigBee network that handles security and key exchanges is called a Trust Center. There are two method of handling security: Centralized Trust Center (CTC) and Distribute Trust Center(DTC). The HA standard method prior to ZigBee 3.0 is CTC. This mode requires devices authenticate and receive keys from a central authority called the coordinator. If the central authority fails, the key information must be transferred to a new centralized trust center. It also requires that there is a coordinator on the network. The concept of coordinator can introduce some negative use cases. Nodes track coordinators differently that every other device in the network and cannot resolve conflicts with multiple coordinators automatically as they can with other nodes (e.g. node id conflict resolution).
Control4 uses DTC mode for all routers. This mode is also now part of the specification for ZigBee 3.0 devices. This allows association to any router within the system, and can remove the concept of coordinator. This has a couple of advantages, in that it eliminates a central point of failure for security management, and it makes address conflict resolution feasible in all use cases. It also allows any device to form a secure network. 

Validation:

1. Using a sniffer, confirm that the apsTrustCenterAddress is all 0xFFFFFFFFFFFFFFFF (e.g. Transport Key (NWK)→ZigBee Application Support Command→Source Address = 0xFFFFFFFFFFFFFFFF), indicating a Distributed Security network per docs-05-3474-21-0csg-zigbee-specification.pdf section . If the apsTrustCenterAddress is any other value, it indicates a Centralized Security network.

			 
## Uses Control4 compatible stack configuration and SAS attribute set

The following is a stack configuration, and therefore part of the compiled firmware image. These values are must be confirmed with the firmware developer for each vendor specific product. Hops, poll interval, and child table count apply only to routing devices. 

Validation:

Confirm with the vendor the device under test is configured with the following SAS and attribute set.

1. Stack profile = 2 (Pro)
2. Protocol version = 2 (rev 17 (2007) or later)
3. Security level = 5 (secure)
4. Maximum hops = 10 **(Need test method)**
5. End-device poll timeout and timeout shift of \>= 320 seconds **(Need test method)**

- end device poll timeout = 255
- end device poll timeout shift \>= 6

6. End-device child count \> 6 
7. Child table timeout


## Implements access point tracking algorithm for 3 or more access points

For routing devices to track multiple ZigBee access points, they must implement a Control4 specific algorithm. Zserver is designed to round robin through Zaps, sending an MTORR from each Zap in sequential fashion. It is expected a device receiving these messages will prioritize Zaps based upon path cost to the sender of the MTORR (also called the concentrator in Many-To-One routing). The "best" access point will be the access point with the lowest path cost. Path cost is proportionality to intermediate hop cost of the route traversal of the MTORR message. Poor cost can be simulated by introducing a poor hop, such as through transmitter attenuation. As attenuation increases, path cost will also increase. As path cost may change over time, it is expected that the routing device will automatically transition to a new access point if it becomes a better destination. It is also expected that the device under test will not simply follow the last access point to send an MTORR. This would indicate a firmware implementation that does not track a best access point. Zserver selects source routes based upon the route record of any message received from a device that is not an announcement. This route should be the route for the "best" access point (whereas the announcement route is simply the route to the last access point to send an MTORR). APS ack routes apply, so simply sending a ping to the device can trigger an APS route back to a "best" access point. The best indicator, however, is an asynchronous message (e.g. a trap), since this must follow a path defined by an address table entry on the device under test.

Validation:

1. Setup a mesh network with 3 Zaps and the routing device under test.
2. Allow the mesh to proceed through one cycle of MTORR's being sent from each Zap.
3. Send an MTORR out each Zap under normal conditions, with roughly equal path cost. 
4. Find the Zap currently being used as the best access point for the device under test. Ensure the device under test selects one, and uses this access point for asynchronous messages after receiving MTORR's from other access points. On a backchannel device, query the device. On a device with no backchannel, send asynchronous unicast (e.g. a trap) from the device by generating an event. 
5. Attenuate the signal on 2 of the 3 Zaps (e.g. remove the antennas).
6. Allow the round robin interval to proceed across all Zaps (configurable in zserver.conf, default 15 minutes for 3 Zaps).
7. Find the Zap currently being used as the best access point for the device under test. On a backchannel device, query the device. On a device with no backchannel, send an asynchronous unicast (e.g. a trap) from the device by generating an event. This should match the Zap with a normal signal level (e.g the one with an antenna)
8. Repeat steps 3-5 with the other Zaps. Ensure the device under test best access point is following the Zap that has normal transmit capability. 


## ZigBee Certification Test Houses

**United States: National Technical Systems Inc.**

Contact:

Mr. Raymond Chung

ph: +1 (310) 641-7700 ext: 1056

e: raymond.chung@nts.com


**Europe: TÜV Rheinland Group**

Contact:

Mr. Henk Veldhuis

ph: +32 427 310863 

e: henk.veldhuis@de.tuv.com





## Certification Contact for Control4

**Control4**

Contact:

Quyen Dungan

11734 S. Election Road

Draper, UT 84020-6432

ph: +1 (801)-523-4223

e: qdungan@control4.com


[1]:	https://control4.github.io/docs-zigbee/#control4-zigbee-cluster-for-end-devices