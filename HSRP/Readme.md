# HSRP Lab Setup

## Overview
This lab demonstrates **Hot Standby Router Protocol (HSRP)** configuration for router redundancy and high availability. The setup consists of three routers: one internet-facing router and two customer-facing routers with HSRP enabled.

---

## Network Topology

```
                    ┌─────────────┐
                    │  Internet   │
                    │  (Gateway)  │
                    └──────┬──────┘
                           │
           ┌───────────────┼───────────────┐
           │               │               │
      (11.1.1.0)      (12.1.1.0)     (8.8.8.8)
           │               │               │
      ┌────▼────┐     ┌────▼────┐         │
      │    R1   │     │    R2   │         │
      │10.1.1.1 │     │10.1.1.2 │         │
      └────┬────┘     └────┬────┘         │
           │               │               │
      (10.1.1.0)  ◄─── HSRP VIP: 10.1.1.254
           │               │
           └───────┬───────┘
                   │
              ┌────▼────┐
              │  Clients │
              └─────────┘
```

---

## Devices Configuration

### 1. Internet Router
- **Hostname**: Internet
- **Interface g0/0/0**: 11.1.1.10/24 (Connection to R1)
- **Interface g0/0/1**: 12.1.1.10/24 (Connection to R2)
- **Interface L1**: 8.8.8.8/24 (Loopback - Simulated ISP DNS)
- **Routing**: EIGRP AS 100 (Networks: 12.1.1.0, 11.1.1.0, 8.8.8.8)

### 2. R1 Router (Primary)
- **Hostname**: R1
- **Interface g0/0/0**: 10.1.1.1/24 (Client-facing side)
- **Interface g0/0/1**: 11.1.1.1/24 (Connection to Internet router)
- **Routing**: EIGRP AS 100 (Networks: 10.1.1.0, 11.1.1.0)
- **HSRP Configuration**:
  - Group: 1
  - Virtual IP: 10.1.1.254
  - Priority: 110 (Higher - becomes Active)
  - Preemption: Enabled (will take over if it recovers)

### 3. R2 Router (Secondary)
- **Hostname**: R2
- **Interface g0/0/0**: 10.1.1.2/24 (Client-facing side)
- **Interface g0/0/1**: 12.1.1.2/24 (Connection to Internet router)
- **Routing**: EIGRP AS 100 (Networks: 10.1.1.0, 12.1.1.0)
- **HSRP Configuration**:
  - Group: 1
  - Virtual IP: 10.1.1.254
  - Priority: 100 (Default - Standby role)

---

## HSRP Details

| Parameter | R1 (Primary) | R2 (Standby) |
|-----------|-------------|-------------|
| Priority | 110 | 100 |
| Preemption | Enabled | Disabled |
| Virtual IP | 10.1.1.254 | 10.1.1.254 |
| HSRP Group | 1 | 1 |
| Role | **Active** | Standby |

### Key Features:
- **Clients** use the Virtual IP (10.1.1.254) as their default gateway
- **R1** is the Active router because it has higher priority (110)
- **Preemption is enabled** on R1, so if it fails and comes back online, it will automatically take over the Active role
- **R2 serves as backup** and takes over if R1 fails

---

## Verification Commands

To verify the HSRP configuration, use these commands:

```
show standby brief
show standby detailed
show standby all
ping 10.1.1.254
```

---

## Lab Objectives
1. ✓ Configure HSRP on both routers for redundancy
2. ✓ Set up proper priority and preemption
3. ✓ Verify Active/Standby roles
4. ✓ Test failover scenarios
5. ✓ Ensure client connectivity via virtual IP


