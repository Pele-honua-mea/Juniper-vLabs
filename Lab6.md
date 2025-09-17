# Lab: Configuring Destination NAT and Security Policies on vSRX for external access to internal DMZ servers:



jcluser@vSRX1> show configuration interfaces 

ge-0/0/0 {

&nbsp;   unit 0 {

&nbsp;       family inet {

&nbsp;           address 10.100.11.1/24;

&nbsp;       }

&nbsp;   }

}

ge-0/0/1 {

&nbsp;   unit 0 {

&nbsp;       family inet {

&nbsp;           address 10.100.12.1/24;

&nbsp;       }

&nbsp;   }

}

ge-0/0/2 {

&nbsp;   unit 0 {

&nbsp;       family inet {

&nbsp;           address 10.100.13.1/24;

&nbsp;       }

&nbsp;   }

}

fxp0 {

&nbsp;   unit 0 {

&nbsp;       family inet {

&nbsp;           dhcp;

&nbsp;       }

&nbsp;   }

}

---(more)---\[abort]



#### **Step 1: Assign public IP addresses to vSRX loopback interfaces that will be used as the destination NAT addresses and assign a public IP address to host3's interface which represents the external host trying to access internal servers:**



jcluser@vSRX1> edit       

Entering configuration mode



\[edit]

jcluser@vSRX1# delete interfaces ge-0/0/2.0 family inet address 10.100.13.1/24                 



\[edit]

jcluser@vSRX1# set interfaces ge-0/0/2.0 family inet address 200.0.13.1/30  



\[edit]

jcluser@vSRX1# commit 



\[edit]

jcluser@vSRX1# set interfaces lo0.0 family inet address 1.1.1.1/24             



\[edit]

jcluser@vSRX1# set interfaces lo0.0 family inet address 2.2.2.2/24



\[edit]

jcluser@vSRX1# commit and-quit 

commit complete

Exiting configuration mode 







jcluser@Host3> show configuration interfaces 

ge-0/0/2 {

&nbsp;   unit 0 {

&nbsp;       family inet {

&nbsp;           address 10.100.13.2/24;

&nbsp;       }

&nbsp;   }

}

fxp0 {

&nbsp;   unit 0 {

&nbsp;       family inet {

&nbsp;           address 100.123.1.2/16;

&nbsp;       }

&nbsp;   }

}



#### **Step 2: Assign a public address to host3's interface:**



jcluser@Host3> edit     

Entering configuration mode



\[edit]

jcluser@Host3# delete interfaces ge-0/0/2.0 inet address 10.100.13.2/24                 



\[edit]

jcluser@Host3# set interfaces ge-0/0/2.0 family inet address 200.0.13.2/30    



\[edit]

jcluser@Host3# commit 

commit complete



#### **Step 3: Rename the host1 and host2 to "Server1" and "Server2" (the internal Resources) and configure static routes on each server pointing to host3's subnet via vSRX as the next-hop:**



jcluser@Host1> edit 

Entering configuration mode



\[edit]

jcluser@Host1# set system host-name Server1 

\[edit]

jcluser@Host1# commit 

commit complete



\[edit]

jcluser@Server1# set routing-options static route 200.0.13.0/30 next-hop 10.100.11.1           



\[edit]

jcluser@Server1# commit and-quit 

commit complete

Exiting configuration mode





jcluser@Host2> edit 

Entering configuration mode



\[edit]

jcluser@Host2# set system host-name Server2 



\[edit]

jcluser@Host2# commit 

commit complete



\[edit]

jcluser@Server2# set routing-options static route 200.0.13.0/30 next-hop 10.100.12.1          



\[edit]

jcluser@Server2# commit and-quit 

commit complete

Exiting configuration mode



#### **Step 4: On host3, add static routes to reach the vSRX loopback addresses used in the NAT pools:**



jcluser@Host3> edit 

Entering configuration mode



\[edit]

jcluser@Host3# set routing-options static route 1.1.1.0/24 next-hop 200.0.13.1                



\[edit]

jcluser@Host3# set routing-options static route 2.2.2.0/24 next-hop 200.0.13.1               



\[edit]

jcluser@Host3# commit and-quit 

commit complete

Exiting configuration mode



#### **Step 5: Remove the default trust-zone security policies, create a DMZ zone for the internal servers, and assign the interfaces appropriately (ge-0/0/0 and ge-0/0/1 --> DMZ, ge-0/0/2 --> untrust):** 





