# Networking tools and commands

How to run the tools, and what the answer means. The exam explanations live in the notes: host commands in [6.7](../notes/6.0-addressing/6.7-services-and-tools.md), hardware and flow data in [9.3](../notes/9.0-operations-and-troubleshooting/9.3-tools.md), device `show` commands in [12.4](../notes/12.0-n10-009/12.4-recovery-dns-and-management.md).

Run these on networks and hosts you administer. A scan or a capture aimed at someone else's network is not troubleshooting.

| | |
|---|---|
| [Host address and routes](host-address-and-routes.md) | `ip`, `ipconfig`, `arp`, `route` |
| [Reachability and DNS](reachability-and-dns.md) | `ping`, `traceroute`, `dig`, `nslookup` |
| [Ports, capture, throughput](ports-capture-and-throughput.md) | `ss`, `netstat`, `tcpdump`, Wireshark, `iperf3` |
| [Switch and router commands](switch-and-router-commands.md) | The `show` commands, SPAN, ping from the device |
| [Hardware tools](hardware-tools.md) | Tester, toner, TDR, OTDR, tap, crimper, Wi-Fi analyzer |
| [SNMP and flow data](monitoring.md) | `snmpwalk`, NetFlow, IPFIX, sFlow |

## Order

1. Does this host have an address, a mask, and one gateway? ([address](host-address-and-routes.md))
2. Can it reach the gateway, then a far address, then a name? ([reachability](reachability-and-dns.md))
3. Is the port open, and is the conversation healthy? ([ports and capture](ports-capture-and-throughput.md))
4. What does the switch or router say about that same port? ([device](switch-and-router-commands.md))
5. If there is no link light, stop using ping and use a [hardware tool](hardware-tools.md).
