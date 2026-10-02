# Ports, capture, and throughput

Use this after an address ping works and the application still fails. Listening ports, a short capture, and a throughput test answer three different questions.

Only capture and scan hosts and networks you administer. A capture of a password or a scan of a network you do not operate is not a troubleshooting step.

## Who is listening, and to whom

| | Listening sockets | All connections |
|---|---|---|
| Linux | `ss -tulpn` | `ss -tpn` |
| Windows | `netstat -ano` | `netstat -ano` |
| macOS | `lsof -nP -iTCP -sTCP:LISTEN` | `netstat -anv` |

`ss -tulpn` columns that matter: the local address and port, the state, and the process. `0.0.0.0:22` or `*:22` means SSH is listening on every IPv4 address. `127.0.0.1:22` means it is listening only for local clients, so a remote PC will fail even though the daemon is running.

States you will actually act on (4.2):

| State | Meaning |
|---|---|
| `LISTEN` | Waiting for new connections |
| `ESTABLISHED` | A working session |
| `SYN-SENT` | This host sent SYN and has no SYN-ACK. The path or a filter is eating it |
| `SYN-RECV` | SYN arrived, SYN-ACK went out, the ACK never came back |
| `TIME-WAIT` | This side closed and is holding the port briefly. A few of these are normal |
| `CLOSE-WAIT` | The other side finished, and this application has not closed. A growing pile is an application bug |

On Windows the PID is the last column of `netstat -ano`. `tasklist /fi "PID eq 1234"` names the process. On Linux, `ss -p` already prints the name when you run it as root.

## A single port, without a scan

```text
nc -vz 192.0.2.10 22
Test-NetConnection 192.0.2.10 -Port 22
```

Use this for one service you already expect. For several ports on a host you administer:

```text
nmap -sT -p 22,53,80,443 192.0.2.10
```

`-sT` completes a normal TCP handshake, the same thing a client does. `open` means a handshake finished. `closed` means a reset, so nothing is listening. `filtered` means no answer, which is a filter or a black hole. Add `-Pn` only when you already know the host is up and it does not answer ping.

Leave it there. A discovery of your own subnet, when you need one, is `nmap -sn 192.0.2.0/24`, which lists hosts that answer. Do not point either command at a network you do not operate.

## tcpdump

Capture a small slice, with a filter, for a short time. Write a file if you need Wireshark later.

```text
sudo tcpdump -n -i eth0 -c 50 icmp
sudo tcpdump -n -i eth0 host 192.0.2.10 and port 53
sudo tcpdump -n -i eth0 -c 100 -w /tmp/dns.pcap host 192.0.2.10 and port 53
```

| Flag | Effect |
|---|---|
| `-n` | Print numbers, not names. A capture that pauses to do DNS is polluting the test |
| `-i eth0` | One interface. `any` sees all of them and hides which NIC the packet used |
| `-c 50` | Stop after 50 packets |
| `-w file` | Write the raw packets. Omit it to watch them scroll |

On macOS the interface is often `en0`. Find it with `ifconfig` before you capture the wrong NIC.

What a line is telling you:

```text
IP 192.0.2.10.51514 > 192.0.2.53.53: UDP, length 40
IP 192.0.2.53.53 > 192.0.2.10.51514: UDP, length 56
```

The client asked DNS from an ephemeral port, and the resolver answered. A query with no answer is a filter or a dead resolver. A TCP line with `Flags [S]` and no `Flags [S.]` back is a handshake that never completed.

Stop a capture that is scrolling past you. A multi-gigabit interface will drop packets in tcpdump, and the missing packets will look like loss that did not happen on the wire. Filter harder, or span the port on the switch and capture there (the device commands).

## Wireshark

Open a `.pcap`, or capture on the right interface and set a capture filter before you start. A capture filter uses pcap syntax (`host 192.0.2.10 and port 53`). A display filter hides rows you already captured.

| Display filter | Shows |
|---|---|
| `ip.addr == 192.0.2.10` | Packets to or from that host |
| `tcp.port == 443` | HTTPS and anything else on 443 |
| `dns` | DNS |
| `icmp` | Ping and the errors traceroute provokes |
| `tcp.analysis.retransmission` | TCP sending the same data again |
| `tcp.analysis.zero_window` | The receiver telling the sender to stop |

Follow one conversation with the TCP stream view when you need the order of a handshake: SYN, SYN-ACK, ACK, then data. A handshake that ends after SYN is the same failure `SYN-SENT` was reporting.

Statistics for the conversation tell you retransmissions and the throughput of that flow. That number is one flow. It is not the capacity of the link. iPerf is the capacity test.

## iperf3

One machine listens. The other sends. Both need `iperf3`, and the path must allow TCP 5201.

```text
iperf3 -s
iperf3 -c 192.0.2.10 -t 10
```

The sender prints a bitrate. Run it for 10 seconds, not one, or a slow start dominates the average. Compare the number to the link you think you have:

- Far under the link speed, with no loss, often means a small window, a bad duplex, or a CPU on one of the two hosts.
- The same test to a second host on the same switch tells you whether the first server was the limit.
- UDP, for a voice-style check of loss and jitter: `iperf3 -c 192.0.2.10 -u -b 20M -t 10`. The receiver's report is the one that counts. The sender does not know what arrived.

A firewall between them that drops 5201 looks like a hung client. Allow the port for the test and remove it afterward.