jcluser@vSRX1> show configuration security zones | display set                 

set security zones security-zone trust host-inbound-traffic system-services any-service

set security zones security-zone trust interfaces ge-0/0/0.0

set security zones security-zone trust interfaces ge-0/0/1.0

set security zones security-zone untrust interfaces ge-0/0/2.0



jcluster@vSRX1> edit

Entering configuration mode



\[edit]

jcluser@vSRX1# delete security-policies policies from-zone trust to-zone trust policy default-permit   



\[edit]

jcluser@vSRX1# delete security policies from-zone trust to-zone untrust policy default-permit      



\[edit]

jcluser@vSRX1# delete security zones security-zone trust



\[edit]

jcluser@vSRX1# commit and-quit  

commit complete

Exiting configuration mode



jcluser@vSRX1> show configuration security zones | display set                 

set security zones security-zone dmz interfaces ge-0/0/0.0

set security zones security-zone dmz interfaces ge-0/0/1.0



jcluser@vSRX1> edit 

Entering configuration mode



\[edit]

jcluser@vSRX1# set security zones security-zone dmz host-inbound-traffic system-services any-service           



\[edit]

jcluser@vSRX1# set security zones security-zone untrust interfaces ge-0/0/2.0                 



\[edit]

jcluser@vSRX1# commit and-quit 



/config/license/TrialJUNOS986842235.lic:1:(0) TrialJUNOS986842235: Ignoring the license for feature id: 292.This feature is not used by your product.

commit complete

Exiting configuration mode



jcluser@vSRX1> show configuration security zones | display set                 

set security zones security-zone dmz host-inbound-traffic system-services any-service

set security zones security-zone dmz interfaces ge-0/0/0.0

set security zones security-zone dmz interfaces ge-0/0/1.0

set security zones security-zone untrust interfaces ge-0/0/2.0



jcluser@vSRX1> show security zones 



Security zone: dmz

&nbsp; Zone ID: 10

&nbsp; Send reset for non-SYN session TCP packets: Off

&nbsp; Policy configurable: Yes  

&nbsp; Interfaces bound: 2

&nbsp; Interfaces:

&nbsp;   ge-0/0/0.0

&nbsp;   ge-0/0/1.0

&nbsp; Advanced-connection-tracking timeout: 1800

&nbsp; Unidirectional-session-refreshing: No



Security zone: untrust

&nbsp; Zone ID: 7

&nbsp; Send reset for non-SYN session TCP packets: Off

&nbsp; Policy configurable: Yes  

&nbsp; Interfaces bound: 1

&nbsp; Interfaces:

&nbsp;   ge-0/0/2.0

&nbsp; Advanced-connection-tracking timeout: 1800

&nbsp; Unidirectional-session-refreshing: No



Security zone: junos-host

&nbsp; Zone ID: 2

&nbsp; Send reset for non-SYN session TCP packets: Off

&nbsp; Policy configurable: Yes 

&nbsp; Interfaces bound: 0

&nbsp; Interfaces:

---(more)---\[abort]



#### **Step 6: Add the internal servers' addresses to the DMZ zone's address book and create a security policy allowing untrust --> DMZ traffic for IMCP, SSH and HTTPS from any source:**



jcluser@vSRX1> edit 

Entering configuration mode



\[edit]

jcluser@vSRX1# set security zones security-zone dmz address-book address DMZ\_SERVER1 10.100.11.2/32           



\[edit]

jcluser@vSRX1# set security zones security-zone dmz address-book address DMZ\_SERVER2 10.100.12.2/32    



\[edit]

jcluser@vSRX1# set security policies from-zone untrust to-zone dmz policy to\_web\_server match source-address any  



\[edit]

jcluser@vSRX1# set security policies from-zone untrust to-zone dmz policy to\_web\_server match destination-address DMZ\_SERVER1 



\[edit]

jcluser@vSRX1# set security policies from-zone untrust to-zone dmz policy to\_web\_server match destination-address DMZ\_SERVER2  



\[edit]

jcluser@vSRX1# set security policies from-zone untrust to-zone dmz policy to\_web\_server match application junos-https 



\[edit]

jcluser@vSRX1# set security policies from-zone untrust to-zone dmz policy to\_web\_server match application junos-ssh



