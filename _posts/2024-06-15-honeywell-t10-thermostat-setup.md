## Make It Work: Setting Up a Honeywell T10+ Smart Thermostat

I don't like IoT, but sometimes the benefits outweigh the risks...if it's done correctly. 

The risk of connecting this thing to the Internet are:
- Risk of breach if Resideo is breached, since it establishes a tunnel between them and the device.
- Risk of extreme damage to personal property if an attacker is able to bypass temperature bounds. Think frozen pipes, electrical fires, equipment failure, high utility bills, etc.

The risk of not connecting to the Internet:
- You don't get firmware updates. Resideo does not have a public change log to see when new firmware is available (not that they release it often anyway)

This guide is assuming that you want full functionality. Personally I don't need to look at the furnace thermostat to know the weather forecast.

### Setup 

#### Physical Location
- Try to install the main thermostat in a central location that will get the best average temperature for the first floor.
- If running new thermostat wiring, make sure to use 18/8 and not 18/2. Even though it will only use two wires, good to future-proof in case Honeywell gets dropped like it's hot.
- Ensure that the outdoor temperature sensor is in a spot where it will have minimal direct contact with the sun or wind.

#### Network

- Since this is IoT, it's recommended to create a new VLAN and segment it from anything else on the network. If there aren't enough SSIDs available on the access points, then ensure that client isolation is enforced so that all traffic goes to the gateway.
- Ensure that DHCP services are enabled on the VLAN and that there's a static reservation for the thermostat.

#### Firewall Rules

##### DNS

Create a rule to the desired DNS server on UDP 53. The thermostat will take direction from DHCP.

##### Web Services

Create a firewall rule permitting the thermostat to access the following hosts on TCP 443 with the SYN flag:
- weather02.clouddevice.io (observed)
- provprod02.clouddevice.io (observed)
- fwuprod02.clouddevice.io (observed)

##### AMQP over TLS

Create a firewall rule permitting the thermostat to access the following hosts on TCP 5671 with the SYN flag:
- lcc-prodsf-lcc03sf-iothub.azure-devices.net

### Homekit/HomeAssistant Integration

If desired, create two firewall rules permitting the thermostat to communicate with Homekit/HomeAssistant using multicast:
- Protocol UDP, destination IP address 224.0.0.251, port 5353
- Protocol IGMP, destination IP address 224.0.0.251

*Note that a relay will be necessary if the automation appliance is on another subnet (which it should be).*

### The Deets

After doing tons of research and weighing options and cost (and using my "contractor" accounts), I decided on the T10+ from Resideo. The problem is, *I hate IoT*. The security around the majority of these things is the equivalent of putting a glass window in a steel reinforced door. However, I need it for a number of home automation tasks that make things easier while traveling. Risk vs. reward.

On boot, the device starts with two LLC broadcasts:

![image](https://github.com/X8ETr1x/x8etr1x.github.io/assets/58610446/586e476f-ef8c-4d53-a8a2-7063ccc705a2)

The device MAC address identifier is registered to Resideo. The DHCP request demands the following parameters:

![image](https://github.com/X8ETr1x/x8etr1x.github.io/assets/58610446/07227f39-81e1-4c71-b621-f2f3669ab79a)

The device doesn't request an NTP server at all. There's also no firewall activity over UDP 123, so it looks like NTP is not used.

Vendor class identifier is Resideo.

![image](https://github.com/X8ETr1x/x8etr1x.github.io/assets/58610446/c2f534c4-65f2-4333-8be5-992b5d531643)

Interesting that the device is requesting an infinite lease in option 51. In terms of dynamic leases, this is a **very bad idea as it can exhaust the address pool.** I'm using a static lease and my DHCP server refuses it, so the risk is mitigated, but *shame on you Resideo.*

![image](https://github.com/X8ETr1x/x8etr1x.github.io/assets/58610446/44ca67cb-2821-488b-96b3-e6c9221e8797)

The device then attempts to discover IPv6 routes.

![image](https://github.com/X8ETr1x/x8etr1x.github.io/assets/58610446/193dbb64-3c99-4ce9-a1de-e3c9dd0e53d2)

Followed by numerous attempts at MDNS queries over both IPv4 and IPv6:

![image](https://github.com/X8ETr1x/x8etr1x.github.io/assets/58610446/550c8ea1-2eb5-485e-9b25-586f76103c43)

Now that the pleasentries are out of the way, time to start connecting to Resideo. The first connection is to Resideo's device provisioning services using HTTPS. The cipher quites are unremarkable.

![image](https://github.com/X8ETr1x/x8etr1x.github.io/assets/58610446/fa751520-0391-44c3-9877-eda1b68b3967)

Now this is interesting. The SNI in the client hello states "pki.resideo" ...

![image](https://github.com/X8ETr1x/x8etr1x.github.io/assets/58610446/d52013c1-bb7e-46b7-b5f4-8f92f5ea2d86)

The server provides certificates and picks its cipher suite:

![image](https://github.com/X8ETr1x/x8etr1x.github.io/assets/58610446/db3ac3c8-b71f-4b17-a55c-70ca7cf25ddd)

![image](https://github.com/X8ETr1x/x8etr1x.github.io/assets/58610446/b670fb23-a291-4420-9615-14580b295f14)

![image](https://github.com/X8ETr1x/x8etr1x.github.io/assets/58610446/8dcb2f8b-cfbf-4457-b0c3-b772f26cc5f5)

![image](https://github.com/X8ETr1x/x8etr1x.github.io/assets/58610446/613bd277-7e21-407d-a928-954d18610ed3)

The server does request a certificate from the client, so authentication is two-way:

![image](https://github.com/X8ETr1x/x8etr1x.github.io/assets/58610446/ef87bc9a-b527-4da1-8726-7b322bd2b7ed)

![image](https://github.com/X8ETr1x/x8etr1x.github.io/assets/58610446/4fa48034-ad17-4bc5-8871-e0517afdf77d)

Subsequent TLS connections follow the same pattern and handshake, including the AMQPS connection. I'll have to dig into it further at some point, but for now I have a million other things to do. Exactly one million. No more, no less.

### Sources
- https://buildings.honeywell.com/content/dam/hbtbt/en/documents/downloads/Pro-Watch_CIP%20V7%20Letter%202022_05.pdf
- https://prod-edam.honeywell.com/content/dam/honeywell-edam/hbt/en-us/documents/manuals-and-guides/reference-guides/31-00131.pdf
- https://prod-edam.honeywell.com/content/dam/honeywell-edam/hbt/en-us/documents/manuals-and-guides/user-manuals/HBT-BMS-31-00452M-SaMBa-Cybersecurity-Manual.pdf
