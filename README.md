# Ansible role to configure networking on FreeBSD

This role allows you to configure IPv4 and IPv6 networking on FreeBSD.

## Interface

This role exposes the following variables for your use.

| Variable | Default value | Function |
|----------|---------------|----------|
| networking_autorestart| False | Permit Ansible to restart networking at the end of a playbook run to effectuate changes. |
| networking_hostname | undefined | Hostname of the machine without the domain part. |
| networking_interfaces | undefined | List of network interfaces to configure. |
| networking_ipv4_gateway | undefined | IPv4 address of the machine's default gateway in dotted-quad notation. |

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
| ipv4_dhcp | False | Automatically configure IPv4 using DHCP. |
| ipv4_netmask | undefined | IPv4 netmask in dotted-quad notation. |
| ipv6_slaac | False | Automatically configure IPv6 using SLAAC. |

## More on network_autorestart

Permitting Ansible to automatically restart networking services can be both a good and a bad idea
depending on your situation. If you use Ansible in a chain of events and the playbook does not come
last, your process may experience serious glitches if the networking on the target hosts gets quite
rudely interrupted and modified mid-run. That's why **network_autorestart** is set to false by default.

If you're using this role in a more simple environment and you know that resetting the network on your
machines won't affect you, then you should set **network_autorestart** to true to avoid unnecessary
reboots.

