---
author: adraffe
comments: false
date: 2013-05-02 16:49:49+00:00
layout: post
link: http://adamraffe.com/2013/05/02/useful-nexus-1000v-vemcmd-commands/
slug: useful-nexus-1000v-vemcmd-commands
title: Useful Nexus 1000V 'vemcmd' Commands
wordpress_id: 492
categories:
- Nexus 1000V
- Virtualisation
---

For anyone using the Nexus 1000V virtual switch, it's sometimes useful to see what is happening directly on the Virtual Ethernet Module (VEM) residing on the host, rather than just having visibility from the Virtual Supervisor Module (VSM). The method to achieve this is via the 'vemcmd' syntax on the ESXi host. Unfortunately these commands are not well documented, so I thought it would be useful to provide a list of some of the more common ones.

**<!-- more -->vemcmd show card **- shows a multitude of information about the VEM, including the domain, switch name, 'slot' number, which control mode you are using (L2 or L3) and much more.

    
    ~ # <strong>vemcmd show card</strong>
    Card UUID type  2: 4225e800-d334-720d-e73e-d3a8d500a201
    Card name: lab.cisco.com
    Switch name: N1KV
    Switch alias: DvsPortset-0
    Switch uuid: b6 e9 19 50 c5 4b 78 ff-c3 f3 a1 13 61 d2 8a d5
    Card domain: 10
    Card slot: 3
    VEM Tunnel Mode: L3 Mode
    L3 Ctrl Index: 49
    L3 Ctrl VLAN: 666
    VEM Control (AIPC) MAC: 00:02:3d:10:0a:02
    VEM Packet (Inband) MAC: 00:02:3d:20:0a:02
    VEM Control Agent (DPA) MAC: 00:02:3d:40:0a:02
    VEM SPAN MAC: 00:02:3d:30:0a:02
    Primary VSM MAC : 00:50:56:99:a7:82
    Primary VSM PKT MAC : 00:50:56:99:94:df
    Primary VSM MGMT MAC : 00:50:56:99:eb:a7
    Standby VSM CTRL MAC : ff:ff:ff:ff:ff:ff
    Management IPv4 address: 192.168.66.55
    Management IPv6 address: 0000:0000:0000:0000:0000:0000:0000:0000
    Primary L3 Control IPv4 address: 192.168.66.57
    Secondary VSM MAC : 00:00:00:00:00:00
    Secondary L3 Control IPv4 address: 0.0.0.0
    Upgrade : Default
    Max physical ports: 32
    Max virtual ports: 300
    Card control VLAN: 1
    Card packet VLAN: 1
    Control type multicast: No
    Card Headless Mode : No
           Processors: 2
      Processor Cores: 2
    Processor Sockets: 1
      Kernel Memory:   8388084
    Port link-up delay: 5s
    Global UUFB: DISABLED
    Heartbeat Set: True
    PC LB Algo: source-mac
    Datapath portset event in progress : no
    Licensed: Yes


**vemcmd  show port **- shows a list of the ports on this VEM and the VMs or VMKs connected to them.**
**

    
    ~ # vemcmd show port
      LTL   VSM Port  Admin Link  State  PC-LTL  SGID  Vem Port  Type
       20     Eth3/4     UP   UP    FWD     561     3    vmnic3
       49      Veth1     UP   UP    FWD       0     3      vmk1
       50      Veth3     UP   UP    FWD       0     3      vmk2  VXLAN
       51      Veth5     UP   UP    FWD       0        Ubuntu-1.eth0
      561        Po1     UP   UP    FWD       0


**vemcmd  show port vlans **- shows a list of the ports on this VEM and the VLANs associated with them.

    
    ~ # vemcmd show port vlans
                              Native  VLAN   Allowed
      LTL   VSM Port  Mode    VLAN    State* Vlans
       20     Eth3/4   T          1   FWD    1,666,800
       49      Veth1   A        666   FWD    666
       50      Veth3   A        800   FWD    800
      561        Po1   T          1   FWD    1,666,800


**vemcmd  show stats **- displays some general statistics (bytes sent & received, etc) associated with each port.

    
    ~ # vemcmd show stats
     LTL  Received        Bytes      Sent        Bytes   Txflood    Rxdrop    Txdrop  Name
     8    853658    163106173   1706853    285044028    853377         0         0
     9   1706853    285044028    853658    163106173    853658         0         0
     10        16          960        42         2520        42         0         0
     12        16          960        42         2520        42         0         0
     15         0            0   3403853    806578120   1682744         0         0
     16      1661       152812         0            0         0         0         0  ar
     20  19831000   2862819019    851081    124246350       653  12789591         2  vmnic3
     49     24916      1494960   2806579    393405034   2781688         0         0  vmk1
     50    813566    119314022    832174    125090824       199         0         0  vmk2
     51    812802     78710878    831405     83473094     18639         0         0  Ubuntu-1.eth0


