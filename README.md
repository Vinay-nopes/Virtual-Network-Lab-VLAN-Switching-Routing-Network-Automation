Virtual Network Lab — VLAN Switching, Routing & Network Automation
A hands-on virtual networking lab that models a small enterprise network using Linux network namespaces, virtual Ethernet links, Linux bridges, VLAN segmentation, 802.1Q trunking, router-on-a-stick inter-VLAN routing, DHCP, traffic filtering, and connectivity diagnostics.
Project date: January 2026
Domain: Networking
Implementation note: This project uses Linux virtual networking. Juniper/Junos commands mentioned in the documentation are conceptual equivalents for learning and interview preparation; they are not commands used to implement the Linux lab.

Overview
The lab simulates a small network with two IPv4 networks:
VLAN	Subnet	Example gateway
VLAN 10	192.168.10.0/24	192.168.10.1
VLAN 20	192.168.20.0/24	192.168.20.1


Virtual PCs connect to Linux bridges acting as Layer 2 switches. Access-port VLAN membership separates endpoint traffic. Trunk links carry traffic for multiple VLANs between virtual switches. A virtual router provides Layer 3 connectivity between the two subnets.
Goals
- Create isolated virtual network environments for PCs and network components.
- Connect components with virtual Ethernet (veth) pairs.
- Use Linux bridges for Layer 2 switching.
- Separate devices into VLAN 10 and VLAN 20.
- Configure access-port VLAN membership and 802.1Q trunk links.
- Enable inter-VLAN communication through a virtual router.
- Provide automatic IP assignment using dnsmasq DHCP service.
- Apply traffic-filtering rules using iptables.
- Validate network behavior with ping, route and neighbor inspection, tracepath, and packet captures.
- Use a command-line interface to simplify repetitive network setup and testing.
Technologies
- Linux network namespaces
- Virtual Ethernet (veth) pairs
- Linux bridges and bridge VLAN filtering
- IEEE 802.1Q VLAN tagging
- Router-on-a-stick inter-VLAN routing
- dnsmasq for DHCP
- iptables for traffic filtering
- ping, ip route, ip neigh, bridge vlan, and tracepath
- Wireshark for packet inspection
- Python-based command-line interface for network operations
Conceptual topology
             VLAN 10 endpoints
          PC3 / PC5 / PC6 (examples)
                    |
              Access ports
                    |
             +--------------+
             | Virtual SWs  |
             +--------------+
               |         |
        802.1Q trunk   802.1Q trunk
               |         |
               +----+----+
                    |
              Virtual ROUTER1
              VLAN 10: 192.168.10.1/24
              VLAN 20: 192.168.20.1/24
                    |
             +--------------+
             | Virtual SWs  |
             +--------------+
                    |
              Access ports
                    |
              VLAN 20 endpoints
                 PC8 (example)
This is a conceptual overview rather than a precise interface-by-interface diagram of every component in the lab. The lab included multiple virtual switches and hosts.
How it works
1. Network namespaces
Linux network namespaces provide separate network environments. They are used to represent virtual hosts and network components on one Linux machine.
2. Virtual Ethernet links
A veth pair behaves like a virtual cable. Each end can be attached to a namespace or bridge to connect network components.
3. Layer 2 switching
Linux bridges forward Ethernet frames between connected interfaces. VLAN membership determines which VLAN's traffic a bridge port can handle.
4. VLAN segmentation
- VLAN 10 uses 192.168.10.0/24.
- VLAN 20 uses 192.168.20.0/24.
- Endpoint-facing ports are configured for the appropriate VLAN.
- Trunk links carry traffic for multiple VLANs, using 802.1Q tags for tagged frames.
VLANs create separate Layer 2 broadcast domains. VLAN separation alone is not a complete firewall policy.
5. Inter-VLAN routing
A host sends traffic for a different subnet to its default gateway. The virtual router routes packets between VLAN 10 and VLAN 20 if routing and filtering configuration permits it.

