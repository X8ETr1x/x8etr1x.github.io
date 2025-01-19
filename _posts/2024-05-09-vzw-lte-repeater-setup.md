## Make It Work: Setting Up a Verizon Wireless LTE Repeater

It never fails that anywhere I've lived has had cellular service that is as strong as my will to paint my basement, so I've always had one of these *fantastically reliable* Verizon LTE repeaters kicking around. There's no great single resource for setting one of these up, so here's how you do it.

Yes, I know Wi-Fi calling is a thing. I like redundancy.

### Setup 

#### Physical Location
- Install the GPS antenna as high up as possible outside to ensure the best signal.
- Try to install the repeater as centrally as possible in the desired service areas.
- These devices aren't PoE. I mean, I guess why would they be. That would only be convenient. Make sure there's a 120 volt receptacle handy.

#### Network

*One note is that the appliance will not accept traffic from a VLAN other than its own.*

- Since this is a Verizon appliance, it's recommended to create a new VLAN and segment it from anything else on the network.
- Ensure that DHCP services are enabled on the VLAN. A static IP can be set after the fact if desired.

#### Firewall Rules

##### DNS

Create a rule to the desired DNS server on UDP 53. The appliance will take direction from DHCP.

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

### The Deets

I have the LTEv2 repeater, though it's version 2 of...version 3? Who's on first?

The device MAC address indicates that the manufacturer is Askey Computer, which is a subsidiary of ASUS. The DHCP request demands the following parameters:

