# Firewall policy design

Two FortiGate firewalls:

- **FortiGate 1 (internal)** — between the campus LAN and the router R1
- **FortiGate 2 (external)** — between R1 / DMZ and the Internet (ISP)

Design principle: **least privilege** — only the flows strictly required are allowed, everything else is denied.

## FortiGate 1 — internal

| # | Name | Source | Destination | Service | Action | NAT |
|---|---|---|---|---|---|---|
| 1 | LAN_to_WAN | internal VLAN address objects (authenticated AD group) | any | all | Accept | Outgoing interface |
| 2 | WAN_to_LAN | WAN | LAN | all | Deny | — |

LAN → WAN additionally carries: SSL deep inspection, IPS (prevention), web filter, application control, and a deny rule for a social-media site.

## FortiGate 2 — external

| # | Name | Source | Destination | Service | Action | NAT |
|---|---|---|---|---|---|---|
| 1 | Internet_to_DMZ | any | VIP `dmz_nat` | HTTP, HTTPS | Accept | DNAT |
| 2 | DMZ_to_Internet | DMZ web/DNS server | any | DNS, HTTP, HTTPS (limited) | Accept | Source NAT |
| 3 | WAN_to_LAN | Internet | LAN | all | Deny | — |

Policy 1 also carries IPS in **monitor** mode for traffic towards the DMZ.

## VIP

| Field | Value |
|---|---|
| Name | `dmz_nat` |
| Interface | `wan1` |
| External IP | 11.0.0.47 |
| Mapped IP | 192.168.10.100 |

## Inspection order for web filtering

1. URL exempt → allowed without filtering
2. Static URL filter
3. FortiGuard category filter
4. Advanced filters (keywords…)
5. Action: block, warn or allow

## Verification performed

- Internal users reach the Internet through NAT (ping OK)
- External client reaches the DMZ web server by IP and by DNS name
- External → LAN ping fails (policy deny)
- Allowed and blocked flows appear in FortiGate logs
