# Juniper-vLabs
Labs in preparation for JNCIA-SEC:
1. Trust-to trust: Configuration of security zones & policies to allow ICMP communication between host1 and host2.
2. Trust to untrust, untrust to trust: Configuration of security zones & policies to allow ICMP communication from host1 and host3 and HTTPS communication from host3 to host1.
3. Simulating communication from private hosts (Host1 & Host2, 10.100.x.x) to a public host (Host3, 200.0.13.2) through a vSRX security device. The vSRX performs source NAT to allow Host3 to reply, demonstrating how NAT enables private-to-public communication in a controlled firewall environment.

Topology:
vSRX is connected to host1 through ge-0/0/0, to host2 through ge-0/0/1 and host3 through ge-0/0/3. Host1 and host2 are in trust zone. Host3 is in untrust zone.
