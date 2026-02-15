# Static NAT Configuration

## Overview
Static NAT (Network Address Translation) creates a one-to-one permanent mapping between a private IP address and a public IP address. This is commonly used for hosting servers that need to be accessible from the internet on a fixed public address.

## Topology
- **Inside Network (Private)**: 192.168.10.0/24 (Connected to interface g0/0)
- **Inside Network 2**: 192.168.20.0/24 (Connected to interface g0/1)
- **Outside Network (Public)**: 11.1.1.0/24 (Connected to interface g0/2)

## Configuration Details

### Interface Configuration
```
interface g0/0
  ip address 192.168.10.1 255.255.255.0
  no shutdown

interface g0/1
  ip address 192.168.20.1 255.255.255.0
  no shutdown

interface g0/2
  ip address 11.1.1.1 255.255.255.0
  no shutdown
```

### Static NAT Configuration
```
interface g0/0
  ip nat inside

interface g0/2
  ip nat outside

ip nat inside source static 192.168.10.10 100.1.1.10
```

## What This Configuration Does
- **Private IP**: 192.168.10.10 (inside network)
- **Public IP**: 100.1.1.10 (outside network)
- Any traffic from 192.168.10.10 is translated to 100.1.1.10 when leaving the router
- Any traffic destined to 100.1.1.10 is translated back to 192.168.10.10 when entering the network

## Use Cases
1. **Web Server Hosting**: Expose an internal web server to the internet
2. **Mail Server**: Make an internal mail server accessible from external networks
3. **FTP Server**: Provide external access to file servers
4. **Permanent Public Address**: When a server needs a fixed public IP address

## Verification Commands
```
show ip nat translations        # View all active NAT translations
show ip nat statistics          # View NAT statistics
show ip route                   # Verify routing
ping 100.1.1.10                # Test connectivity to translated address
```

## Key Differences from Dynamic NAT
- **Static**: One-to-one, permanent mapping (1 private IP = 1 public IP)
- **Dynamic NAT**: Many-to-many, temporary mappings as needed
- **Static**: Better for servers that need consistent external addresses
- **Dynamic**: Better for client machines with temporary outbound connections
