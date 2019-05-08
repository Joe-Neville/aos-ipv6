# How to deploy a DHCPv6 server (Windows & Linux)

## Windows 2016 / 2019

### DHCP Server Installation

Start > Server Manager > Manage > Add Roles and Features
Proceed to the Server Roles screen and tick 'DHCP Server' > Add Features
Proceed to Confirmation then hit 'Install'
The DHCP Server Feature will then install, you can close that windows down and the process will run in the background.
Once complete there will be a warning in Server Manager for 'Post-deployment Configuration'. Hit the link 'Complete DHCP configuration'. You can proceed through the configuration with the default settings.

### DHCPv6 Configuration

Start > Windows Administrative Tools > DHCP
Expand your Server to reveal 'IPv4' and 'IPv6'


#### DHCPv6 Stateless 

Only DHCPv6 Options are returned to the client, right-click 'Server Options' > Configure Options

dhcpv6-stateless-server-options pic

Configure the required options such as Option 23 RDNSS, Option 24 DNSSL. If you wish to communicate the controller address to the AP using DHCPv6 Option 52 it can be enabled here. Ensure you have completed the steps to create this Option first. See below.

dhcpv6-stateless pic

#### DHCPv6 Stateful

Right-click IPv6 > New Scope
Complete the 'New Scope Wizard' to configure the Scope name and prefix, any excluded addresses and the lifetime.
Hit Finish to activate the Scope.

To add DHCPv6 Options, expand the Scope and right-click 'Scope Options' > Configure Options

dhcpv6-stateful-options pic

Configure the desired options, see DHCPv6 Stateless above for details.

#### Configure Option 52 - CAPWAP

Right-click 'IPv6' under your server in the DHCP window.

Select 'Set Predefined Options' > Add
The 'Option Type' window will open.
Name the option, such as 'CAPWAP'
For Data Type select IPv6 Address from the drop-down menu.
Tick the Array box.
Enter Code 52 then OK

capwap-pic

Activate this new Option under Scope Options, for DHCPv6 Stateful or Server Options for Stateless.

## Ubuntu Linux

Install the ISC DHCP server:

```
sudo apt update
sudo apt-get install isc-dhcp-server
```

### DHCPv6 Configuration

Edit the DHCP server configuration file in your preferred editor. It is good practice to make a copy of the original first.

sudo cp /etc/dhcp/dhcpd6.conf /etc/dhcp/dhcpd6.conf.old
sudo nano /etc/dhcp/dhcpd6.conf

The ISC DHCP server uses as a single configuration file. To enable DHCPv6 Stateful or Stateless edit this file as appropriate:

```
# DNS Server addresses
option dhcp6.name-servers 2001:db8:a:50::10;
# DNS Search ListServer
option dhcp6.domain-search "actium.home";
# IPv6 subnet and the range within that subnet to offer addresses from
subnet6 2001:db8:a:50::/64 {
        range6 2001:db8:a:50::200 2001:db8:a:50::299;
}
```

#### Configure Option 52 - CAPWAP

The DHCPv6 CAPWAP option is configured in the dhcp6.conf file.
First create the option then apply it to a subnet:

```
# create option 52 capwap-ac-v6
option dhcp6.capwap-ac-v6 code 52 = array of ip6-address;
# apply to a subnet
subnet6 2001:db8:a:50::/64 {
	range6 2001:db8:a:50::100 2001:db8:a:50::115;
    option dhcp6.capwap-ac-v6 2001:db8:a:50::30;
}
```