Example gateway addresses:
- VLAN 10: 192.168.10.1
- VLAN 20: 192.168.20.1
6. DHCP with dnsmasq
dnsmasq provides DHCP service so that virtual clients can obtain IP configuration automatically. DHCP may supply an IP address, subnet mask, default gateway, and DNS server information, depending on the configuration.
7. Traffic filtering with iptables
iptables is used to apply Linux packet-filtering rules. These rules can allow or deny selected traffic. Routing determines where packets can be forwarded; filtering determines whether matching traffic is permitted.
8. CLI-based automation
The command-line interface exposes operations such as:
- Create a virtual PC.
- Create a virtual switch.
- Connect devices.
- Assign a VLAN.
- Configure an IP address.
- Create a trunk.
- Test connectivity.
- Display network status.
The backend performs the corresponding virtual-network setup operations. The exact behavior of any operation depends on its implementation. Only document deletion or cleanup behavior if it is implemented and tested in the current codebase.
Validation and diagnostics
The lab uses a combination of connectivity tests and state inspection.
Tool	Purpose
ping	Test IP reachability and round-trip responses
ip route	Inspect routes in the current network namespace
ip neigh	Inspect IPv4/IPv6 neighbor entries
bridge vlan show	Inspect bridge VLAN membership and port configuration
tracepath	Investigate path behavior and path MTU information
Wireshark	Inspect packet headers and traffic visible at the capture point


Recorded test examples
The following results were recorded during development and should be re-run if reproducing the project on another machine:
- Same-VLAN test: PC3 to PC5 succeeded, with 3 packets transmitted, 3 received, and 0% packet loss.
- Inter-VLAN test: PC6 to PC8 succeeded, with 3 packets transmitted, 3 received, and 0% packet loss.
- The router had directly connected routes for 192.168.10.0/24 and 192.168.20.0/24.
- VLAN inspection showed access ports assigned to VLAN 10 or VLAN 20 and trunk links carrying both VLANs.
A TTL change from 64 to 63 is consistent with one router hop when the initial TTL is 64, but TTL alone is not conclusive proof of the complete path.
Troubleshooting approach
When connectivity fails, check in this order:
1. Confirm the relevant interfaces are up.
2. Check IP addresses and subnet masks.
3. Confirm the endpoint is connected to the intended virtual switch.
4. Verify access-port VLAN membership.
5. Verify trunk links carry the required VLANs.
6. Check the default gateway on each host.
7. Inspect the router's route table.
8. Check ARP/neighbor resolution.
9. Review firewall rules and packet-filtering behavior.
10. Use ping, tracepath, and packet capture to narrow down the failure.
Repository documentation
- [`PROJECT_DOCUMENTATION.md`](PROJECT_DOCUMENTATION.md) — detailed explanation of architecture, concepts, packet flow, DHCP, filtering, automation, validation, troubleshooting, and Juniper learning mappings.
Juniper/Junos learning connection
The project was implemented with Linux virtual networking, not Juniper hardware. The underlying concepts map to Juniper networking topics:
Linux lab concept	Related Junos learning topic
VLAN membership	VLAN configuration and switch-port membership
Access port	Ethernet switching interface in access mode
Trunk	Ethernet switching interface in trunk mode
Inter-VLAN routing	Tagged logical interfaces or IRB interfaces, depending on platform
Interface inspection	show interfaces terse
VLAN inspection	show vlans
Routing table	show route
ARP table	show arp
Reachability test	ping <destination>
Path diagnostic	traceroute <destination>


Junos syntax varies by platform, Junos version, and switching configuration style. These mappings are conceptual and should be adapted to the target device.
Limitations and scope
- This is a virtual lab, not a production network.
- Results depend on the Linux environment and the configuration currently present.
- tracepath output depends on network behavior and which diagnostic responses are permitted.
- A successful ping validates the tested ICMP path, not every application or firewall condition.
- Do not claim that every CLI operation, cleanup path, or firewall policy is implemented unless it is present in the code and has been tested.
Learning outcomes
- Practical understanding of Layer 2 switching and Layer 3 routing.
- VLAN and trunk configuration concepts.
- Inter-VLAN packet flow and default-gateway behavior.
- DHCP operation and automatic client configuration.
- Traffic filtering and the distinction between routing and security policy.
- Virtual network setup and repeatable testing through a CLI.
- Systematic troubleshooting using routes, neighbor state, connectivity tests, and packet captures.
