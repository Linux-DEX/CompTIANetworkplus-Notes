# Host address and routes

What this host thinks its address, gateway, and neighbor MACs are. Reachability tests are in [reachability-and-dns.md](reachability-and-dns.md).

Linux commands below also work on macOS where noted. macOS has no `ip` command. Use `ifconfig` there.

## Address, mask, gateway

| Question | Linux | Windows | macOS |
|---|---|---|---|
| Address and mask | `ip -br addr` | `ipconfig /all` | `ifconfig` |
| One interface | `ip addr show eth0` | | `ipconfig getifaddr en0` |
| Gateway | `ip route` | `ipconfig /all` or `route print` | `route -n get default` |
| DNS servers | `resolvectl status` or the file the host uses | `ipconfig /all` | `scutil --dns` |

`ip -br addr` prints one line per interface: the name, whether it is UP, and the address with its prefix length. `192.0.2.10/24` means the mask is 255.255.255.0.

Read it for three things:

- **169.254.x.x** means DHCP failed. The host gave itself an APIPA address (6.4). It can talk only to other 169.254 hosts on that link.
- **No default route** means the host will not leave its own subnet. `ip route` should show a line starting with `default via`.
- **Two defaults** means two gateways. Traffic can leave by either one. Keep one.

`ipconfig /all` is the Windows page that also shows the DHCP server, the lease times, and the DNS servers. `ipconfig` alone hides those.

Renew a lease after you fix DHCP:

```text
ipconfig /release
ipconfig /renew
```

On Linux, renewing depends on how the address was obtained (`dhclient -r eth0` then `dhclient eth0`, or a NetworkManager restart). Check `ip -br addr` again afterward. A renew that returns 169.254 means the server still did not answer.

Flush a stale DNS cache on Windows with `ipconfig /flushdns`. See the cache with `ipconfig /displaydns`. Linux caches in the resolver (`resolvectl flush-caches` on systemd-resolved). The next `dig` should miss and ask again.

## Is the interface actually up?

```text
ip link show eth0
```

`state UP` means the driver believes the link is up. `state DOWN` means there is nothing to ping until the cable, the far end, or an `ip link set eth0 up` is dealt with. Windows shows the same fact as "Media disconnected" in `ipconfig`, or in the adapter status.

Speed and duplex on Linux:

```text
ethtool eth0
```

Look at `Speed`, `Duplex`, and `Link detected`. A gigabit NIC that negotiated 100 Mb/s full is often a damaged pair (9.4). `ethtool -S eth0` shows the driver's error counters (CRC, drops) when the switch is not nearby.

## ARP and neighbors

The host delivers on the local subnet by MAC address. If the gateway's MAC is wrong, routing is not the problem.

| | Command |
|---|---|
| Linux | `ip neigh` |
| Windows | `arp -a` |
| macOS | `arp -a` |

Each line is an IP, a MAC, and a state. `REACHABLE` or a normal dynamic entry is fine. `FAILED` means the host asked and nobody answered. A gateway that flips between two MACs is two devices answering for one address.

Clear one entry and let the host ask again:

```text
ip neigh del 192.0.2.1 dev eth0
arp -d 192.0.2.1
```

The Windows form needs an elevated prompt. The entry comes back on the next packet to that IP.

IPv6 neighbors are in the same `ip neigh` table. A link-local `fe80::` entry for the router is normal (6.3).

## The routing table

```text
ip route
route print
netstat -rn
```

Linux, Windows, macOS, in that order. The line that matters is the one the destination matches.

- A connected subnet (`192.0.2.0/24 dev eth0`) is delivered directly, by ARP.
- Everything else follows the default route to the gateway.
- Longest prefix wins. A route to `10.1.0.0/16` beats the default for destinations inside that prefix.

`ip route get 8.8.8.8` asks the kernel which way that one address will go, and prints the interface and the gateway. Use it when the table is long and you do not want to pick the row by hand.
