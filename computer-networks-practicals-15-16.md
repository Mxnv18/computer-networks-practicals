# Computer Networks Practical Documentation

**Subject:** Computer Networks  
**Practical:** Dynamic routing with RIPv2 and single-area OSPFv2  
**Student:** _Add your name and roll number_  
**Tool:** Cisco Packet Tracer

> This is an original working template using a different VLSM address plan from the reference material. Build and test the topology in Packet Tracer, then replace the verification placeholders with your own command output and screenshots before submission.

---

# Practical 15: RIPv2 Dynamic Routing

## 1. Objective

Configure and verify RIPv2 on three routers. The topology uses three `/27` LANs and two `/30` point-to-point serial networks to practice classless routing with VLSM.

## 2. Topology

Connect the devices as follows:

- PC0 — R0 `FastEthernet2/0`
- R0 `Serial0/0` — R1 `Serial1/0`
- PC2 — R1 `FastEthernet2/0`
- R1 `Serial0/0` — R2 `Serial1/0`
- PC4 — R2 `FastEthernet2/0`

Use serial DCE clocking on R0 `Serial0/0` and R1 `Serial0/0` if those ends are DCE in your Packet Tracer file. Set the clock rate on the DCE end only. Interface names may differ by router model; check the actual device before pasting commands.

## 3. Addressing Plan

| Device | Interface | IP address | Mask | Default gateway |
|---|---|---:|---:|---:|
| R0 | FastEthernet2/0 | 10.15.0.1 | 255.255.255.224 (`/27`) | — |
| R0 | Serial0/0 | 10.15.0.97 | 255.255.255.252 (`/30`) | — |
| R1 | FastEthernet2/0 | 10.15.0.33 | 255.255.255.224 (`/27`) | — |
| R1 | Serial1/0 | 10.15.0.98 | 255.255.255.252 (`/30`) | — |
| R1 | Serial0/0 | 10.15.0.101 | 255.255.255.252 (`/30`) | — |
| R2 | FastEthernet2/0 | 10.15.0.65 | 255.255.255.224 (`/27`) | — |
| R2 | Serial1/0 | 10.15.0.102 | 255.255.255.252 (`/30`) | — |
| PC0 | FastEthernet0 | 10.15.0.2 | 255.255.255.224 (`/27`) | 10.15.0.1 |
| PC2 | FastEthernet0 | 10.15.0.34 | 255.255.255.224 (`/27`) | 10.15.0.33 |
| PC4 | FastEthernet0 | 10.15.0.66 | 255.255.255.224 (`/27`) | 10.15.0.65 |

Subnet ranges: LANs `10.15.0.0/27`, `10.15.0.32/27`, `10.15.0.64/27`; serial links `10.15.0.96/30` and `10.15.0.100/30`.

## 4. Router Configurations

### R0

```text
enable
configure terminal
hostname R0
interface fastEthernet2/0
 ip address 10.15.0.1 255.255.255.224
 no shutdown
 exit
interface serial0/0
 ip address 10.15.0.97 255.255.255.252
 clock rate 64000
 no shutdown
 exit
router rip
 version 2
 no auto-summary
 network 10.0.0.0
end
write memory
```

If R0 `Serial0/0` is DTE in your topology, remove the `clock rate` command there and apply it to the connected DCE end.

### R1

```text
enable
configure terminal
hostname R1
interface fastEthernet2/0
 ip address 10.15.0.33 255.255.255.224
 no shutdown
 exit
interface serial1/0
 ip address 10.15.0.98 255.255.255.252
 no shutdown
 exit
interface serial0/0
 ip address 10.15.0.101 255.255.255.252
 clock rate 64000
 no shutdown
 exit
router rip
 version 2
 no auto-summary
 network 10.0.0.0
end
write memory
```

### R2

```text
enable
configure terminal
hostname R2
interface fastEthernet2/0
 ip address 10.15.0.65 255.255.255.224
 no shutdown
 exit
interface serial1/0
 ip address 10.15.0.102 255.255.255.252
 no shutdown
 exit
router rip
 version 2
 no auto-summary
 network 10.0.0.0
end
write memory
```

## 5. Verification

On each router, use:

```text
show ip interface brief
show ip protocols
show ip route
```

On PC0, test the other LANs after the routes have converged:

```text
ping 10.15.0.34
ping 10.15.0.66
```

**Record your results:** Paste your own `show ip route` output and ping results here. A successful remote route should be marked `R`; directly connected networks are marked `C`.

_Add your Packet Tracer topology and successful ping screenshots here._

---

# Practical 16: Single-Area OSPFv2

## 1. Objective

Replace RIPv2 with OSPFv2 in Area 0. Configure a unique router ID on each router and advertise the same LAN and serial subnets using wildcard masks.

## 2. OSPF Network Statements

| Router | Network | Wildcard | Area | Router ID |
|---|---|---|---:|---:|
| R0 | 10.15.0.0 | 0.0.0.31 | 0 | 1.1.1.1 |
| R0 | 10.15.0.96 | 0.0.0.3 | 0 | 1.1.1.1 |
| R1 | 10.15.0.32 | 0.0.0.31 | 0 | 2.2.2.2 |
| R1 | 10.15.0.96 | 0.0.0.3 | 0 | 2.2.2.2 |
| R1 | 10.15.0.100 | 0.0.0.3 | 0 | 2.2.2.2 |
| R2 | 10.15.0.64 | 0.0.0.31 | 0 | 3.3.3.3 |
| R2 | 10.15.0.100 | 0.0.0.3 | 0 | 3.3.3.3 |

## 3. Router Configurations

The interface addressing from Practical 15 stays in place. Remove the RIP process and configure OSPF as shown.

### R0

```text
enable
configure terminal
no router rip
router ospf 1
 router-id 1.1.1.1
 network 10.15.0.0 0.0.0.31 area 0
 network 10.15.0.96 0.0.0.3 area 0
end
write memory
```

### R1

```text
enable
configure terminal
no router rip
router ospf 1
 router-id 2.2.2.2
 network 10.15.0.32 0.0.0.31 area 0
 network 10.15.0.96 0.0.0.3 area 0
 network 10.15.0.100 0.0.0.3 area 0
end
write memory
```

### R2

```text
enable
configure terminal
no router rip
router ospf 1
 router-id 3.3.3.3
 network 10.15.0.64 0.0.0.31 area 0
 network 10.15.0.100 0.0.0.3 area 0
end
write memory
```

If OSPF was already running when you changed the router ID, use `clear ip ospf process` and confirm the prompt in Packet Tracer, or restart the router, so the new ID takes effect.

## 4. Verification

Use these commands on the routers:

```text
show ip ospf neighbor
show ip route
show ip protocols
```

From PC0, test both remote PCs:

```text
ping 10.15.0.34
ping 10.15.0.66
```

**Record your results:** Paste your own neighbor table, routing table, and ping results here. Learned OSPF routes are marked `O`, while directly connected routes are marked `C`.

_Add your Packet Tracer topology and successful ping screenshots here._

## Conclusion

Summarize what you observed when using RIPv2 and OSPFv2. Include whether the remote PCs could communicate, which route codes appeared in `show ip route`, and any configuration issue you corrected. Base this section on your own Packet Tracer results.
