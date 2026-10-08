# Troubleshooting log

Issues met during the build, in the order they were solved.

## 1. Spanning-tree instability with the wireless controller connected

**Symptoms.** Links cycling between blocking and forwarding, VLAN interfaces going down, OSPF neighbors dropping with "dead timer expired", HSRP state changes and clients failing DHCP.

**Isolation.** The network was stable with the controller's switch port shut and unstable with it up. The controller was removed and re-added in stages (management VLAN first, then one VLAN at a time) with the same regression tests after each stage. BPDU guard was enabled on the controller-facing port as a diagnostic.

**Finding.** BPDU guard err-disabled the port straight away (`%SPANTREE-2-BLOCK_BPDUGUARD: Received BPDU on port GigabitEthernet1/0/23`), so the switch was receiving BPDUs on a port that should be an edge port.

**Interpretation.** A production Cisco WLC does not run spanning tree and does not send BPDUs, so this is treated as behavior of the Packet Tracer controller model, not of real hardware. The EtherChannel fault in item 3 was present in the same period and also contributed to the instability, so the controller port was not the only cause.

**Fix.** `spanning-tree portfast trunk` and `spanning-tree bpdufilter enable` on that port, so the switch neither sends BPDUs to the controller nor acts on any it receives. This is acceptable only because the controller is single-homed and no loop can form through it. On real equipment the port would use PortFast trunk with BPDU guard left on.

## 2. Two HSRP active routers on VLAN 60

**Symptoms.** `show standby brief` listed both cores as active for group 60 with the standby router "unknown".

**Root cause.** The IoT ACL is applied inbound on the VLAN interface and ended with `deny ip any any`. It was dropping the other core's HSRP hellos (UDP 1985 to 224.0.0.102). The other ACLs end with a permit, so only VLAN 60 was affected.

**Fix.** Two permit entries for HSRP, limited to the two core interface addresses.

## 3. EtherChannel members suspended

**Symptoms.** `show etherchannel summary` showed the port-channel down with members stand-alone or suspended, and the log reported "vlan mask is different".

**Root cause.** The member ports and the port-channel interface had different trunk settings.

**Fix.** Removed the bundle, defaulted the member ports, joined them to the channel group while unconfigured, then applied the trunk settings to the port-channel only.

## 4. OSPF adjacencies between the cores on every VLAN

**Symptoms.** Six parallel adjacencies between the same two switches, with repeated resets in the log.

**Cause.** OSPF was enabled on every VLAN interface without passive interfaces. This is also a security exposure, because a host in a user VLAN could attempt to become a neighbor.

**Fix.** `passive-interface default`, with OSPF active only on the border uplink and VLAN 10. The adjacency stabilised after this change and a reload of the simulation.

## 5. Wireless clients associated but received no address in their VLAN

**Symptoms.** APs registered to the controller and advertised all WLANs, but clients either did not associate or fell back to 169.254.x.x.

**Root cause.** In Packet Tracer the controller does not tag centrally switched client traffic with the WLAN's VLAN.

**Fix.** FlexConnect local switching on each WLAN, APs set to FlexConnect mode with VLAN support, and AP switch ports changed from access to trunk (native VLAN 10, VLANs 20, 30, 40 allowed).

## 6. APs not joining after the controller address changed

**Symptoms.** APs had DHCP addresses but showed "CAPWAP status: not connected".

**Root cause.** APs learn the controller address (option 43) only when they obtain a lease, so they were still using the old address.

**Fix.** Updated the DHCP pool and power-cycled the APs.

## 7. NTP clients never synchronised

**Symptoms.** The server was reachable (reach 377) but never selected, with an offset of about 12 hours.

**Root cause.** The server clock was set to local time while NTP carries UTC, and the client clocks started far from the correct time.

**Fix.** Server clock set to UTC, client clocks set close to the correct time, and `clock timezone PHT 8 0` on every device.
