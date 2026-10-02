# Reachability and DNS

Ping proves a path for ICMP. DNS proves a name. They fail separately, so test an address before you test a name.

## ping

| | Command |
|---|---|
| Linux, macOS | `ping -c 4 192.0.2.1` |
| Windows | `ping -n 4 192.0.2.1` |
| IPv6 | `ping -6 -c 4 2001:db8::1` or, on Windows, `ping -6 2001:db8::1` |

Four replies with a steady time mean the path carried ICMP both ways. Read the summary, not a single reply:

| Result | What it usually means |
|---|---|
| `Destination host unreachable` from your own host | No route, or ARP for the gateway failed. Check [the address and the route](host-address-and-routes.md) |
| Unreachable from a router | That router has no route, or the far host did not answer ARP |
| `Request timed out` / 100% loss | Filtered ICMP, a black hole, or a host that is down. Try a TCP check on a port you expect to be open |
| Replies, time swinging from 10 ms to 400 ms | Delay or congestion. Loss in the same run is a different problem from high delay |
| Address pings, name does not | DNS. Stay in this page and use `dig` |

A name in the ping command hides a DNS failure inside a reachability test. Ping the IP first.

### A size test, for MTU

The standard Ethernet payload is 1500 bytes. ICMP echo adds 8 bytes, and the IPv4 header adds 20, so the largest ping data that still fits is **1472**.

| | Don't-fragment ping |
|---|---|
| Windows | `ping -f -l 1472 -n 2 192.0.2.1` |
| Linux | `ping -M do -s 1472 -c 2 192.0.2.1` |
| macOS | `ping -D -s 1472 -c 2 192.0.2.1` |

A reply means 1500-byte packets fit. "Packet needs to be fragmented" or a silent loss of only the large ping means some hop has a smaller MTU (a tunnel is the usual reason, 9.4). Drop to 1400, then 1300, until it passes, and the working size plus 28 is the path MTU.

## traceroute

TTL starts at 1. Each router that decrements it to 0 should answer, then the probe is sent with TTL 2, and so on (5.2).

| | Command | Probe |
|---|---|---|
| Windows | `tracert -d 192.0.2.1` | ICMP |
| Linux | `traceroute -n 192.0.2.1` | UDP, high ports |
| macOS | `traceroute -n 192.0.2.1` | UDP |
| Linux, ICMP probes | `traceroute -n -I 192.0.2.1` | ICMP, closer to what Windows sends |

`-n` and `-d` skip reverse DNS so a slow name lookup does not look like a slow hop.

How to read the rows:

- The last hop that answers, before the stars, is as far as you can see.
- A row of `* * *` often means that hop does not send ICMP time-exceeded, or a filter dropped the probe. Later hops may still answer. Stars in the middle with a final hop that answers are a filter, not a break.
- A sudden jump in time that stays high for every hop after that is delay introduced at the jump.
- The same address repeating until the trace ends is a loop.

`pathping 192.0.2.1` on Windows waits and then prints loss at each hop. `mtr -nzwb 192.0.2.1` does the same job on Linux and macOS and keeps running until you stop it. Loss on the last hop only is the destination. Loss that begins at an early hop and continues is that hop or the link into it.

## dig

`dig` asks a resolver and shows the answer plus which server sent it.

```text
dig example.com
dig example.com AAAA
dig @192.0.2.53 example.com
dig -x 192.0.2.10
dig example.com MX
```

| Piece of the output | Meaning |
|---|---|
| `status: NOERROR` with an answer section | The name resolved |
| `status: NXDOMAIN` | The resolver says this name does not exist |
| `status: SERVFAIL` | The resolver could not complete the lookup. The failure may be upstream |
| `SERVER:` | Which resolver answered. If this is a public resolver, it will not know internal names |
| The ANSWER section | The records. A CNAME means you were sent to another name; keep reading |
| Query time | The resolver's speed, not the web server's |

`dig @192.0.2.53 example.com` asks that server directly. Use it when you suspect the host is pointed at the wrong resolver. `dig +trace example.com` walks from the root and shows each delegation. `dig -x` is the reverse lookup (PTR, 6.5).

Compare an internal name on the internal resolver and on a public one. The public one should fail the internal name. If the host's configured resolver is the public one, that is why `files.corp.example` does not resolve (6.5).

## nslookup

Windows still has `nslookup` everywhere. `dig` is the clearer tool when it is installed.

```text
nslookup example.com
nslookup example.com 192.0.2.53
nslookup -type=MX example.com
```

The server line is who answered. A non-authoritative answer came from a cache. An authoritative answer came from a server that holds the zone.

An interactive session:

```text
nslookup
set type=MX
example.com
exit
```

`set type=A`, `AAAA`, `PTR`, `NS`, and `TXT` are the other queries worth making. For a PTR, ask the address: `nslookup 192.0.2.10`.

## A TCP check when ping is the wrong tool

Many servers drop ICMP and still serve the application.

```text
nc -vz 192.0.2.10 443
telnet 192.0.2.10 443
curl -Iv https://192.0.2.10/
```

`telnet` to a port is the same handshake test on a PC that has no `nc`. A blank screen, or a few HTTP bytes, means the port accepted the connection. "Connection refused" means nothing is listening. A hang means the SYN was dropped. Leave with Ctrl-] then `quit` on a Unix telnet, or close the window. This is not the Telnet protocol on port 23.

`nc -vz` prints `succeeded` when the TCP handshake completes, and `refused` when nothing is listening. A hang until timeout means the SYN was dropped. `curl -Iv` then shows the TLS certificate name and the HTTP status, which separates "the port is open" from "this is the right site."

PowerShell, when `nc` is not there:

```text
Test-NetConnection 192.0.2.10 -Port 443
```

`TcpTestSucceeded : True` is the handshake. The ping column on the same report can be False at the same time. Believe the port result for the application.
