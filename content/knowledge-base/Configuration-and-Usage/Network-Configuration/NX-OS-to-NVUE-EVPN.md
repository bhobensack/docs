---
title: NX-OS to NVUE EVPN Configuration
weight: 300
---

Cumulus Linux version 4.4 introduces a new CLI called [NVUE]({{<ref "/cumulus-linux-51/System-Configuration/NVIDIA-User-Experience-NVUE/" >}}); a complete object model for Cumulus Linux. NVUE makes translating configurations from one vendor to another much more reliable the first time you use Cumulus Linux and across Cumulus Linux versions.

This KB article describes how to take a basic NX-OS configuration for EVPN and translate it to NVUE. The example confiuration derives from [this Cisco Configuration Example](https://www.cisco.com/c/en/us/support/docs/ip/border-gateway-protocol-bgp/200952-Configuration-and-Verification-VXLAN-wit.html).

| NX-OS Command | NVUE Command | Comments |
| -----         | -----        | -----    |
|`nv overlay evpn`|  _none_ | Cumulus Linux does not require *features* to be explicitly enabled. |
|`feature ospf`|   _none_ | |
|`feature bgp`|   _none_ | |
|`feature pim`|   _none_ |  |
|`feature interface-vlan`|   _none_ |  |
|`feature vn-segment-vlan-based`|   _none_ |  |
|`feature lacp`|   _none_ |  |
|`feature vpc`|   _none_ |  |
|`feature nv overlay`|   _none_ |  |
|`fabric forwarding anycast-gateway-mac 0001.0001.0001`| `nv set system global anycast-mac 44:38:39:BE:EF:AA` | |
|`ip pim rp-address 192.168.9.9 group-list 224.0.0.0/4`| _none_ | NVIDIA recommends you use Head End Replication for EVPN, removing the need for PIM. Scale-out deployments support {{<kb_link latest="cl" url="Network-Virtualization/Ethernet-Virtual-Private-Network-EVPN/EVPN-PIM.md" text="PIM Replication" >}}.|
|`ip pim ssm range 232.0.0.0/8`| _none_ |
|`vlan 1,10,20,30,40`| `nv set bridge domain br_default vlan 30,40` | {{<kb_link latest="cl" url="Layer-2/Ethernet-Bridging-VLANs/VLAN-aware-Bridge-Mode.md" text="Ethernet Bridging" >}}
|`vlan 10`| _none_ | The layer 3 VNI does not require a unique VLAN interface. |
|` name L3-VNI-VLAN-10`|  _none_ | |
|` vn-segment 10000010`|  _none_ | |
|`vlan 30`| `nv set bridge domain br_default vlan 30 vni 10000030` | |
|` vn-segment 10000030`| _none_ | A single command with NVUE defines the `vn-segment` and `vlan`. |
|`vlan 40`| `nv set bridge domain br_default vlan 40 vni 10000040` | |
|` vn-segment 10000040`| _none_ | |
|`vpc domain 2`| `nv set mlag backup 10.197.204.103` | Cumulus Linux {{<kb_link latest="cl" url="Layer-2/Multi-Chassis-Link-Aggregation-MLAG.md" text="Multi-Chassis Link Aggregation - MLAG" >}} uses remote and local peering to ensure uptime.|
|` peer-keepalive destination 10.197.204.103`| `nv set mlag peer-ip linklocal` | You can determine the local peer IP automatically with link-local addresses. |
|`interface Vlan10`| _none_ | The layer 3 VNI does not require a unique VLAN interface. |
|` no shutdown`| _none_ | |
|` vrf member EVPN-L3-VNI-VLAN-10`| _none_ | |
|` ip forward`| _none_ | |
|`interface Vlan30`| | |
|` no shutdown`| _none_ | Interfaces are `no shut` by default. |
|` vrf member EVPN-L3-VNI-VLAN-10`| `nv set interface vlan30 ip vrf EVPN-L3-VNI-VLAN-10` | |
|` ip address 172.16.30.1/24`| `nv set interface vlan30 ip address 172.16.30.1/24` | |
|` fabric forwarding mode anycast-gateway`| _none_ | You do not speceify an explicit command for this setting in NVUE. |
|`interface Vlan40 `| | |
|` no shutdown`| _none_ | |
|` vrf member EVPN-L3-VNI-VLAN-10`| `nv set interface vlan40 ip vrf EVPN-L3-VNI-VLAN-10` | |
|` ip address 172.16.40.1/24`| `nv set interface vlan40 ip address 172.16.40.1/24` | |
|` fabric forwarding mode anycast-gateway`| _none_ | | 
|`interface port-channel2 `| `nv set interface bond2 bond member swp1` | This example uses `swp1` instead of NX-OS port `e1/13`. | 
|` switchport mode trunk`| `nv set interface bond2 bridge domain br_default vlan all` | `br_default` is the name of the default bridge. |
|` vpc 2`| `nv set interface bond2 bond mlag id 2`| |
|`interface port-channel34`|`nv set interface peerlink bond member swp49` | This example uses `swp49` instead of NX-OS port `e1/1`. |
|` switchport mode trunk`|  _none_ | The peerlink is a trunk port by default. | 
|` spanning-tree port type network`| _none_ | Enabled by default. | 
|` vpc peer-link`| _none_ | You do this with the name `peerlink`. |
|`interface nve1`| _none_ | NVUE does not have an explicit interface. |
|` no shutdown`|  _none_ | | 
|` source-interface loopback2`| `nv set nve vxlan source address 192.168.33.33` | You configure NVUE interface settings globally with `nv set nve vxlan` commands.|
|` host-reachability protocol bgp`|  _none_ | |
|` member vni 10000010 associate-vrf`| `nv set vrf EVPN-L3-VNI-VLAN-10 evpn vni 10000010` | | 
|` member vni 10000030`| _none_ | You only need to associate the VRF VNI. | 
|` suppress-arp`| `nv set nve vxlan arp-nd-suppress on` | | 
|` mcast-group 239.1.1.10`| _none_ | NVIDIA recommends you use Head end Replication. |
|` member vni 10000040`|   _none_ | |
|` suppress-arp`|   _none_ | |
|` mcast-group 239.1.1.20`|   _none_ | |
|`interface Ethernet1/1`| _none_ | | 
|` switchport mode trunk`| _none_ | You defined the interface when you created the bond earlier. |
|` channel-group 34 mode active`|  _none_ | |
|`interface Ethernet1/2`|   _none_ | |
|` no switchport`| _none_ | Interfaces are layer 3 by default. |
|` ip address 192.168.39.3/24`| _none_ | NVIDIA recommends {{<kb_link latest="cl" url="Layer-3/Border-Gateway-Protocol-BGP/Basic-BGP-Configuration.md#bgp-unnumbered" text="BGP Unnumbered" >}} for the underlay, removing the need for an IP address. |
|` ip router ospf UNDERLAY area 0.0.0.0`| _none_ | NVIDIA recommends BGP Unnumbered. | 
|` ip pim sparse-mode`| _none_ | NVIDIA recommends Head End Replication. |
|` no shutdown`| _none_ | | 
|`interface Ethernet1/13`| _none_ | This example uses port `swp1` instead of `e1/13`.| 
|` switchport mode trunk`| _none_ | Port configuration is part of the bond definition. | 
|` channel-group 2 mode active`| _none_ | | 
|`interface loopback2`| `nv set interface lo ip address 192.168.33.33/32` | Cumulus Linux uses a single loopback `lo`. |
|` ip address 192.168.33.33/32`| _none_ | Defined with the creation of the loopback interface. | 
|` ip address 192.168.33.34/32 secondary`| `nv set nve vxlan mlag shared-address 192.168.33.34` | | 
|` ip router ospf UNDERLAY area 0.0.0.0`| _none_ | NVIDIA recommends using BGP for both the underlay and overlay. |
|`router bgp 65000`| `nv set router bgp autonomous-system  65000` | | 
|` address-family ipv4 unicast`| _none_ | | 
|` address-family l2vpn evpn`| _none_ | | 
|` neighbor 192.168.9.9 remote-as 100`| `nv set vrf default router bgp neighbor swp51 remote-as external` | This command uses eBGP unnumbered instead of IP based peering. |
|` remote-as 65000`| _none_ | This command combines with the peer command above. |
|` update-source loopback2`| _none_ | BGP unnumbered uses the interface instead of a loopback source. | 
|` address-family ipv4 unicast`| _none_ | | 
|` address-family l2vpn evpn`| `nv set vrf default router bgp neighbor swp51 address-family l2vpn-evpn enable on` | | 
|` send-community extended`| _none_ | Enabled by default. | 
|` vrf EVPN-L3-VNI-VLAN-10`| _none_ | You manage this through an earlier `nv set vrf` command. |
|` address-family ipv4 unicast`| _none_ | | 
|` advertise l2vpn evpn`| _none_ | | 
|`evpn`| _none_ | | 
|` vni 10000030 l2`| _none_ | |  
|` rd auto`| _none_ | | 
|` route-target import auto`| _none_ | | 
|` route-target export auto`| _none_ | | 
|` vni 10000040 l2`| _none_ | | 
|` rd auto`| _none_ | | 
|` route-target import auto`| _none_ | | 
|` route-target export auto `| _none_ | | 

<details>
<summary>Complete NVUE Configuration</summary>

``` 
nv set system global anycast-mac 44:38:39:BE:EF:AA
nv set bridge domain br_default vlan 30,40
nv set bridge domain br_default vlan 30 vni 10000030
nv set bridge domain br_default vlan 40 vni 10000040
nv set mlag backup 10.197.204.103
nv set mlag peer-ip linklocal
nv set interface vlan30 ip vrf EVPN-L3-VNI-VLAN-10
nv set interface vlan30 ip address 172.16.30.1/24
nv set interface vlan40 ip vrf EVPN-L3-VNI-VLAN-10
nv set interface vlan40 ip address 172.16.40.1/24
nv set interface bond2 bond member swp1
nv set interface bond2 bridge domain br_default vlan all
nv set interface bond2 bond mlag id 2
nv set interface peerlink bond member swp49
nv set nve vxlan source address 192.168.33.33
nv set vrf EVPN-L3-VNI-VLAN-10 evpn vni 10000010
nv set nve vxlan arp-nd-suppress on
nv set interface lo ip address 192.168.33.33/32
nv set nve vxlan mlag shared-address 192.168.33.34
nv set router bgp autonomous-system 65000
nv set vrf default router bgp neighbor swp51 remote-as external
nv set vrf default router bgp neighbor swp51 address-family l2vpn-evpn enable on
```

</details>