**vemcmd  show vlan **- displays VLAN information and which port is associated with each one.

    
    ~ # vemcmd show vlan
     Number of valid VLANs: 8
     VLAN 1, vdc 1, swbd 1, hwbd 1, 4 ports
    Portlist:
     VLAN 666, vdc 1, swbd 666, hwbd 7, 3 ports
    Portlist:
     20  vmnic3
     49  vmk1
     561
    VLAN 800, vdc 1, swbd 800, hwbd 8, 3 ports
    Portlist:
     20  vmnic3
     50  vmk2
     561
    VLAN 3968, vdc 1, swbd 3968, hwbd 5, 3 ports
    Portlist:
     1  inban
     5  inband port securit
     11
    VLAN 3969, vdc 1, swbd 3969, hwbd 4, 2 ports
    Portlist:
     8
     9
    VLAN 3970, vdc 1, swbd 3970, hwbd 3, 0 ports
    Portlist:
     VLAN 3971, vdc 1, swbd 3971, hwbd 6, 2 ports
    Portlist:
     14
     15
    <strong>vemcmd  show l2 all </strong>- shows MAC address and other information for all bridge domains and VLANs.
    ~ # vemcmd show l2 all
     Bridge domain    1 brtmax 4096, brtcnt 6, timeout 300
     VLAN 1, swbd 1, ""
     Flags:  P - PVLAN  S - Secure  D - Drop
     Type         MAC Address   LTL   timeout   Flags    PVLAN
     Static   00:02:3d:80:0a:02     6         0
     Static   00:02:3d:40:0a:02    10         0
     Static   00:02:3d:30:0a:02     3         0
     Static   00:02:3d:60:0a:00     5         0
     Static   00:02:3d:20:0a:02    12         0
     Static   00:02:3d:10:0a:02     2         0


**vemcmd  show l2 bd-name <_name_> **- shows MAC address and other information for a specific bridge domain (useful if using VXLANs).

    
    ~ # vemcmd show l2 bd-name test-vm
    Bridge domain    9 brtmax 4096, brtcnt 2, timeout 300
    Segment ID 5000, swbd 4096, "test-vm"
    Flags:  P - PVLAN  S - Secure  D - Drop
           Type         MAC Address   LTL   timeout   Flags    PVLAN    Remote IP
        SwInsta   00:50:56:99:3e:45   561         0                   172.16.1.2
         Static   00:50:56:99:1b:c3    51         0                      0.0.0.0


**vemcmd  show packets **- shows traffic statistics for broadcast / unicast / multicast.

    
    ~ # vemcmd show packets
      LTL   RxUcast   TxUcast   RxMcast   TxMcast   RxBcast   TxBcast   Txflood    Rxdrop    Txdrop  Name
        8    854192    854171         0         0       161    854072    854072         0         0
        9    854171    854192         0         0    854072       161    854353         0         0
       10         0         0         0         0        16        42        42         0         0
       12         0         0         0         0        16        42        42         0         0
       15         0   3406656         0         0         0         0   1684145         0         0
       16      1661         0         0         0         0         0         0         0         0  ar
       20   4269784    836945     60064     14332  15537893       753       653  12800775         2  vmnic3
       49     25007     24969        18      3483       148   2789944   2782799         0         0  vmk1
       50    813658    832692        20         0       718       200       200         0         0  vmk2
       51    813522    813495      2183     15737       766      2890     18639         0         0  Ubuntu-1.eth0
      561   4269784    836945     60064     14332  15537893       753       653  12800775         2


**vemcmd  show arp all **- shows arp information (could be useful to show VTEP information).

    
    ~ # vemcmd show arp all
    Flags: D-Dynamic S-Static d-Delete s-Sticky
           P-Proxy B-Public C-Create X-Exclusive
    VLAN/SEGID     IP Address           MAC Address          Flags            Expiry
    800            172.16.1.1           00:50:56:68:e9:94    D                550
    800            172.16.1.2           00:50:56:60:9a:7c    D                550


**vemcmd  show vxlan interfaces **- shows which interfaces (i.e. VMKs and their associated vEths) are configured as VXLAN interfaces.

    
    ~ # vemcmd show vxlan interfaces
    LTL     VSM Port      IP       Seconds since Last   Vem Port
                                   IGMP Query Received
    (* = IGMP Join Interface/Designated VTEP)
    --------------------------------------------------------------
     50        Veth3    172.16.1.1    856205             vmk2         *


That will do for now - you can actually get a full list of the commands available by doing 'vem-support all' on the VEM, but the ones above are some of the most useful. Hope this helps!
