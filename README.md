# OSPF-Dynamic-Routing-DHCP-and-Bandwidth-Configuration-Lab
# OSPF Dynamic Routing, DHCP and Bandwidth Configuration Lab Report

## 1. Aim
To configure Dynamic Routing using OSPF protocol on a multi-router network using subnet `10.0.0.0/8`, implement DHCP services, and configure bandwidth to control OSPF path selection.

## 2. Objectives
- Divide `10.0.0.0/8` network into 8 subnets.
- Configure IP addressing on 5 routers.
- Implement OSPF dynamic routing.
- Configure DHCP for automatic IP allocation.
- Apply bandwidth to influence OSPF cost.
- Verify connectivity between two LAN networks.

## 3. Network Topology Description
The topology consists of:
- **5 Routers** (R0, R1, R2, R3, R4)
- **2 Switches**
- **6 PCs** (3 in each LAN)

Two possible paths exist between the LANs:
- **Upper Path** → R0–R1–R2–R3 (10 Mbps)
- **Lower Path** → R0–R4–R3 (5 Mbps)

OSPF should automatically choose the 10 Mbps upper path as the primary route.

## 4. IP Addressing Scheme
The network `10.0.0.0/8` is divided into 8 subnets using a `/11` mask (`255.224.0.0`).

| Link | Network | Use |
| :--- | :--- | :--- |
| **R0 – R1** | `10.0.0.0/11` | Inter-router link |
| **R1 – R2** | `10.32.0.0/11` | Inter-router link |
| **R2 – R3** | `10.64.0.0/11` | Inter-router link |
| **R0 – R4** | `10.96.0.0/11` | Inter-router link |
| **R4 – R3** | `10.128.0.0/11` | Inter-router link |
| **Left LAN** | `10.160.0.0/11` | End-user devices (R0) |
| **Right LAN** | `10.192.0.0/11` | End-user devices (R3) |
| **Reserved** | `10.224.0.0/11` | Unused |

*Subnet Mask for all interfaces: `255.224.0.0`*

## 5. Router Interface Configurations

### Router0 (R0)
```text
enable
configure terminal
hostname R0

interface fa0/0   ! Link to R1
ip address 10.0.0.1 255.224.0.0
no shutdown
bandwidth 10000

interface fa0/1   ! Link to R4
ip address 10.96.0.1 255.224.0.0
no shutdown
bandwidth 5000

interface fa1/0   ! Left LAN
ip address 10.160.0.1 255.224.0.0
no shutdown
exit
```

### Router1 (R1)
```text
enable
configure terminal
hostname R1

interface fa0/1   ! Link to R0
ip address 10.0.0.2 255.224.0.0
no shutdown
bandwidth 10000

interface fa0/0   ! Link to R2
ip address 10.32.0.1 255.224.0.0
no shutdown
bandwidth 10000
exit
```

### Router2 (R2)
```text
enable
configure terminal
hostname R2

interface fa0/1   ! Link to R1
ip address 10.32.0.2 255.224.0.0
no shutdown
bandwidth 10000

interface fa0/0   ! Link to R3
ip address 10.64.0.1 255.224.0.0
no shutdown
bandwidth 10000
exit
```

### Router4 (R4)
```text
enable
configure terminal
hostname R4

interface fa0/0   ! Link to R0
ip address 10.96.0.2 255.224.0.0
no shutdown
bandwidth 5000

interface fa0/1   ! Link to R3
ip address 10.128.0.1 255.224.0.0
no shutdown
bandwidth 5000
exit
```

### Router3 (R3)
```text
enable
configure terminal
hostname R3

interface fa0/1   ! Link to R2
ip address 10.64.0.2 255.224.0.0
no shutdown
bandwidth 10000

interface fa0/0   ! Link to R4
ip address 10.128.0.1 255.224.0.0
no shutdown
bandwidth 5000

interface fa1/0   ! Right LAN
ip address 10.192.0.1 255.224.0.0
no shutdown
exit
```