\[edit]

jcluser@vSRX1# set security policies from-zone untrust to-zone dmz policy to\_web\_server match application junos-ping 



\[edit]

jcluser@vSRX1# set security policies from-zone untrust to-zone dmz policy to\_web\_server then permit



\[edit]

jcluser@vSRX1# commit and-quit 

commit complete

Exiting configuration mode



#### **Step 7: Configure destination NAT so that traffic from host3 to 1.1.1.1 is translated to Server1's internal address (10.100.11.2):** 



jcluser@vSRX1> edit 

Entering configuration mode



\[edit]

jcluser@vSRX1# set security nat destination pool DMZ\_SERVER1\_PL address 10.100.11.2/32        



\[edit]

jcluser@vSRX1# set security nat destination rule-set DMZ\_SERVER\_RS from zone untrust          



\[edit]

jcluser@vSRX1# set security nat destination rule-set DMZ\_SERVER\_RS rule DMZ\_SERVER\_RL match destination-address 1.1.1.1/32                    



\[edit]

jcluser@vSRX1# set security nat destination rule-set DMZ\_SERVER\_RS rule DMZ\_SERVER\_RL then destination-nat pool DMZ\_SERVER1\_PL                  



\[edit]

jcluser@vSRX1# commit and-quit 



#### **Step 8: Verify connectivity from host3 to Server1 using ping, SSH and telnet:**



jcluser@Host3> ping 1.1.1.1                   

PING 1.1.1.1 (1.1.1.1): 56 data bytes

64 bytes from 1.1.1.1: icmp\_seq=0 ttl=63 time=1.190 ms

64 bytes from 1.1.1.1: icmp\_seq=1 ttl=63 time=1.150 ms

64 bytes from 1.1.1.1: icmp\_seq=2 ttl=63 time=1.062 ms

64 bytes from 1.1.1.1: icmp\_seq=3 ttl=63 time=1.194 ms

64 bytes from 1.1.1.1: icmp\_seq=4 ttl=63 time=1.358 ms

64 bytes from 1.1.1.1: icmp\_seq=5 ttl=63 time=1.430 ms

64 bytes from 1.1.1.1: icmp\_seq=6 ttl=63 time=1.332 ms

^C

--- 1.1.1.1 ping statistics ---

8 packets transmitted, 7 packets received, 12% packet loss

round-trip min/avg/max/stddev = 1.062/1.245/1.430/0.121 ms







jcluser@vSRX1> show security flow session | refresh 

---(refreshed at 2025-09-16 09:28:16 UTC)---

Total sessions: 0

---(refreshed at 2025-09-16 09:28:21 UTC)---

.

.

.

Session ID: 192, Policy name: to\_web\_server/4, State: Stand-alone, Timeout: 2, Valid

&nbsp; In: 200.0.13.2/20564 --> 1.1.1.1/5;icmp, Conn Tag: 0x0, If: ge-0/0/2.0, Pkts: 1, Bytes: 84, 

&nbsp; Out: 10.100.11.2/5 --> 200.0.13.2/20564;icmp, Conn Tag: 0x0, If: ge-0/0/0.0, Pkts: 1, Bytes: 84, 



Session ID: 193, Policy name: to\_web\_server/4, State: Stand-alone, Timeout: 4, Valid

&nbsp; In: 200.0.13.2/20564 --> 1.1.1.1/6;icmp, Conn Tag: 0x0, If: ge-0/0/2.0, Pkts: 1, Bytes: 84, 

&nbsp; Out: 10.100.11.2/6 --> 200.0.13.2/20564;icmp, Conn Tag: 0x0, If: ge-0/0/0.0, Pkts: 1, Bytes: 84, 



Session ID: 194, Policy name: to\_web\_server/4, State: Stand-alone, Timeout: 4, Valid

&nbsp; In: 200.0.13.2/20564 --> 1.1.1.1/7;icmp, Conn Tag: 0x0, If: ge-0/0/2.0, Pkts: 1, Bytes: 84, 

&nbsp; Out: 10.100.11.2/7 --> 200.0.13.2/20564;icmp, Conn Tag: 0x0, If: ge-0/0/0.0, Pkts: 1, Bytes: 84, 

---(\*more)---\[abort]





jcluser@Host3> ssh 1.1.1.1 

