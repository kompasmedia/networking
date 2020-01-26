# Ansible role to configure networking on FreeBSD

This role allows you to configure IPv4 and IPv6 networking on FreeBSD.

## Interface

This role exposes the following variables for your use.

| Variable | Default value | Function |
|----------|---------------|----------|
| networking_interfaces | undefined | List of network interfaces to configure. |
| networking_ipv4_gateway | undefined | IPv4 address of the machine's default gateway in dotted-quad notation. |

## Configuring a network interface

The **networking_interfaces** variable contains a list of interface definitions this role cycles
through and sets up. An example would be:

```
networking_ipv4_gateway: 10.0.0.1
networking_interfaces:
- iface: em0
  ipv4_address: 10.0.0.189
  ipv4_netmask: 255.255.255.0
```

This sets up the **em0** interface with a static IPv4-address of 10.0.0.189 and netmask 255.255.255.0.
The table below lists the variables you can use in your interface definitions. Required values are
shown ing **bold** typeface.

| Variable | Function |
|----------|----------|
| **iface**    | The name by which the operating system knows this particular interface. |
| ipv4_address | IPv4 address in dotted-quad notation. |
| ipv4_netmask | IPv4 netmask in dotted-quad notation. |