## 6. OSPF Configuration

OSPF is configured on all routers using Process ID `1` and Area `0` (Backbone Area). The wildcard mask corresponding to a `/11` subnet (`255.224.0.0`) is `0.31.255.255`.

### Router0
```text
configure terminal
router ospf 1
network 10.0.0.0 0.31.255.255 area 0
network 10.96.0.0 0.31.255.255 area 0
network 10.160.0.0 0.31.255.255 area 0
end
```

### Router1
```text
configure terminal
router ospf 1
network 10.0.0.0 0.31.255.255 area 0
network 10.32.0.0 0.31.255.255 area 0
end
```

### Router2
```text
configure terminal
router ospf 1
network 10.32.0.0 0.31.255.255 area 0
network 10.64.0.0 0.31.255.255 area 0
end
```

### Router4
```text
configure terminal
router ospf 1
network 10.96.0.0 0.31.255.255 area 0
network 10.128.0.0 0.31.255.255 area 0
end
```

### Router3
```text
configure terminal
router ospf 1
network 10.64.0.0 0.31.255.255 area 0
network 10.128.0.0 0.31.255.255 area 0
network 10.192.0.0 0.31.255.255 area 0
end
```

*(Note: While `network 10.0.0.0 0.255.255.255 area 0` works as a uniform catch-all command, defining specific subnets with `0.31.255.255` is best practice and deployed here).*

## 7. DHCP Configuration

### Router0 (Left LAN DHCP)
Assigns IPs automatically to the Left LAN (PC0–PC2).
```text
configure terminal
ip dhcp pool P1
network 10.160.0.0 255.224.0.0
default-router 10.160.0.1
exit
```

### Router3 (Right LAN DHCP)
Assigns IPs automatically to the Right LAN (PC3–PC5).
```text
configure terminal
ip dhcp pool P2
network 10.192.0.0 255.224.0.0
default-router 10.192.0.1
exit
```

## 8. Bandwidth Configuration & OSPF Cost
OSPF uses interface bandwidth to calculate the metric (cost) to a destination. Links with higher bandwidth result in lower OSPF path costs.

| Path | Configured Bandwidth | OSPF Cost Result | Selection |
| :--- | :--- | :--- | :--- |
| **R0 → R1 → R2 → R3** | 10,000 Kbps (10 Mbps) | Low | **PRIMARY PATH ⭐** |
| **R0 → R4 → R3** | 5,000 Kbps (5 Mbps) | High | **BACKUP PATH** |

**Crucial detail:** Because of this explicit bandwidth configuration applied to interfaces, OSPF prefers traversing three hops (R0-R1-R2-R3) because its cumulative cost is lower than traversing the two-hop path (R0-R4-R3).

## 9. Verification Commands

**To verify OSPF Routing:**
```text
show ip route
show ip route ospf
show ip ospf neighbor
show ip ospf database
```


**To test End-to-End Connectivity:**
From a Command Prompt on PC0 (Left LAN), ping a PC dynamically assigned in the Right LAN:
```text
ping 10.192.0.11
```

## 10. Result
- OSPF dynamic routing successfully configured and operational across 5 routers.
- DHCP pools correctly established, efficiently leasing IP addresses to endpoint devices.
- Bandwidth manipulation successfully influenced OSPF cost calculations, steering traffic reliably over the specified 10 Mbps primary route instead of the 5 Mbps backup route.
- Full end-to-end network connectivity validated via ICMP echoes.

## 11. Conclusion
This lab practically demonstrated OSPF's sophisticated capability to dynamically learn routes and determine the most optimal path strictly based on interface bandwidth metrics rather than just hop count. Furthermore, it showcased how implementing DHCP directly on a routing platform simplifies endpoint administration across segregated LANs, culminating in a highly robust, scalable, and responsive network infrastructure.