![image](https://github.com/X8ETr1x/x8etr1x.github.io/assets/58610446/c1b3b981-5a46-4363-bb6c-843a97ab37b0)

Which is interesting that NTP servers are requested, since the device simply ignores the NTP server configured provided in the offer.

![image](https://github.com/X8ETr1x/x8etr1x.github.io/assets/58610446/2bae8300-9181-491a-a995-19bb8173b46f)

We see here it's running udhcp 1.24.2. 

NTP was unremarkable.

The device offers TLSv1.2 when initiating the TLS handshake:

>                 Cipher Suite: TLS_ECDHE_RSA_WITH_AES_256_GCM_SHA384 (0xc030)
>                 Cipher Suite: TLS_ECDHE_ECDSA_WITH_AES_256_GCM_SHA384 (0xc02c)
>                 Cipher Suite: TLS_ECDHE_RSA_WITH_AES_256_CBC_SHA384 (0xc028)
>                 Cipher Suite: TLS_ECDHE_ECDSA_WITH_AES_256_CBC_SHA384 (0xc024)
>                 Cipher Suite: TLS_ECDHE_RSA_WITH_AES_256_CBC_SHA (0xc014)
>                 Cipher Suite: TLS_ECDHE_ECDSA_WITH_AES_256_CBC_SHA (0xc00a)
>                 Cipher Suite: TLS_DH_DSS_WITH_AES_256_GCM_SHA384 (0x00a5)
>                 Cipher Suite: TLS_DHE_DSS_WITH_AES_256_GCM_SHA384 (0x00a3)
>                 Cipher Suite: TLS_DH_RSA_WITH_AES_256_GCM_SHA384 (0x00a1)
>                 Cipher Suite: TLS_DHE_RSA_WITH_AES_256_GCM_SHA384 (0x009f)
>                 Cipher Suite: TLS_DHE_RSA_WITH_AES_256_CBC_SHA256 (0x006b)
>                 Cipher Suite: TLS_DHE_DSS_WITH_AES_256_CBC_SHA256 (0x006a)
>                 Cipher Suite: TLS_DH_RSA_WITH_AES_256_CBC_SHA256 (0x0069)
>                 Cipher Suite: TLS_DH_DSS_WITH_AES_256_CBC_SHA256 (0x0068)
>                 Cipher Suite: TLS_DHE_RSA_WITH_AES_256_CBC_SHA (0x0039)
>                 Cipher Suite: TLS_DHE_DSS_WITH_AES_256_CBC_SHA (0x0038)
>                 Cipher Suite: TLS_DH_RSA_WITH_AES_256_CBC_SHA (0x0037)
>                 Cipher Suite: TLS_DH_DSS_WITH_AES_256_CBC_SHA (0x0036)
>                 Cipher Suite: TLS_DHE_RSA_WITH_CAMELLIA_256_CBC_SHA (0x0088)
>                 Cipher Suite: TLS_DHE_DSS_WITH_CAMELLIA_256_CBC_SHA (0x0087)
>                 Cipher Suite: TLS_DH_RSA_WITH_CAMELLIA_256_CBC_SHA (0x0086)
>                 Cipher Suite: TLS_DH_DSS_WITH_CAMELLIA_256_CBC_SHA (0x0085)
>                 Cipher Suite: TLS_ECDH_RSA_WITH_AES_256_GCM_SHA384 (0xc032)
>                 Cipher Suite: TLS_ECDH_ECDSA_WITH_AES_256_GCM_SHA384 (0xc02e)
>                 Cipher Suite: TLS_ECDH_RSA_WITH_AES_256_CBC_SHA384 (0xc02a)
>                 Cipher Suite: TLS_ECDH_ECDSA_WITH_AES_256_CBC_SHA384 (0xc026)
>                 Cipher Suite: TLS_ECDH_RSA_WITH_AES_256_CBC_SHA (0xc00f)
>                 Cipher Suite: TLS_ECDH_ECDSA_WITH_AES_256_CBC_SHA (0xc005)
>                 Cipher Suite: TLS_RSA_WITH_AES_256_GCM_SHA384 (0x009d)
>                 Cipher Suite: TLS_RSA_WITH_AES_256_CBC_SHA256 (0x003d)
>                 Cipher Suite: TLS_RSA_WITH_AES_256_CBC_SHA (0x0035)
>                 Cipher Suite: TLS_RSA_WITH_CAMELLIA_256_CBC_SHA (0x0084)
>                 Cipher Suite: TLS_ECDHE_RSA_WITH_AES_128_GCM_SHA256 (0xc02f)
>                 Cipher Suite: TLS_ECDHE_ECDSA_WITH_AES_128_GCM_SHA256 (0xc02b)
>                 Cipher Suite: TLS_ECDHE_RSA_WITH_AES_128_CBC_SHA256 (0xc027)
>                 Cipher Suite: TLS_ECDHE_ECDSA_WITH_AES_128_CBC_SHA256 (0xc023)
>                 Cipher Suite: TLS_ECDHE_RSA_WITH_AES_128_CBC_SHA (0xc013)
>                 Cipher Suite: TLS_ECDHE_ECDSA_WITH_AES_128_CBC_SHA (0xc009)
>                 Cipher Suite: TLS_DH_DSS_WITH_AES_128_GCM_SHA256 (0x00a4)
>                 Cipher Suite: TLS_DHE_DSS_WITH_AES_128_GCM_SHA256 (0x00a2)
>                 Cipher Suite: TLS_DH_RSA_WITH_AES_128_GCM_SHA256 (0x00a0)
>                 Cipher Suite: TLS_DHE_RSA_WITH_AES_128_GCM_SHA256 (0x009e)
>                 Cipher Suite: TLS_DHE_RSA_WITH_AES_128_CBC_SHA256 (0x0067)
>                 Cipher Suite: TLS_DHE_DSS_WITH_AES_128_CBC_SHA256 (0x0040)
>                 Cipher Suite: TLS_DH_RSA_WITH_AES_128_CBC_SHA256 (0x003f)
>                 Cipher Suite: TLS_DH_DSS_WITH_AES_128_CBC_SHA256 (0x003e)
>                 Cipher Suite: TLS_DHE_RSA_WITH_AES_128_CBC_SHA (0x0033)
>                 Cipher Suite: TLS_DHE_DSS_WITH_AES_128_CBC_SHA (0x0032)
>                 Cipher Suite: TLS_DH_RSA_WITH_AES_128_CBC_SHA (0x0031)
>                 Cipher Suite: TLS_DH_DSS_WITH_AES_128_CBC_SHA (0x0030)
>                 Cipher Suite: TLS_DHE_RSA_WITH_SEED_CBC_SHA (0x009a)
>                 Cipher Suite: TLS_DHE_DSS_WITH_SEED_CBC_SHA (0x0099)
>                 Cipher Suite: TLS_DH_RSA_WITH_SEED_CBC_SHA (0x0098)
>                 Cipher Suite: TLS_DH_DSS_WITH_SEED_CBC_SHA (0x0097)
>                 Cipher Suite: TLS_DHE_RSA_WITH_CAMELLIA_128_CBC_SHA (0x0045)
>                 Cipher Suite: TLS_DHE_DSS_WITH_CAMELLIA_128_CBC_SHA (0x0044)
>                 Cipher Suite: TLS_DH_RSA_WITH_CAMELLIA_128_CBC_SHA (0x0043)
>                 Cipher Suite: TLS_DH_DSS_WITH_CAMELLIA_128_CBC_SHA (0x0042)
>                 Cipher Suite: TLS_ECDH_RSA_WITH_AES_128_GCM_SHA256 (0xc031)
>                 Cipher Suite: TLS_ECDH_ECDSA_WITH_AES_128_GCM_SHA256 (0xc02d)
>                 Cipher Suite: TLS_ECDH_RSA_WITH_AES_128_CBC_SHA256 (0xc029)
>                 Cipher Suite: TLS_ECDH_ECDSA_WITH_AES_128_CBC_SHA256 (0xc025)
>                 Cipher Suite: TLS_ECDH_RSA_WITH_AES_128_CBC_SHA (0xc00e)
>                 Cipher Suite: TLS_ECDH_ECDSA_WITH_AES_128_CBC_SHA (0xc004)
>                 Cipher Suite: TLS_RSA_WITH_AES_128_GCM_SHA256 (0x009c)
>                 Cipher Suite: TLS_RSA_WITH_AES_128_CBC_SHA256 (0x003c)
>                 Cipher Suite: TLS_RSA_WITH_AES_128_CBC_SHA (0x002f)
>                 Cipher Suite: TLS_RSA_WITH_SEED_CBC_SHA (0x0096)
>                 Cipher Suite: TLS_RSA_WITH_CAMELLIA_128_CBC_SHA (0x0041)
>                 Cipher Suite: TLS_RSA_WITH_IDEA_CBC_SHA (0x0007)
>                 Cipher Suite: TLS_ECDHE_RSA_WITH_3DES_EDE_CBC_SHA (0xc012)
>                 Cipher Suite: TLS_ECDHE_ECDSA_WITH_3DES_EDE_CBC_SHA (0xc008)
>                 Cipher Suite: TLS_DHE_RSA_WITH_3DES_EDE_CBC_SHA (0x0016)
>                 Cipher Suite: TLS_DHE_DSS_WITH_3DES_EDE_CBC_SHA (0x0013)
>                 Cipher Suite: TLS_DH_RSA_WITH_3DES_EDE_CBC_SHA (0x0010)
>                 Cipher Suite: TLS_DH_DSS_WITH_3DES_EDE_CBC_SHA (0x000d)
>                 Cipher Suite: TLS_ECDH_RSA_WITH_3DES_EDE_CBC_SHA (0xc00d)
>                 Cipher Suite: TLS_ECDH_ECDSA_WITH_3DES_EDE_CBC_SHA (0xc003)
>                 Cipher Suite: TLS_RSA_WITH_3DES_EDE_CBC_SHA (0x000a)
>                 Cipher Suite: TLS_EMPTY_RENEGOTIATION_INFO_SCSV (0x00ff)
>             Compression Methods Length: 1
>             Compression Methods (1 method)
>                 Compression Method: null (0)
>             Extensions Length: 307
>             Extension: server_name (len=28) name=xtrapath1.izatcloud.net
>                 Type: server_name (0)
>                 Length: 28
>                 Server Name Indication extension
>             Extension: ec_point_formats (len=4)
>                 Type: ec_point_formats (11)
>                 Length: 4
>                 EC point formats Length: 3
>                 Elliptic curves point formats (3)
>             Extension: supported_groups (len=28)
>                 Type: supported_groups (10)
>                 Length: 28
>                Supported Groups List Length: 26
>                 Supported Groups (13 groups)
>                     Supported Group: secp256r1 (0x0017)
>                     Supported Group: secp521r1 (0x0019)
>                     Supported Group: brainpoolP512r1 (0x001c)
>                     Supported Group: brainpoolP384r1 (0x001b)
>                     Supported Group: secp384r1 (0x0018)
>                     Supported Group: brainpoolP256r1 (0x001a)
>                     Supported Group: secp256k1 (0x0016)
>                     Supported Group: sect571r1 (0x000e)
>                     Supported Group: sect571k1 (0x000d)
>                     Supported Group: sect409k1 (0x000b)
>                     Supported Group: sect409r1 (0x000c)
>                     Supported Group: sect283k1 (0x0009)
>                     Supported Group: sect283r1 (0x000a)
>             Extension: signature_algorithms (len=32)
>                 Type: signature_algorithms (13)
>                 Length: 32
>                 Signature Hash Algorithms Length: 30
>                 Signature Hash Algorithms (15 algorithms)
>                     Signature Algorithm: rsa_pkcs1_sha512 (0x0601)
>                     Signature Algorithm: SHA512 DSA (0x0602)
>                     Signature Algorithm: ecdsa_secp521r1_sha512 (0x0603)
>                     Signature Algorithm: rsa_pkcs1_sha384 (0x0501)
>                     Signature Algorithm: SHA384 DSA (0x0502)
>                     Signature Algorithm: ecdsa_secp384r1_sha384 (0x0503)
>                     Signature Algorithm: rsa_pkcs1_sha256 (0x0401)
>                     Signature Algorithm: SHA256 DSA (0x0402)
>                     Signature Algorithm: ecdsa_secp256r1_sha256 (0x0403)
>                     Signature Algorithm: SHA224 RSA (0x0301)
>                     Signature Algorithm: SHA224 DSA (0x0302)
>                     Signature Algorithm: SHA224 ECDSA (0x0303)
>                     Signature Algorithm: rsa_pkcs1_sha1 (0x0201)
>                     Signature Algorithm: SHA1 DSA (0x0202)
>                     Signature Algorithm: ecdsa_sha1 (0x0203)
>             Extension: heartbeat (len=1)
>             Extension: next_protocol_negotiation (len=0)
>             Extension: application_layer_protocol_negotiation (len=11)
>                 Type: application_layer_protocol_negotiation (16)
>                 Length: 11
>                 ALPN Extension Length: 9
>                 ALPN Protocol
>                     ALPN string length: 8
>                     ALPN Next Protocol: http/1.1
>             Extension: padding (len=171)
>                 Type: padding (21)
>                 Length: 171
>                 Padding Data [truncated]: 000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000
>             [JA4: t12d8008h1_9cedc1f1428b_046e095b7c4a]
>             [JA4_r [truncated]: t12d8008h1_0007,000a,000d,0010,0013,0016,002f,0030,0031,0032,0033,0035,0036,0037,0038,0039,003c,003d,003e,003f,0040,0041,0042,0043,0044,0045,0067,0068,0069,006a,006b,0084,0085,0086,0087,0088,0096,0097,0098,0099,009a,009c]
>             [JA3 Fullstring [truncated]: 771,49200-49196-49192-49188-49172-49162-165-163-161-159-107-106-105-104-57-56-55-54-136-135-134-133-49202-49198-49194-49190-49167-49157-157-61-53-132-49199-49195-49191-49187-49171-49161-164-162-160-158-103-64-63]
 >            [JA3: 535aca3d99fc247509cd50933cd71d37]

The client does not offer a client certificate.

The device then proceeds to attempt to reach time.google.com, which gets NAT'd to not time.google.com.

![image](https://github.com/X8ETr1x/x8etr1x.github.io/assets/58610446/dc854550-40af-4691-b729-610e24b339f1)

If you like unnecessary IPv6 noise on your networks, then boy is this the right appliance for you.

![image](https://github.com/X8ETr1x/x8etr1x.github.io/assets/58610446/20121d8e-a1a6-49c8-b99d-a07549b4c330)

I notice numerous initial failures to establish the VPN tunnel with no request from the server, but eventually after a few failures it receives a response.

- The device initiates a request with IKEv2 using AES-CBC-128 for encryption, HMAC-SHA1-96 for integrity, HMAC-SHA1 for PRF, and Diffie-Hllman group 4.
- Key exchange uses MODP group 2.
- Fragmentation is advertised as supported.
- Signature algorithm is SHA-256.

The VPN tunnel is established and ESP is off to the races.

### Sources
- https://www.verizon.com/support/pdf/user_guide/verizon-4g-lte-network-extender3-enterprise-v81-user-guide.pdf
