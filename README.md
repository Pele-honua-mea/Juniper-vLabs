# Juniper-vLabs
Labs in preparation for JNCIA-SEC:
1. Configuration of security zones & policies to allow ICMP communication between host1 and host2.
2. Configuration of security zones & policies to allow ICMP communication from host1 and host3 and HTTPS communication from host3 to host1.

Topology:
vSRX is connected to host1 through ge-0/0/0, to host2 through ge-0/0/1 and host3 through ge-0/0/3. Host1 and host2 are in trust zone. Host3 is in untrust zone.
