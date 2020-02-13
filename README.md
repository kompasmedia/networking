# Ansible role to configure networking on FreeBSD

This role allows you to configure IPv4 and IPv6 networking on FreeBSD.

## Interface

This role exposes the following variables for your use.

| Variable | Default value | Function |
|----------|---------------|----------|
| networking_autorestart| False | Permit Ansible to restart networking at the end of a playbook run to effectuate changes. |
| networking_dhcp_type | DHCP | Set to SYNCDHCP to block the boot process while waiting for a DHCP lease. |
| networking_dns_resolvers| undefined | List of IP-addresses (IPv4 and/or IPv6) of resolving DNS servers. |
| networking_dns_search | undefined | Name of the local domain (usually), where the resolver will look for unqualified hostnames. |
| networking_epairs | undefined | Number of interfaces of class *epair* to create. |
| networking_gateway_dhcp | True | Use default gateway from DHCP. |
| networking_hostname | undefined | Hostname of the machine without the domain part. |
| networking_interfaces | undefined | List of network interfaces to configure. |
| networking_ipv4_gateway | undefined | IPv4 address of the machine's default gateway in dotted-quad notation. |
| networking_ipv6_gateway | undefined | IPv6 address of the machine's default gateway. |
| networking_resolvers_dhcp | True | Use DNS Resolvers from DHCP(v6) instead of Ansible-configured values. |

## Configuring a network interface

The **networking_interfaces** variable contains a list of interface definitions this role cycles
through and sets up. An example would be:

```
networking_hostname: bsdbox
networking_ipv4_gateway: 10.0.0.1
networking_interfaces:
- iface: em0
  ipv4_address: 10.0.0.189
  ipv4_netmask: 255.255.255.0
```

This sets up the **em0** interface with a static IPv4-address of 10.0.0.189 and netmask 255.255.255.0.
The table below lists the variables you can use in your interface definitions. Required values are
shown in **bold** typeface.

| Variable | Default value | Function |
|----------|---------------|----------|
| **iface**    | undefined | The name by which the operating system knows this particular interface. |
| ipv4_address | undefined | IPv4 address in dotted-quad notation. |
| ipv6_address | undefined | IPv6 address |
| ipv4_dhcp | False | Automatically configure IPv4 using DHCP. |
| ipv4_netmask | undefined | IPv4 netmask in dotted-quad notation. |
| ipv6_prefixlen | undefined | IPv6 prefix length, usually 64. |
| ipv6_slaac | False | Automatically configure IPv6 using SLAAC. |
| vlans | undefined | List of VLAN numbers from 1 to 4096. |

## More on network_autorestart

Permitting Ansible to automatically restart networking services can be both a good and a bad idea
depending on your situation. If you use Ansible in a chain of events and the playbook does not come
last, your process may experience serious glitches if the networking on the target hosts gets quite
rudely interrupted and modified mid-run. That's why **network_autorestart** is set to false by default.

If you're using this role in a more simple environment and you know that resetting the network on your
machines won't affect you, then you should set **network_autorestart** to true so Ansible can take care
of reconfiguring the networking according to the requested state.

## Bridges

A bridge in FreeBSD behaves quite like a virtualized unmanaged layer-2 switch. It forwards traffic to interfaces
that are connected to it, so-called *members* of the bridge. It will even learn attached interfaces' MAC addresses
like a real switch and knows how to deal with spanning tree protocols. This is a very useful type of interface
to have available when working with virtualization and/or jails.

Bridges are defined just like any other network interface except for the fact that bridges have members. Member
interfaces network interfaces that will be connected to and participating on the bridge. Note that the reverse
also applies. If an interface has no members, Ansible will not treat it as a bridge.

Bridges themselves can have IP configuration applied, which allows you to set them up as gateways/routers for
member interfaces. The example below creates a bridge between *em0* and *em1* and assigns a static IPv4 address
to the bridge itself.

```
networking_interfaces:
- iface: bridge0
  ipv4_address: "10.30.0.1"
  ipv4_netmask: "255.255.255.0"
  members: ['em0','em1']
```

Bridges are also a great way to implement transparent firewalling. You can set up filtering rules on a bridge
while not assigning an address to the bridge itself. Packets passing through the bridge will still get filtered
but attackers will have no way to touch the construct where the filtering is actually happening.

## DHCP and the IPv4 default gateway

This role allows you to set the IPv4 default gateway through **networking_ipv4_gateway** but you're also
quite likely to receive a gateway through DHCP. These two may very well clash, or you have some particular
override scenario in mind where your default gateway lives on a statically configured network.

For this reason you have **networking_gateway_dhcp** available. When you set this to *true*, Ansible will
actively remove any statically set default gateway from the system and ignore the **networking_ipv4_gateway**
variable if present, which really shouldn't be the case at all to begin with.

If you want to use a static IPv4 gateway while still using DHCP on one or more interfaces, set **networking_gateway_dhcp**
to false. This has no impact on what the DHCP client sees or does with what it gets from the server, but it does
keep the statically configured gateway from Ansible in place.

## FreeBSD epair(4) interfaces

In a setup with VNET jails on a host, it's common practice to use a single bridge interface as the gateway
in an RFC1918 subnet while each invidual jail is outfitted with *epair* network interfaces. An *epair* is a
special kind of virtual network interface in FreeBSD which connects two points as if there were a virtual cable
in place. In this case one end is made a member of the bridge while the other is attached to the VNET jail's
local networking stack. In this way the host system can behave as NAT gateway between the bridge and the
host machine's own physical uplink.

The *epair* interface is created by *interface cloning* and is a bit of a special case because upon creation of
a single *epair*, you will end up with two new interfaces on your machine. Because you'll usually need a number of
epairs in sequence, the way we create epair interfaces through Ansible is by passing the number of them we need:

```
networking_epairs: 10
```

This will instruct Ansible to generate 10 interface-pairs which will be named *epair0a* and *epair0b* all the way
through *epair0b* and *epair9b*. Each of these represents the end of a virtual 'cable', so to speak and can be
configured like any other regular network interface by adding it to **networking_interfaces**.

## DHCP and the DNS resolvers

Similar to how the default gateway can come from multiple sources, DNS resolvers can be pushed from elsewhere.
In this case we're looking at:

- Static configuration through Ansible
- DHCP on an IPv4 interface
- DHCPv6 on an IPv6 interface

This role will step out of the way once you set **networking_resolvers_dhcp** to true, which is indeed the
default behavior, similar to what **networking_gateway_dhcp** does. The reason why we're allowing ourselves to be
overruled by DHCP(v6) is because the owner of the local network, who took the effort to set up a DHCP server,
is likely to know best which DNS resolvers should actually be used.

Of course, if you disagree and somehow must push alternative DNS resolvers, you can do so through Ansible. Just
set **networking_resolvers_dhcp** to false and configure them manually.

```
networking_dns_search: "lan.acme.com"
networking_dns_resolvers:
- "8.8.8.8"
- "8.8.8.4"
```

## IPv6 configuration

IPv6 has more degrees of freedom than this Ansible role can realistically cater for. For that reason, the variables
behave in a very simple manner and you are responsible for setting up sensible combinations. For example,
setting **networking_ipv6_gateway** in combination with SLAAC is legal as far as Ansible is concerned, but you really
shouldn't do that because it'll probably conflict with what SLAAC provides or at the very least not be very useful.