The authenticity of host '1.1.1.1 (1.1.1.1)' can't be established.

ECDSA key fingerprint is SHA256:Ndf+fnyVBCUHIr0yq5sxQQnO57LsZ4N+YozIRAHMqts.

Are you sure you want to continue connecting (yes/no)? no

Host key verification failed.





jcluser@vSRX1> show security flow session | refresh          

---(refreshed at 2025-09-16 09:34:01 UTC)---

Session ID: 217, Policy name: to\_web\_server/4, State: Stand-alone, Timeout: 1792, Valid

&nbsp; In: 200.0.13.2/61312 --> 1.1.1.1/22;tcp, Conn Tag: 0x0, If: ge-0/0/2.0, Pkts: 7, Bytes: 1925, 

&nbsp; Out: 10.100.11.2/22 --> 200.0.13.2/61312;tcp, Conn Tag: 0x0, If: ge-0/0/0.0, Pkts: 5, Bytes: 1865, 

Total sessions: 1

---(refreshed at 2025-09-16 09:34:06 UTC)---

Session ID: 217, Policy name: to\_web\_server/4, State: Stand-alone, Timeout: 1788, Valid

&nbsp; In: 200.0.13.2/61312 --> 1.1.1.1/22;tcp, Conn Tag: 0x0, If: ge-0/0/2.0, Pkts: 7, Bytes: 1925, 

&nbsp; Out: 10.100.11.2/22 --> 200.0.13.2/61312;tcp, Conn Tag: 0x0, If: ge-0/0/0.0, Pkts: 5, Bytes: 1865, 

Total sessions: 1

---(refreshed at 2025-09-16 09:34:11 UTC)---

Session ID: 217, Policy name: to\_web\_server/4, State: Stand-alone, Timeout: 1782, Valid

&nbsp; In: 200.0.13.2/61312 --> 1.1.1.1/22;tcp, Conn Tag: 0x0, If: ge-0/0/2.0, Pkts: 7, Bytes: 1925, 

&nbsp;   Out: 10.100.11.2/22 --> 200.0.13.2/61312;tcp, Conn Tag: 0x0, If: ge-0/0/0.0, Pkts: 5, Bytes: 1865, 

Total sessions: 1

---(refreshed at 2025-09-16 09:34:16 UTC)---

Session ID: 217, Policy name: to\_web\_server/4, State: Stand-alone, Timeout: 1778, Valid

&nbsp; In: 200.0.13.2/61312 --> 1.1.1.1/22;tcp, Conn Tag: 0x0, If: ge-0/0/2.0, Pkts: 7, Bytes: 1925, 

&nbsp; Out: 10.100.11.2/22 --> 200.0.13.2/61312;tcp, Conn Tag: 0x0, If: ge-0/0/0.0, Pkts: 5, Bytes: 1865, 

Total sessions: 1

---(refreshed at 2025-09-16 09:34:21 UTC)---

Session ID: 217, Policy name: to\_web\_server/4, State: Stand-alone, Timeout: 1772, Valid

&nbsp; In: 200.0.13.2/61312 --> 1.1.1.1/22;tcp, Conn Tag: 0x0, If: ge-0/0/2.0, Pkts: 7, Bytes: 1925, 

&nbsp; Out: 10.100.11.2/22 --> 200.0.13.2/61312;tcp, Conn Tag: 0x0, If: ge-0/0/0.0, Pkts: 5, Bytes: 1865, 

Total sessions: 1

---(refreshed at 2025-09-16 09:34:26 UTC)---

Total sessions: 0

---(\*more 100%)---\[abort]





jcluser@Host3> telnet port 443 1.1.1.1    

Trying 1.1.1.1...

telnet: connect to address 1.1.1.1: Connection refused

telnet: Unable to connect to remote host



jcluser@Host3> telnet port 443 1.1.1.1    

Trying 1.1.1.1...

telnet: connect to address 1.1.1.1: Connection refused

telnet: Unable to connect to remote host

.

.

.



jcluser@Host3> telnet port 443 1.1.1.1    

Trying 1.1.1.1...

telnet: connect to address 1.1.1.1: Connection refused

telnet: Unable to connect to remote host



jcluser@vSRX1> show security flow session | refresh          

---(refreshed at 2025-09-16 09:36:20 UTC)---

Session ID: 223, Policy name: to\_web\_server/4, State: Stand-alone, Timeout: 2, Valid

