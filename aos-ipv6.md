# aos-ipv6

This document details the various methods available to allow an Aruba Access Point to boot up and communicate with a Controller in an IPv6 only environment.

## Aruba Access Point Boot Procedure

Aruba Access Points that are managed by a Controller go through three phases in an attempt to make contact with the Controller. This provides a robust provisioning process and flexibility of campus network design. These methods are utilising whether the AP is in an IPv4, an IPv6 or a dual-stack environment.

1. DHCP - attempt to contact a DHCP server which can provide the controller address via Option 52 or Option ....
2. DNS - Transmit a DNS query for 
3. A??? - multicast on the local LAN using this protocol.

## IPv6 Specific Design and Considerations

With IPv4 the methods of address allocation are either statically assigned or via DHCPv4.
With IPv6, address allocation functionality is incorporated into the protocol itself, thus expanding the methods in which a client can receive an address:

1. Static - user configured IPv6 prefix. Note that multiple IPv6 prefixes can be configured per interface.

2. Stateless Address AutoConfiguration SLAAC - a client can create its own globally routable address without the need to deploy a DHCPv6 server.
The local IPv6 gateway transmits a Router Advertisement packet that contains one or more /64 prefixes with the RA flag 'Autonomous' flag, or the A flag, ON. This tells the client to create its own address using the /64 from the RA as the first 64-bits, or Network ID, of the address and generating the latter 64-bits, or the Host ID, itself. This functionlity is active on all major operating systems and the Host ID generation method is....
In modern networks, clients often require more than just an IP address. Information such as DNS servers are vital to basic operation. This data can be provided to a client using DHCPv4/DHCPv6 Options (see below), but in a SLAAC environment there is no DHCP server. To address this, the IETF produced RFC6106 'IPv6 Router Advertisement Options for DNS Configuration' which detailed using Router Advertisements to communicate the Recursive DNS Server (RNDSS) and DNS
Search-List (DNSSL) to SLAAC clients. Please note that RFC8106 obsoletes RFC6106 but, at present, vendor datasheets and OS specifications still quote RFC6106 to indicate the said functionality.

3. Stateful DHCPv6 - Upon boot the client will either attempt to contact a DHCPv6 server with a DHCPv6 SOLICIT packet (Windows10) or the local gateway will tell the client to use DHCPv6 via the Router Advertisement packet. The RA will have the Managed flag, or the M flag, set to ON. Upon receipt the client will transmit DHCPv6 SOLICIT messages.
If the DHCPv6 server is on a different subnet, the local gateway should be configured with a DHCPv6 helper address pointing to the DHCPv6 server.
To summarize a successful DHCPv6 exchange, the server will issue an address to the requesting client which the client will use as its globally routable address.
Please note that the DHCPv6 server can also issue DHCPv6 Options in addition to the address. The relevant options in this environment as as follows:
23???
24???
52???
Also note that DHCPv6 and SLAAC are not mutually exclusive on a subnet. The local gateway can issue a Router Advertisment with the M flag set and one or more prefixes with the A flag ON. By default, clients will auto-generate addresses based on the received A-flag enabled prefixes and still attempt to solicit an addres from a DHCPv6 server.

4. Stateless DHCPv6 This method combines SLAAC for address generation and DHCPv6 for the additional information found in DHCPv6 options.
The local gateway will issue one or more prefixes with the A flag set to the client to use to creeate their addresses, but will also set the Other flag, or O flag, in the RA to ON. This tells the client to attempt to contact a DHCPv6 server for additional, non-address information.
The client transmits a DHCPv6 IOnformation-Request and the DHCPv6 server returns the configured Options, without an IPv6 address.

Here is a summary of the four address allocation methods:

## Aruba Network Design Options

The multiple controller discovery methods open to Aruba Access Points, combined with the varying IPv6 address allocation and configuration methods creates numerous permeatations for network design and those that have been lab-tested are detailed here.
Please note that the designs require a DHCP server, whether DHCPv6 or DHCPv4, see Option xxxxx SLAAC in combination with RFC8106 RDNSS & DNSSL alone is currently not supported.

1. Stateful DHCPv6 with Option xxxx RDNSS.
Setup:
Deploy a DHCPv6 server.
For Windows 2016/2019 the DHCPv6 configuration:
DHCP > 

Sample configuration for ISC DHCPv6 Linux-based server:

Please note that when an Aruba AP boots it will begin transmitting DHCPv6 SOLICITS and does not need to receive a Router Advertisement with the M flag ON.
However, the gateway should not suppress RAs because the AP, and other local clients require the local prefix to determine that they are ON-LINK.
If the DHCPv6 server is on a different subnet to the APs, remember to configure the 'ipv6 helper-address' command on the local gateway.






