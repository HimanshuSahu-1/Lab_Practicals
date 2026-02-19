# PAT (Port Address Translation) Using NAT Pool

## Overview
Port Address Translation (PAT) using a NAT pool allows multiple private IP addresses to be translated to a pool of public IP addresses using port numbers to distinguish between different connections. The `overload` keyword enables port multiplexing, allowing many clients to share a few public IP addresses.

## Topology
- **Inside Network (Private)**: 192.168.20.0/24 (Connected to interface g0/1)
- **Outside Network (Public)**: Connected to interface g0/2

## Configuration Details

### Step 1: Interface Configuration
```
interface g0/1
  ip nat inside

interface g0/2
  ip nat outside
```

### Step 2: ACL to Select Traffic
```
access-list 1 permit 192.168.20.0 0.0.0.255
```
- Selects all traffic from the private 192.168.20.0/24 network for translation

### Step 3: NAT Pool Configuration
```
ip nat pool CCNA 200.1.1.1 200.1.1.1 netmask 255.255.255.0
```
- Creates a pool named "CCNA" with a single public IP: 200.1.1.1
- Note: Start IP and End IP are the same (200.1.1.1)

### Step 4: PAT with Overload
```
ip nat inside source list 1 pool CCNA overload
```
- Enables PAT using the single IP in the pool
- The `overload` keyword allows port multiplexing
- Multiple clients can share the same public IP by using different ports

## How PAT Works

### Private to Public Translation
```
Private Packet:  192.168.20.10:4000 → 200.1.1.1:80
Public Packet:   200.1.1.1:1001     → 200.1.1.1:80
```

### Mapping Table
The router maintains a translation table:
```
Inside IP:Port      Outside IP:Port
192.168.20.10:4000  200.1.1.1:1001
192.168.20.11:5000  200.1.1.1:1002
192.168.20.12:6000  200.1.1.1:1003
```

## Use Cases
1. **Small Office**: One public IP for entire office of clients
2. **Home Network**: Share a single ISP IP among multiple devices
3. **SOHO (Small Office/Home Office)**: Cost-effective internet sharing
4. **Limited Public IPs**: When few public addresses available
5. **Outbound Access**: Allowing internal clients to initiate connections to internet

## Verification Commands
```
show ip nat translations        # View all translation mappings
show ip nat statistics          # Overall NAT statistics
show ip nat translations verbose # Detailed translation info
clear ip nat translations       # Clear all translations (careful!)
debug ip nat                    # Enable NAT debugging
```

## Translation Table Example
When 3 clients access the internet through a single public IP:
```
Client 1: 192.168.20.10:4000 → 200.1.1.1:1001
Client 2: 192.168.20.11:5000 → 200.1.1.1:1002
Client 3: 192.168.20.12:6000 → 200.1.1.1:1003

Server sees all connections from: 200.1.1.1
```

## Configuration Syntax Options

### Single IP Pool
```
ip nat pool CCNA 200.1.1.1 200.1.1.1 netmask 255.255.255.0
ip nat inside source list 1 pool CCNA overload
```

### Multiple IP Pool with PAT
```
ip nat pool CCNA 200.1.1.1 200.1.1.5 netmask 255.255.255.0
ip nat inside source list 1 pool CCNA overload
```
- Provides 5 public IPs, each can support thousands of clients through port multiplexing
- Even more scalable than single IP PAT

## Advantages
- **Efficient IP Usage**: One public IP can support hundreds of clients
- **Scalability**: Easy to add more IPs to the pool
- **Cost-Effective**: Minimal public IP requirement
- **Transparent to Users**: Applications work normally

## Limitations
- **Port Dependency**: Some applications sensitive to port changes
- **Games/P2P**: Reduced functionality with UDP-based protocols
- **Traceability**: Harder to trace individual users from outside
- **Bidirectional Connections**: Inbound connections difficult (use static NAT for servers)