&nbsp; In: 200.0.13.2/64384 --> 1.1.1.1/443;tcp, Conn Tag: 0x0, If: ge-0/0/2.0, Pkts: 1, Bytes: 64, 

&nbsp; Out: 10.100.11.2/443 --> 200.0.13.2/64384;tcp, Conn Tag: 0x0, If: ge-0/0/0.0, Pkts: 1, Bytes: 40, 



Session ID: 224, Policy name: to\_web\_server/4, State: Stand-alone, Timeout: 2, Valid

&nbsp; In: 200.0.13.2/51844 --> 1.1.1.1/443;tcp, Conn Tag: 0x0, If: ge-0/0/2.0, Pkts: 1, Bytes: 64, 

&nbsp; Out: 10.100.11.2/443 --> 200.0.13.2/51844;tcp, Conn Tag: 0x0, If: ge-0/0/0.0, Pkts: 1, Bytes: 40, 

Total sessions: 2

---(\*more 100%)---\[abort]



#### **Step 9: Configure destination NAT so that traffic from host3 to 2.2.2.2 is translated to Server2's internal address (10.100.12.2):**



jcluser@vSRX1> edit 

Entering configuration mode



\[edit]

jcluser@vSRX1# set security nat destination pool DMZ\_SERVER2\_PL address 10.100.12.2/32        



\[edit]

jcluser@vSRX1# set security nat destination rule-set DMZ\_SERVER\_RS from zone untrust          



\[edit]

jcluser@vSRX1# set security nat destination rule-set DMZ\_SERVER\_RS rule DMZ\_SERVER2\_RL match destination-address 2.2.2.2/32                    



\[edit]

jcluser@vSRX1# set security nat destination rule-set DMZ\_SERVER\_RS rule DMZ\_SERVER2\_RL then destination-nat pool DMZ\_SERVER2\_PL   



\[edit]

jcluser@vSRX1# commit and-quit 

commit complete

Exiting configuration mode



jcluser@vSRX1> clear security nat statistics destination rule all    



#### **Step 10: Verify connectivity from host3 to Server2 using ping, SSH and telnet:**





jcluser@Host3> ping 2.2.2.2 

PING 2.2.2.2 (2.2.2.2): 56 data bytes

64 bytes from 2.2.2.2: icmp\_seq=0 ttl=63 time=1.622 ms

64 bytes from 2.2.2.2: icmp\_seq=1 ttl=63 time=1.171 ms

64 bytes from 2.2.2.2: icmp\_seq=2 ttl=63 time=1.423 ms

64 bytes from 2.2.2.2: icmp\_seq=3 ttl=63 time=1.308 ms

64 bytes from 2.2.2.2: icmp\_seq=4 ttl=63 time=1.224 ms

64 bytes from 2.2.2.2: icmp\_seq=5 ttl=63 time=1.268 ms

64 bytes from 2.2.2.2: icmp\_seq=6 ttl=63 time=1.106 ms

64 bytes from 2.2.2.2: icmp\_seq=7 ttl=63 time=1.282 ms

64 bytes from 2.2.2.2: icmp\_seq=8 ttl=63 time=1.237 ms

64 bytes from 2.2.2.2: icmp\_seq=9 ttl=63 time=1.291 ms

64 bytes from 2.2.2.2: icmp\_seq=10 ttl=63 time=1.010 ms

64 bytes from 2.2.2.2: icmp\_seq=11 ttl=63 time=1.185 ms

64 bytes from 2.2.2.2: icmp\_seq=12 ttl=63 time=1.305 ms

^C

--- 2.2.2.2 ping statistics ---

13 packets transmitted, 13 packets received, 0% packet loss

round-trip min/avg/max/stddev = 1.010/1.264/1.622/0.143 ms







jcluser@vSRX1> show security flow session | refresh          

---(refreshed at 2025-09-16 09:44:13 UTC)---

Session ID: 241, Policy name: to\_web\_server/4, State: Stand-alone, Timeout: 2, Valid

&nbsp; In: 200.0.13.2/47956 --> 2.2.2.2/3;icmp, Conn Tag: 0x0, If: ge-0/0/2.0, Pkts: 1, Bytes: 84, 

&nbsp; Out: 10.100.12.2/3 --> 200.0.13.2/47956;icmp, Conn Tag: 0x0, If: ge-0/0/1.0, Pkts: 1, Bytes: 84, 



