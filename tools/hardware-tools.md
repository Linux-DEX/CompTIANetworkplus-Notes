# Hardware tools

Use these when there is no link, a link at the wrong speed, or a radio problem. Ping will not find a split pair. What each tool is for, in exam language, is in [9.3](../notes/9.0-operations-and-troubleshooting/9.3-tools.md).

Safety that applies to all of them: do not look into a live fiber, do not tone a pair that is still patched into a switch if you can unplug it, and do not open a UPS or a supply (10.5).

## Cable tester (wiremap)

Unplug both ends. Put one end in the main unit and the other in the remote.

The display maps pin 1 to pin 1, through pin 8. You want eight straight pairs, each pair still a pair.

| Reading | What to do |
|---|---|
| Open on one pin | That conductor is broken, or a punch-down missed it. Redo the end the remote is not seeing |
| Short | Two conductors touch. Look at the plug you just crimped |
| Crossed | The pinout differs at the ends. A deliberate crossover looks like this. An accident does too (1.5) |
| Split pair | Pins look "right" one at a time and the pairs are mixed. The wiremap may pass a cheap tester. A certifier fails NEXT, and gigabit performs badly |
| All pins open | You are on the wrong cable, or both ends are unplugged from the tester |

A link LED can be on with a split pair. Wiremap before you reconfigure the switch.

## Certifier

A certifier is the wiremap plus the electrical tests for the category: near-end crosstalk, return loss, length, and delay. You tell it the category (Cat 6, Cat 6a) and the standard, plug in both ends the same way as a wiremap, and save the report.

A pass means that permanent link meets the standard you selected. A fail names the test and often the end. Testing Cat 6a against a Cat 5e limit produces a pass that does not mean what you think. Match the limit to the cable you installed.

## Toner and probe

Find one cable in a bundle of fifty.

1. Unplug the far end from the switch or the PC.
2. Clip or plug the tone generator onto the pair, or into the jack.
3. Sweep the probe along the bundle at the other closet. The loudest cable is the one.
4. Turn the toner off and label both ends before you patch it back.

Tone on a live pair leaks into the adjacent pairs and can upset a switch port. Unplug first. A toner does not tell you the pinout. The cable tester does that after you have found the cable.

## TDR

A time-domain reflectometer sends a pulse down copper and times the reflection. You connect it to one end. It reports a distance to an open, a short, or the end of the cable.

Walk that distance from the end you tested, following the path, not the straight line on the floor plan. A result of "open at 47 m" on a 40 m run means you have more cable in the ceiling than the drawing says, or you started from the wrong end. Length is the useful number. The tester does not punch the pair down for you.

## Visual fault locator, light meter, OTDR

Fiber, in the order you should reach for them.

**Visual fault locator.** A visible red laser into one end of a dark fiber. A break, a bad splice, or a sharp bend glows through the jacket. Use it on a fiber you have confirmed is dark. Do not look into the connector at the far end. Do not use it on a fiber that is still patched into a live optic.

**Light meter** (optical power meter), with a known source at the other end. It reads the power that arrived, in dBm. Compare that to the optic's receive range from its data sheet. A number below the minimum is why the link is down or full of errors: too long, too many connectors, or a dirty end. Clean the connector with a proper cassette and measure again before you replace the optic. Blowing on it makes it worse.

**OTDR.** The fiber version of the TDR. It shows a trace: a drop at each connector or splice, and a spike at a reflective break. The distance to the event is the number you walk. An OTDR needs a launch cable so the first connector is not hidden in the dead zone. It is the tool for "where on this 2 km strand," not for "is this patch cord dead." The light meter answers the patch cord.

TX and RX swapped still looks like no light. Swap the patch at one end before you call it a cut (9.4).

## Loopback

A physical loopback plug ties TX to RX on a copper port or a fiber pair, so the port tests itself with no far device. The interface should come up. If it does, the port electronics are willing, and the fault is the cable or the far end. If it does not, the port or the optic is the fault.

A fiber loopback has to match the connector and has to cross TX to RX. A single-fiber loop on the wrong optic tests nothing.

## Multimeter

Useful on a phone pair (is there battery voltage from the CO) and on a PoE argument only if you know where the DC sits on the pairs. It is the wrong first tool on Ethernet. A pair can measure fine on resistance and still be split. Use the cable tester.

## Wi-Fi analyzer and spectrum analyzer

Stand where the user stands. A reading from the wiring closet is a reading of the closet.

A **Wi-Fi analyzer** lists SSIDs, BSSIDs, channels, channel width, security, and signal (RSSI). You are looking for:

- Two of your own APs on the same 2.4 GHz channel, or on channels other than 1, 6, and 11.
- The client stuck to a distant BSSID when a nearer one is louder (a sticky client, 9.4).
- Security that says open, WEP, or WPS when the design says WPA3.

A **spectrum analyzer** does not decode SSIDs. It shows energy on the frequency, including microwaves, cameras, and anything else that does not speak 802.11. Use it when the Wi-Fi analyzer says the channel is empty and the users still fall off the air whenever a particular machine runs.

Write down the channel, the BSSID, and the RSSI at that spot. "The Wi-Fi is bad" is not a result you can compare after you move the AP.

## Tap

A network tap sits in the cable. It copies both directions to a monitor port and forwards the original traffic. No switch configuration. Use one when a SPAN session would drop packets, or when you are not allowed to change the switch.

Plug the two network ports inline, and the monitor port into the capture laptop. A fiber tap uses a splitter. An aggregating tap merges both directions onto one capture NIC. A non-aggregating tap needs two capture interfaces, one per direction.

Know what the tap does when it loses power. Some pass traffic through (fail open). Some break the link (fail closed). Read that label before you put it in a live path. Remove it when the capture is done.

## PoE tester

A handheld tester plugs into the switch port, or sits between the switch and the device. It reports whether power is present, the voltage, the class, and which pairs carry it.

Use it when an AP or a phone will not power up and you cannot trust the switch display yet. Power on the tester with no device means the switch port is supplying PoE. Power only when the real device is attached means the handshake is the question (`show power inline` on the switch). No power on a port that should provide it is the switch budget, the port config, or a midspan injector that is not in the path (7.1).

## Crimper, punch-down, fiber stripper

These make the end. The cable tester then proves you made it right.

**Crimper.** Strip only the jacket. Keep the pair twist as close to the plug as the category allows. Lay the wires in T568A or T568B (1.5), the same scheme at both ends of a straight-through cable. Seat them so the copper reaches the end of the plug, then crimp once. Wiremap it. A crimp that passes a tug test and fails the wiremap is still a bad crimp.

**Punch-down.** The permanent cable lands on a 110 block or the back of a jack. The tool blade seats the conductor and trims the tail. The blade has a cut side. Point the cut side toward the scrap, not toward the cable you are keeping. Punch once, firmly. A light punch that leaves the wire proud will open or corrode later.

**Fiber stripper.** It removes the jacket and buffer to the glass, in steps, using the hole that matches that layer. Do not touch the bare fiber. Cleave it square if you are terminating it. Then inspect the end face before it goes into a connector or a splice. A stripper is not an OTDR and it is not a cleaner. The light meter still has to pass after you are done.
