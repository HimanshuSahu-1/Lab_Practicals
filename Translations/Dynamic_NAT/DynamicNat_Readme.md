# Dynamic NAT Configuration

## Overview
Dynamic NAT creates a one-to-one temporary mapping between private IP addresses and a pool of public IP addresses. When a private host initiates an outbound connection, it is assigned an available public IP from the pool. The mapping is released when the connection closes.

## Topology
- **Inside Network (Private)**: 192.168.20.0/24 (Connected to interface g0/1)
- **Outside Network (Public)**: Not specified in config but typically the internet-facing side (interface g0/2)

## Configuration Details

### Interface Configuration
```
interface g0/1
  ip nat inside

interface g0/2
  ip nat outside
```

### NAT Pool and ACL Configuration
```
access-list 1 permit 192.168.20.0 0.0.0.255
ip nat pool CCNA 150.1.1.11 150.1.1.20 netmask 255.255.255.0

ip nat inside source list 1 pool CCNA
```

## What This Configuration Does
- **ACL 1**: Selects traffic from the 192.168.20.0/24 network (which needs translation)
- **NAT Pool "CCNA"**: Contains public IP addresses from 150.1.1.11 to 150.1.1.20 (10 addresses)
- **Dynamic Translation**: Each private host that initiates outbound traffic gets assigned one of the available pool IPs
- **Temporary Mapping**: When the connection closes, the IP address is returned to the pool

## Translation Process
1. Private host 192.168.20.X initiates outbound connection
2. Router assigns next available IP from pool (150.1.1.11 - 150.1.1.20)
3. Outbound packets are translated with the assigned public IP
4. Return packets are translated back to the original private IP
5. When connection closes, the public IP is released back to the pool

## Use Cases
1. **Corporate Network**: Translate outbound traffic from multiple internal clients
2. **ISP Services**: Provide internet access to subscriber networks
3. **Limited Public IPs**: Share a small pool of public IPs among many private hosts
4. **Temporary Connections**: When clients need temporary external access

## Configuration Syntax Reference
```
int <inside-interface>
  ip nat inside

int <outside-interface>
  ip nat outside

access-list <ACL-number> permit <network> <wildcard-mask>

ip nat pool <pool-name> <start-ip> <end-ip> netmask <subnet-mask>

ip nat inside source list <ACL-number> pool <pool-name>
```

## Verification Commands
```
show ip nat translations        # View all active translations
show ip nat statistics          # View translation statistics
show access-lists               # Verify ACL configuration
show ip nat pool CCNA           # View pool information
```

## Advantages Over Static NAT
- **Efficient IP Usage**: 10 pool IPs can serve 100+ clients (not all need simultaneous connections)
- **Scalability**: Easy to add more addresses to the pool
- **Flexibility**: Multiple clients can share the same pool
- **Temporary Mappings**: Automatic cleanup when connections close

## Limitations
- **One-to-One**: Still requires one public IP per simultaneous connection
- **Limited Scalability**: If pool exhausts, new connections are rejected
- **No Port Overloading**: Different from PAT which uses ports for multiplexing