Session ID: 242, Policy name: to\_web\_server/4, State: Stand-alone, Timeout: 2, Valid

&nbsp; In: 200.0.13.2/47956 --> 2.2.2.2/4;icmp, Conn Tag: 0x0, If: ge-0/0/2.0, Pkts: 1, Bytes: 84, 

&nbsp; Out: 10.100.12.2/4 --> 200.0.13.2/47956;icmp, Conn Tag: 0x0, If: ge-0/0/1.0, Pkts: 1, Bytes: 84, 

Session ID: 243, Policy name: to\_web\_server/4, State: Stand-alone, Timeout: 4, Valid

&nbsp; In: 200.0.13.2/47956 --> 2.2.2.2/5;icmp, Conn Tag: 0x0, If: ge-0/0/2.0, Pkts: 1, Bytes: 84, 

&nbsp; Out: 10.100.12.2/5 --> 200.0.13.2/47956;icmp, Conn Tag: 0x0, If: ge-0/0/1.0, Pkts: 1, Bytes: 84, 

Total sessions: 3

---(refreshed at 2025-09-16 09:44:18 UTC)---

Session ID: 247, Policy name: to\_web\_server/4, State: Stand-alone, Timeout: 2, Valid

&nbsp; In: 200.0.13.2/47956 --> 2.2.2.2/9;icmp, Conn Tag: 0x0, If: ge-0/0/2.0, Pkts: 1, Bytes: 84, 

&nbsp; Out: 10.100.12.2/9 --> 200.0.13.2/47956;icmp, Conn Tag: 0x0, If: ge-0/0/1.0, Pkts: 1, Bytes: 84, 



Session ID: 248, Policy name: to\_web\_server/4, State: Stand-alone, Timeout: 2, Valid

&nbsp; In: 200.0.13.2/47956 --> 2.2.2.2/10;icmp, Conn Tag: 0x0, If: ge-0/0/2.0, Pkts: 1, Bytes: 84, 

&nbsp; Out: 10.100.12.2/10 --> 200.0.13.2/47956;icmp, Conn Tag: 0x0, If: ge-0/0/1.0, Pkts: 1, Bytes: 84, 



Session ID: 249, Policy name: to\_web\_server/4, State: Stand-alone, Timeout: 4, Valid

&nbsp; In: 200.0.13.2/47956 --> 2.2.2.2/11;icmp, Conn Tag: 0x0, If: ge-0/0/2.0, Pkts: 1, Bytes: 84, 

&nbsp; Out: 10.100.12.2/11 --> 200.0.13.2/47956;icmp, Conn Tag: 0x0, If: ge-0/0/1.0, Pkts: 1, Bytes: 84, 

Total sessions: 3

---(refreshed at 2025-09-16 09:44:23 UTC)---

Total sessions: 0

---(refreshed at 2025-09-16 09:44:28 UTC)---

Total sessions: 0

---(refreshed at 2025-09-16 09:44:33 UTC)---

Total sessions: 0

---(\*more 100%)---\[abort]





jcluser@vSRX1> ...estination rule DMZ\_SERVER2\_RL               

Destination NAT rule: DMZ\_SERVER2\_RL         Rule-set: DMZ\_SERVER\_RS

&nbsp; Rule-Id                    : 2

&nbsp; Rule position              : 2

&nbsp; From zone                  : untrust

&nbsp;   Destination addresses    : 2.2.2.2         - 2.2.2.2

&nbsp; Action                     : DMZ\_SERVER2\_PL

&nbsp; Translation hits           : 13

&nbsp;   Successful sessions      : 13

&nbsp; Number of sessions         : 0





jcluser@Host3> ssh 2.2.2.2                

The authenticity of host '2.2.2.2 (2.2.2.2)' can't be established.

ECDSA key fingerprint is SHA256:Ndf+fnyVBCUHIr0yq5sxQQnO57LsZ4N+YozIRAHMqts.

Are you sure you want to continue connecting (yes/no)? ^C



jcluser@vSRX1> show security flow session | refresh          

---(refreshed at 2025-09-16 09:46:08 UTC)---

Total sessions: 0

---(refreshed at 2025-09-16 09:46:13 UTC)---

Total sessions: 0

---(refreshed at 2025-09-16 09:46:18 UTC)---

