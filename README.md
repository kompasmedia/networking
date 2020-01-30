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

## More on network_autorestart and VLAN's

Permitting Ansible to automatically restart networking services can be both a good and a bad idea
depending on your situation. If you use Ansible in a chain of events and the playbook does not come
last, your process may experience serious glitches if the networking on the target hosts gets quite
rudely interrupted and modified mid-run. That's why **network_autorestart** is set to false by default.

If you're using this role in a more simple environment and you know that resetting the network on your
machines won't affect you, then you should set **network_autorestart** to true so Ansible can take care
of reconfiguring the networking according to the requested state

Configuring VLAN's is a bit of a hairy exception to the above. In order to use VLAN's, FreeBSD creates
a whole new virtual network interface. This new interface behaves exactly like a regular one and you're
free to configure it accordingly. However, upon the first run of this role the VLAN interfaces will not
be present when Ansible hits the configuration directives to set them up. This will lead to errors, so
we force-restart networking whenever changes to VLAN configuration are made.

The above condition can not be postponed over overruled and will **not** honor **network_autorestart**. The
mechanism used is *flush_handlers*, which will immediately execute any and all queued Ansible handlers that
are still outstanding in the current playbook. In order to prevent ugly surprises, place this role at the
very top of your playbook.

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
setting **networking_ipv6_gateway** in combination with SLAAC is legal as far as Anible is concerned, but I really
wouldn't do that because it'll probably conflict with what SLAAC provides or at the very least not be very useful.
