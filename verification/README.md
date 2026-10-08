# Verification screenshots

## Redundancy and routing

| File | Content |
|---|---|
| `hsrp-core-sw1.png` | `show standby brief` on CORE_SW1: active for VLANs 10, 20, 30 and standby for 40, 50, 60 |
| `hsrp-core-sw2.png` | `show standby brief` on CORE_SW2: the reverse |
| `etherchannel-summary.png` | `show etherchannel summary` on CORE_SW1: Po1 in use with both members bundled |
| `ospf-neighbors.png` | `show ip ospf neighbor` on the border router: both cores FULL |

## Internet edge

| File | Content |
|---|---|
| `nat-translations.png` | A student PC (10.1.30.130) pings 8.8.8.8, and `show ip nat translations` on the border router shows it translated to 203.0.113.17 |
| `isp-failover.png` | Two traceroutes to 8.8.8.8 from an admin PC. First: normal path through the primary ISP (203.0.113.2). Second: primary link shut, so the path moves to ISPB (198.51.100.2) and crosses the ISP peering link (192.0.2.1) |

## Access policy

| File | Content |
|---|---|
| `acl-student.png` | Student PC: Internet reachable; management, faculty and SSH to the gateway blocked |
| `acl-guest.png` | Guest PC: Internet reachable; servers and students blocked |
| `acl-faculty.png` | Faculty PC: Internet and servers reachable; management blocked |
| `acl-admin-and-iot.png` | Admin PC: SSH to a core and ping to a camera succeed. Border router: ping to the same camera fails, because the camera may only answer management and servers |

## Wireless and services

| File | Content |
|---|---|
| `wlc-access-points.png` | Controller page listing the six registered APs |
| `ntp-status.png` | `show ntp status` and `show ntp associations` on both cores: synchronised to 10.1.50.10 |

**Note on `wlc-access-points.png`:** the list shows seven entries. Six are the
registered APs (status REG). The seventh (0002.4A9D.9B01, IP 0.0.0.0, status
DOWN) is a stale record of a test AP that was removed from the topology.
Packet Tracer's controller keeps the entry and does not support deleting it
from the GUI. On a real controller it would age out or be removed manually.