Session ID: 254, Policy name: to\_web\_server/4, State: Stand-alone, Timeout: 1800, Valid

&nbsp; In: 200.0.13.2/54397 --> 2.2.2.2/22;tcp, Conn Tag: 0x0, If: ge-0/0/2.0, Pkts: 7, Bytes: 1925, 

&nbsp; Out: 10.100.12.2/22 --> 200.0.13.2/54397;tcp, Conn Tag: 0x0, If: ge-0/0/1.0, Pkts: 5, Bytes: 1865, 

Total sessions: 1

---(refreshed at 2025-09-16 09:46:23 UTC)---

Session ID: 254, Policy name: to\_web\_server/4, State: Stand-alone, Timeout: 1794, Valid

&nbsp; In: 200.0.13.2/54397 --> 2.2.2.2/22;tcp, Conn Tag: 0x0, If: ge-0/0/2.0, Pkts: 7, Bytes: 1925, 

&nbsp; Out: 10.100.12.2/22 --> 200.0.13.2/54397;tcp, Conn Tag: 0x0, If: ge-0/0/1.0, Pkts: 5, Bytes: 1865, 

Total sessions: 1

---(\*more 100%)---\[abort]



jcluser@vSRX1> show security nat destination rule DMZ\_SERVER2\_RL    

Destination NAT rule: DMZ\_SERVER2\_RL         Rule-set: DMZ\_SERVER\_RS

&nbsp; Rule-Id                    : 2

&nbsp; Rule position              : 2

&nbsp; From zone                  : untrust

&nbsp;   Destination addresses    : 2.2.2.2         - 2.2.2.2

&nbsp; Action                     : DMZ\_SERVER2\_PL

&nbsp; Translation hits           : 14

&nbsp;   Successful sessions      : 14

&nbsp; Number of sessions         : 0

&nbsp; 

&nbsp; 

jcluser@Host3> telnet port 443 2.2.2.2    

Trying 2.2.2.2...

telnet: connect to address 2.2.2.2: Connection refused

telnet: Unable to connect to remote host

.

.

.

.

jcluser@Host3> telnet port 443 2.2.2.2    

Trying 2.2.2.2...

telnet: connect to address 2.2.2.2: Connection refused

telnet: Unable to connect to remote host

&nbsp; 

&nbsp; 



jcluser@vSRX1> show security nat destination rule DMZ\_SERVER2\_RL    

Destination NAT rule: DMZ\_SERVER2\_RL         Rule-set: DMZ\_SERVER\_RS

&nbsp; Rule-Id                    : 2

&nbsp; Rule position              : 2

&nbsp; From zone                  : untrust

&nbsp;   Destination addresses    : 2.2.2.2         - 2.2.2.2

&nbsp; Action                     : DMZ\_SERVER2\_PL

&nbsp; Translation hits           : 21

&nbsp;   Successful sessions      : 21

&nbsp; Number of sessions         : 0

&nbsp; 

&nbsp; 

jcluser@vSRX1> show configuration security nat  

destination {

&nbsp;   pool DMZ\_SERVER1\_PL {

&nbsp;       address 10.100.11.2/32;

&nbsp;   }

&nbsp;   pool DMZ\_SERVER2\_PL {

&nbsp;       address 10.100.12.2/32;

&nbsp;   }

&nbsp;   rule-set DMZ\_SERVER\_RS {

&nbsp;       from zone untrust;

&nbsp;       rule DMZ\_SERVER1\_RL {

&nbsp;           match {

&nbsp;               destination-address 1.1.1.1/32;

&nbsp;           }

&nbsp;           then {

&nbsp;               destination-nat {

&nbsp;                   pool {

&nbsp;                       DMZ\_SERVER1\_PL;

&nbsp;                   }

&nbsp;               }

&nbsp;           }

&nbsp;       }

&nbsp;       rule DMZ\_SERVER2\_RL {

&nbsp;           match {

&nbsp;               destination-address 2.2.2.2/32;

&nbsp;           }

&nbsp;           then {

&nbsp;               destination-nat {

&nbsp;                   pool {

&nbsp;                       DMZ\_SERVER2\_PL; 

&nbsp;                   }                   

&nbsp;               }                       

&nbsp;           }                           

&nbsp;       }                               

&nbsp;   }                                   

}                                       

&nbsp;        

