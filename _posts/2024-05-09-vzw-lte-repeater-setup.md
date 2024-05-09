## Setting Up a Verizon Wireless LTE Repeater

It never fails that anywhere I've lived has had cellular service that is as strong as my will to paint my basement, so I've always had one of these fantastically reliable Verizon LTE repeaters kicking around. There's no great single resource for setting one of these up, so here's how you do it.

Yes, I know Wi-Fi calling is a thing. I like redundancy.

### Setup 

#### Physical Location
- Install the GPS antenna as high up as you can outside to ensure the best signal.
- Try to install the repeater as centrally as you can in the areas.
- These devices aren't PoE. I mean, IK guess why would they be. That would only be convenient. Make sure you have a 120 volt receptacle handy.

#### Network

*One note is that the appliance will not accept traffic from a VLAN other than its own.*

- Since this is a Verizon appliance, you'll want to create a new VLAN an segment it from anything else on your network.
- Ensure that you have DHCP services enabled. You can set a static IP after the fact if you want to.

#### Firewall Rules

##### DNS

Create a rule to your desired DNS server on UDP 53. The appliance will take direction from DHCP.

##### NTP

The appliance will ignore DHCP NTP settings, so configure a NAT rule to point to an internal NTP server if desired. Alternatively, permit it to the following hosts:

- 0.north-america.pool.ntp.org
- 1.north-america.pool.ntp.org

Create a firewall rule to the NTP server(s) on UDP 123.

##### GPS

*Verizon states that this traffic is over HTTP, but network captures beg to differ.*

The GPS servers are on Amazon, so using IP ranges won't work. Permit the appliance to the following hosts on TCP 443 with the SYN flag:
  - xtrapath1.izatcloud.net
  - xtrapath2.izatcloud.net
  - xtrapath3.izatcloud.net

##### Certificate Services

*This used to be needed, though recently I am no longer seeing this traffic.*

Create a firewall rule permitting the appliance to access the following hosts on TCP 80 with the SYN flag:
- scep.gtm.securitycredentialing.com (observed)
- gtm.securitycredentialing.com (observed)
- cmp.securitycredentialing.com (documented, not observed)

##### IPSec VPN

The VPN connection to Verizon uses IPSec. A firewall rule will be needed to allow the appliance to one of the two below destination hosts. Connectivity will be via UDP 500 (ISAKMP) and UDP 4500 (IPSec NAT-T).

- sgw.vzwfemto.com (CNAME)
- sgw.gtm.vzwfemto.com (observed, undocumented)
- sgw-rcmdva83.vzwfemto.com
- sgw-chntvaav.vzwfemto.com
- sgw-wsbomagj.vzwfemto.com
- sgw-ynkrnyzl.vzwfemto.com
- sgw-wmtppaaa.vzwfemto.com
- sgw-esyrnyen.vzwfemto.com
- sgw-aurscoty.vzwfemto.com
- sgw-temqazkw.vzwfemto.com
- sgw-snvacanx.vzwfemto.com
- sgw-rdmewa22.vzwfemto.com
- sgw-azusca21.vzwfemto.com
- sgw-elsstx13.vzwfemto.com
- sgw-hsnotx08.vzwfemto.com
- sgw-tulyok13.vzwfemto.com
- sgw-chrxnclh.vzwfemto.com
- sgw-orlhfl01.vzwfemto.com
- sgw-alprgagq.vzwfemto.com
- sgw-omalnexu.vzwfemto.com
- sgw-hchlilmt.vzwfemto.com
- sgw-lenykscj.vzwfemto.com
- sgw-jhtwpadp.vzwfemto.com
- sgw-sfldmilr.vzwfemto.com

*Resolution will be decided by geolocation.*

### Sources
- https://www.verizon.com/support/pdf/user_guide/verizon-4g-lte-network-extender3-enterprise-v81-user-guide.pdf
