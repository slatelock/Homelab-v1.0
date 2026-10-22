# WAN Speed Troubleshooting

## Summary

The gateway's WAN link was stuck at Fast Ethernet (100 Mbps), capping internet speed at about 90 Mbps on a 3 Gbps plan. Isolation testing ruled out the cabling, the ISP and the gateway's configuration. A test suggested by vendor support then showed the WAN port itself was fine: the actual fault was a link-negotiation incompatibility between the ISP modem and the gateway. Putting a switch between the two restored a 2.5 Gbps link, and vendor support recommended keeping it that way.

## Environment

- ISP modem/router used for internet hand-off only, running ISP-pushed pre-release firmware
- UniFi Cloud Gateway Max (2.5GbE WAN port)
- 3 Gbps plan; the modem itself reported over 3 Gbps with no other traffic. The gateway's 2.5GbE WAN port is the realistic ceiling

## Symptom

The gateway reported its WAN link at 100 Mbps, and speed tests through it topped out around 90 Mbps.

## Investigation

| Step | Test | Result | Conclusion |
| --- | --- | --- | --- |
| 1 | Forced the modem's port to 2.5 Gbps (gateway left on auto) | Link dropped; modem needed a factory reset. Repeated once with the same result | Forcing one side of a link breaks auto-negotiation |
| 2 | Swapped in a known-good cable, then re-tested the suspect run | Both cables worked | Cabling ruled out |
| 3 | A/B test with a wired test device (1 Gbps port): modem → device, then gateway → device, using both cables | Modem: ~939 Mbps, the device's own ceiling. Gateway: ~90 Mbps | ISP side ruled out; fault isolated to the modem-to-gateway link |
| 4 | Full factory reset of the gateway | Still 100 Mbps | Corrupted config and firmware ruled out. Looked like a hardware fault, so an RMA was opened |
| 5 | Vendor support asked for an intermediate switch: modem → Flex 2.5G switch → gateway WAN | Link negotiated at 2.5 Gbps; the 1 Gbps test device maxed out again at ~872 Mbps | The WAN port works. The fault is negotiation between these two specific devices |

After the step 4 reset, the full network config (VLANs, firewall zones, switch ports, Wi-Fi, RF plan) was rebuilt from documentation.

## Root cause

A link-negotiation incompatibility between the ISP modem, which runs pre-release firmware, and the gateway's WAN port. It is not a hardware failure. Step 5 is what overturned the earlier hardware conclusion.

## Resolution

- Vendor support recommended keeping a switch in the WAN path permanently, so no RMA.
- Rather than tie up the Flex, a cheap unmanaged 2.5GbE switch now sits between the modem and gateway. Its only job is translating between the two.
- The Flex went back to normal LAN duty.
- Result: a desktop now pulls almost 2 Gbps down and 600 Mbps up, and that's through a VPN tunnel to a server in another city.

## Notes to self

- Don't force link speed on one side. That was two modem factory resets for nothing.
- The A/B test is what actually cracked it. Should have done it first.
- "A reset didn't fix it" doesn't mean the hardware's dead. I was one step away from shipping back a working gateway.
- Writing the config down paid off fast. The factory reset wiped everything, and rebuilding was copying, not thinking.
