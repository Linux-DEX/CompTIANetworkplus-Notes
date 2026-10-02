# Switch and router commands

Cisco IOS is the wording the exam uses. Other vendors rename the same facts (`show lldp neighbors` is nearly universal; the route command is not). Run these from a console or from SSH on a device you administer (12.4).

Replace the interface name with the one printed on the box. `GigabitEthernet0/1`, `Gi1/0/1`, and `ge-0/0/1` are the same idea.

## Is the port up?

```text
show ip interface brief
show interfaces status
```

`show ip interface brief` gives the address and two columns, **Status** and **Protocol**. Both must say `up` for the interface to pass traffic.

| Status / Protocol | Meaning |
|---|---|
| up / up | Link is up and the protocol is running |
| administratively down / down | Someone shut it. `no shutdown` after you mean to use it |
| down / down | No link. Cable, optic, or the far end is shut |
| up / down | Layer 1 is up and layer 2 is not. A mismatch, an encapsulation problem, or a protocol that has not come up |

`show interfaces status` adds speed, duplex, and the VLAN on a switch. `a-1000` means autonegotiation settled on 1000. A forced `100` next to an `a-` on the other end is the duplex-mismatch pattern (9.4).

## Counters on that interface

```text
show interfaces GigabitEthernet0/1
```

Near the top: MTU, bandwidth, duplex, and the input and output rate over the last minutes. Then the counters.

| Counter | Read it as |
|---|---|
| CRC | Damaged frames. Cable, noise, a dirty optic, a bad NIC |
| runts | Frames under 64 bytes |
| giants | Frames over the MTU this port expects |
| input errors | The sum of several of the above. Open the detail |
| output drops | The interface queue overflowed. Congestion, or a speed step-down |
| collisions, late collisions | On a full-duplex port, a duplex mismatch |

Clearing counters (`clear counters`) is useful only so you can see whether the number is still climbing. Write down the old value first. A counter that is large and stable is history. A counter that climbs while the user reproduces the fault is the fault.

## MAC, ARP, VLAN

```text
show mac address-table
show mac address-table address 0011.2233.4455
show vlan brief
show interfaces trunk
show arp
```

The MAC table says which port last sourced that MAC, in which VLAN. A PC MAC learned on an uplink means the PC is behind that uplink, which is correct for an access switch's uplink and wrong for a port you thought the PC was plugged into. The same MAC flapping between two ports is a loop or a dual-homed device without a bundle.

`show vlan brief` lists the access ports in each VLAN. A phone in VLAN 1 when the rest of the floor is VLAN 20 is an unconfigured port. `show interfaces trunk` lists the VLANs allowed on each trunk. A VLAN missing from the allowed list is a VLAN that stops at this switch.

`show arp` on a router or a layer-3 switch maps IP to MAC for networks it is directly attached to. The gateway address the PC uses should appear here with a MAC you recognize.

## Routes

```text
show ip route
show ip route 192.0.2.10
```

The second form shows the one route that matches, which is the one the router will use. The letter is the source: `C` connected, `S` static, `O` OSPF, `B` BGP, `D` EIGRP. Longest prefix wins, then administrative distance (7.2). A default route looks like `S* 0.0.0.0/0`.

No route for that destination means the packet is dropped here, or it follows the default. Check which of those the output says before you blame the far server.

## Neighbors, power, spanning tree, bundles

```text
show cdp neighbors
show lldp neighbors
show power inline
show spanning-tree vlan 20
show etherchannel summary
```

CDP and LLDP tell you what is on the other end of the cable: hostname, platform, and port. That is the fastest check that you patched the right switch. Disable them facing an untrusted network (10.4).

`show power inline` shows the PoE budget and what each port is drawing. A new AP that never boots, with the port at the power limit, is this table (9.4).

`show spanning-tree vlan 20` shows the root and the role of each port: designated (forwarding toward downstream), root (the best path toward the root), or alternate (blocked). A port you expected to forward that says `BLK` is spanning tree doing its job. A sudden root you do not recognize is a switch that should not be winning the election (7.1).

`show etherchannel summary` uses flags. `P` on a member means it is in the bundle. `s` suspended, or a member missing, means that link's settings do not match the others (12.4).

## Ping and traceroute from the device

A PC ping and a router ping leave from different addresses.

```text
ping 192.0.2.10
ping 192.0.2.10 source 198.51.100.1
traceroute 192.0.2.10
```

Set the source to the address the far side expects, or the return path is a different interface than the one you are testing. Five exclamation marks are success. `U` is unreachable. `.` is a timeout. The same letters mean the same things in traceroute.

## Config you can read without changing it

```text
show running-config
show startup-config
```

Running is what is in effect. Startup is what comes back after a reload. A change that was never saved exists only in running. Pipe to a section when the config is long: `show running-config | section interface GigabitEthernet0/1`. Read it. Making the change is a separate step, in a window, with a backout (9.5).

## Port mirror (SPAN)

A mirror copies frames from the port you care about to the port where the laptop is running Wireshark. The laptop does not have to sit inline.

```text
monitor session 1 source interface GigabitEthernet1/0/5 both
monitor session 1 destination interface GigabitEthernet1/0/24
```

`both` copies traffic in and out of the source. The destination port is dedicated to the capture. Plug the laptop in there, start the capture, reproduce the fault, then remove the session:

```text
no monitor session 1
```

`show monitor session 1` confirms it is on. A mirror drops packets when the source is busier than the destination port. A 10 Gb/s server mirrored onto a 1 Gb/s laptop port will lie about loss. For a full copy of a busy link, use a tap, below, in the hardware page. Guests on one virtual switch never appear on a physical SPAN (11.1). Mirror on the virtual switch, or route that traffic through something you can see.
