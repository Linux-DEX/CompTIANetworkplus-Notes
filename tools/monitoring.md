# SNMP and flow data

What filled the link, and what the device itself reports, without capturing every packet. The ideas are in [9.3](../notes/9.0-operations-and-troubleshooting/9.3-tools.md). These are the commands.

Poll and export only devices you administer. An SNMPv2c community string is a password. Prefer v3, and do not leave the secrets in a shell history you share.

## SNMP

The manager asks the agent on UDP 161. A useful walk starts at a small branch of the MIB, not at the root of the whole device.

```text
snmpget -v3 -l authPriv -u USER -a SHA -A AUTH -x AES -X PRIV 192.0.2.1 sysName.0
snmpwalk -v3 -l authPriv -u USER -a SHA -A AUTH -x AES -X PRIV 192.0.2.1 ifTable
```

Replace `USER`, `AUTH`, and `PRIV` with the v3 user configured on that device. `snmpget` returns one object. `sysName.0` is the device name, which confirms you reached the box you think you reached. `snmpwalk` follows a branch. `ifTable` is the interface list: names, speeds, and the in and out octets.

| What you wanted | Where it is |
|---|---|
| Device name, uptime, description | `system` (`sysName`, `sysUpTime`, `sysDescr`) |
| Bytes and errors per interface | `ifTable` |
| Who the manager may be | The device's allowed manager list, not the command |

A timeout means UDP 161 is filtered, the user is wrong, or the agent is off. A walk of the entire MIB (`snmpwalk ... 1`) on a busy core is a load you chose. Walk the branch you need.

Traps are the other direction. The device sends them to UDP 162 on the collector when a threshold trips. You read those on the collector, not with `snmpwalk`. If a port goes down and no trap arrives, the device has no trap receiver configured, or 162 is filtered.

## NetFlow, IPFIX, and sFlow

The router or switch exports a summary of conversations to a collector. You ask the collector who talked to whom. You do not ask the router to print every flow on the console.

On the device, confirm export is on and pointed at the collector:

```text
show flow exporter
show flow monitor
```

You want a destination address you recognize, a recent packet count that is increasing, and no "transport failed" style counter climbing. Typical collector ports are UDP 2055 for classic NetFlow and UDP 4739 for IPFIX. sFlow is the same job with sampled packets instead of a router-built flow record. The collector still answers the question.

On the collector, a query looks like this (`nfdump` is one common reader):

```text
nfdump -R /var/flows -t now-15m -s ip/bytes
nfdump -R /var/flows -t now-15m 'src ip 192.0.2.10'
```

The first lists who moved the most bytes in the last 15 minutes. The second lists conversations from one host. That is how you find the backup that filled the WAN. It will not show you the HTTP headers. A packet capture does that, on the flow you just identified.

No files for that window means the exporter never reached the collector, or the clock on the collector and the time filter disagree. Check NTP before you rebuild the exporter (3.1).
