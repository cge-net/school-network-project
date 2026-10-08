# Design decisions

## Layer 2

- **Collapsed core.** Two Catalyst 3650 switches act as core and distribution. Each access switch has one trunk to each core, so losing a core or an uplink does not isolate a building.
- **Rapid PVST+ with planned roots.** CORE_SW1 is root for VLANs 10, 20, 30 and secondary for 40, 50, 60. CORE_SW2 is the reverse. Access switches have higher priorities so they can never become root.
- **PAgP EtherChannel (Po1) between the cores,** trunking all six VLANs.
- **PortFast and BPDU guard on every edge port.** Clients get DHCP immediately after a link comes up, and a switch plugged into an access port is shut out.
- **Unused native VLAN (111) on inter-switch trunks,** and unused ports shut down in a parking VLAN (999).
- **VTP.** CORE_SW1 is the server, all other switches are clients.

## Gateway redundancy

- **HSRP version 2** on every VLAN interface, with the virtual address at `.1`, CORE_SW1 at `.2` and CORE_SW2 at `.3`.
- The active router has priority 110 and preemption, and matches the STP root for the same VLAN.

## Routing

- **Single-area OSPF** between the two cores and the border router over routed /30 links.
- **`passive-interface default`** on the cores. Only the border uplink and VLAN 10 form adjacencies. All VLAN subnets are still advertised.
- **Core-to-core adjacency** lets a core that loses its uplink reach the Internet through the other core.
- **`default-information originate`** on the border router distributes the default route.

## Internet edge

- **ISPA is primary, ISPB is a floating static backup** (administrative distance 5).
- **PAT to 203.0.113.17,** an address from a /29 that both ISPs route toward the school. ISPA holds a floating route to the block through ISPB, so return traffic follows the failover.
- The ISP routers have no route to private address space, which confirms translation is working.

## Wireless

- **WLC 3504** on a single trunk to CORE_SW1. Lightweight APs learn the controller address through DHCP option 43.
- **FlexConnect mode with local switching.** AP switch ports are trunks with native VLAN 10 (AP management) and VLANs 20, 30, 40 for clients.
- **Three WLANs,** one per user group, each WPA2-PSK with AES.
- **AP groups per building and user type** control which SSID each area broadcasts.

## Security

- **Least-privilege ACLs** inbound on the faculty, student, guest and IoT VLAN interfaces, identical on both cores so policy survives an HSRP failover.
- **Management plane:** SSH version 2 only, local accounts, VTY access restricted to 10.1.10.0/24, idle timeouts, encrypted stored passwords.
- **Access switches** are managed through an address in VLAN 10 only.
- **Port security** (sticky MAC, restrict) on selected staff ports, and **DHCP snooping** on the cores with trusted uplinks and server port.

## Services

- **DHCP** from one server in VLAN 50, relayed by `ip helper-address` on every VLAN interface of both cores.
- **NTP** with MD5 authentication. The server keeps UTC and each device applies the local time zone, with timestamped logging.
