# PAT (Port Address Translation) Using Interface IP

## Overview
Port Address Translation (PAT) using an interface IP is the second method of implementing PAT. Instead of using a pre-configured NAT pool, this method translates traffic using the IP address of the router's outside interface directly. This is the most common PAT configuration and is used in home routers and small office networks.

## Topology
- **Inside Network (Private)**: 192.168.20.0/24 (Connected to interface g0/1)
- **Outside Interface**: g0/2 (Interface IP will be used for translation)

## Configuration Details

### Step 1: Interface Configuration
```
interface g0/1
  ip nat inside

interface g0/2
  ip nat outside
```
- Designates g0/1 as the inside (private) interface
- Designates g0/2 as the outside (public) interface

### Step 2: ACL to Select Traffic for Translation
```
access-list 1 permit 192.168.20.0 0.0.0.255
```
- Selects all traffic originating from 192.168.20.0/24 network
- Only traffic matching this ACL will be translated

### Step 3: PAT Using Interface IP
```
ip nat inside source list 1 interface g0/2 overload
```
- Enables PAT using the IP address configured on interface g0/2
- No separate NAT pool needed
- The `overload` keyword enables port multiplexing
- All clients share a single public IP (the interface IP)

## How This Configuration Works

### Dynamic IP Scenario
If interface g0/2 has IP address 200.1.1.1:
```
Private Client:  192.168.20.10:4000
↓ (Translation)
Outside Network: 200.1.1.1:1001
```

### DHCP/Dynamic IP Support
- **Key Advantage**: If g0/2's IP changes (e.g., DHCP), PAT automatically uses the new IP
- No configuration changes needed
- Perfect for ISP connections with dynamic IP addressing

## Comparison: Interface IP vs. NAT Pool

| Feature | Interface IP | NAT Pool |
|---------|-------------|----------|
| Public IP Source | Interface IP (dynamic support) | Pre-configured pool |
| Configuration | Simpler, fewer commands | More complex |
| Flexibility | Adapts to IP changes | Fixed IPs |
| Multiple IPs | Single IP only | Can use multiple IPs |
| ISP Connection | Better for dynamic ISP IPs | Better for static IPs |

## Use Cases
1. **Home Network**: Share single ISP-provided IP
2. **SOHO Offices**: When using DHCP from ISP
3. **Small Branch Offices**: Simple, straightforward setup
4. **ISP Dynamic IPs**: When ISP assigns IPs via DHCP
5. **Quick Deployment**: No pool planning needed

## Configuration Steps Summary
```
# Enable NAT on interfaces
int g0/1
  ip nat inside

int g0/2
  ip nat outside
exit

# Define traffic to translate
access-list 1 permit 192.168.20.0 0.0.0.255

# Enable PAT using interface IP
ip nat inside source list 1 interface g0/2 overload

# Verify
do show ip nat translations
```

## Verification Commands
```
show ip nat translations              # View active translations
show ip nat translations verbose      # Detailed view with timers
show ip nat statistics                # Overall statistics
show interface g0/2                   # Verify outside interface IP
show access-lists 1                   # Verify ACL configuration
clear ip nat translations all         # Clear all translations
```

## Translation Examples

### Single Client
```
Private:  192.168.20.10:4000 → www.example.com:80
Public:   200.1.1.1:1001 → www.example.com:80
```

### Multiple Clients (Port Multiplexing)
```
Client 1: 192.168.20.10:4000 → www.example.com:80
          ↓ Translates to
          200.1.1.1:1001 → www.example.com:80

Client 2: 192.168.20.11:5000 → www.example.com:80
          ↓ Translates to
          200.1.1.1:1002 → www.example.com:80

Client 3: 192.168.20.12:6000 → www.example.com:80
          ↓ Translates to
          200.1.1.1:1003 → www.example.com:80
```

## Key Differences from "PAT Using NAT Pool"

### Interface IP Method (This Configuration)
```
ip nat inside source list 1 interface g0/2 overload
```
- Uses the interface IP directly
- Works with dynamic IPs from DHCP
- Simpler configuration
- Only one public IP possible

### NAT Pool Method (Other Configuration)
```
ip nat pool CCNA 200.1.1.1 200.1.1.1 netmask 255.255.255.0
ip nat inside source list 1 pool CCNA overload
```
- Uses a pre-configured IP pool
- Works with static IPs
- More flexible for multiple IPs
- More explicit configuration

## Advantages of Interface IP Method
1. **Dynamic IP Support**: Automatically adapts to DHCP changes
2. **Simplicity**: No pool planning or configuration
3. **Best for ISPs**: Perfect for ISP-assigned dynamic IPs
4. **Minimal Configuration**: Fewer commands needed
5. **Automatic Failover**: If you add redundancy, adapts automatically

## Limitations
1. **Single Public IP**: Only interface IP can be used
2. **Port Scalability**: Limited by port number range for multiplexing
3. **No Load Balancing**: All traffic uses same interface
4. **Interface Dependent**: If interface goes down, all translations fail

## Real-World Example: Home Router
Most home routers use this exact configuration:
```
interface LAN
  ip nat inside

interface WAN
  ip nat outside

access-list 1 permit 192.168.1.0 0.0.0.255

ip nat inside source list 1 interface WAN overload
```

This allows all home devices to share the single public IP assigned by the ISP.

## Troubleshooting Tips
- **No translations showing**: Check interface IP assignments
- **Specific traffic not translating**: Verify ACL matches source
- **Performance issues**: Monitor NAT translations with `show ip nat statistics`
- **IP change issues**: Ensure interface IP is correctly specified
