# Docker Bridge Networks - Advanced Command Reference Guide

### Quick Reference Table

| Operation | Command | Purpose |
|-----------|---------|----------|
| Create Network | `docker network create` | Create custom bridge network |
| Connect Container | `docker network connect` | Attach container to network |
| Inspect Network | `docker network inspect` | View detailed network config |
| List Networks | `docker network ls` | Show all networks |
| Remove Network | `docker network rm` | Delete unused network |

### Content Sections
- [Core Network Operations](#core-network-operations)
- [Container Network Management](#container-network-management)
- [Network Analysis & Diagnostics](#network-analysis--diagnostics)
- [Security & Maintenance](#security--maintenance)
- [Troubleshooting Guide](#troubleshooting-guide)

> **Author**: [Md Toriqul Islam](https://linkedin.com/in/thetoriqul)  
> **Focus**: Advanced Docker bridge network implementation and analysis  
> **Environment**: Docker Engine 20.10+, Alpine Linux 3.8  
> **Note**: This is a reference guide. Do not execute commands directly without understanding their implications.

## Core Network Operations

### Custom Bridge Network Creation
```bash
# Create primary user network with custom subnet
docker network create \
    --driver bridge \
    --label project=dockerinaction \
    --attachable \
    --scope local \
    --subnet 10.0.42.0/24 \
    --ip-range 10.0.42.128/25 \
    user-network

# Verification:
docker network inspect user-network | jq '.[] | {Name, Driver, Scope, IPAM}'
```

### Secondary Network Setup
```bash
# Create secondary network for multi-network testing
docker network create \
    --driver bridge \
    --attachable \
    --subnet 10.0.43.0/24 \
    --ip-range 10.0.43.128/25 \
    user-network2

# List networks with custom format
docker network ls --format "table {{.ID}}\t{{.Name}}\t{{.Driver}}\t{{.Scope}}"
```

## Container Network Management

### Network Explorer Container
```bash
# Launch network analysis container
docker run -it \
    --name network-explorer \
    --network user-network \
    --cap-add=NET_ADMIN \  # Required for network analysis
    alpine:3.8 \
    sh

# Verify IP configuration
ip -f inet -4 -o addr
```

### Multi-Network Configuration
```bash
# Attach to secondary network
docker network connect user-network2 network-explorer

# Verify network interfaces
docker exec network-explorer ip -4 addr show

# Display detailed network config
docker inspect \
    --format='{{range .NetworkSettings.Networks}}{{.IPAddress}}{{"\n"}}{{end}}' \
    network-explorer
```

## Network Analysis & Diagnostics

### Setup Analysis Environment
```bash
# Install network analysis tools
docker exec -it network-explorer sh -c \
    "apk add --no-cache \
    nmap \
    tcpdump \
    iptables \
    curl"

# Verify installations
docker exec network-explorer which nmap tcpdump iptables curl
```

### Network Discovery
```bash
# Comprehensive network scan
docker exec -it network-explorer \
    nmap -sn 10.0.42.0/24 10.0.43.0/24 \
    -oG /dev/stdout | grep Status

# TCP port scan on specific subnet
docker exec -it network-explorer \
    nmap -p- -T4 10.0.42.0/24 \
    --exclude 10.0.42.1

# Capture network traffic
docker exec -it network-explorer \
    tcpdump -i eth0 -n
```

## Security & Maintenance

### Network Isolation Verification
```bash
# Test network isolation
docker exec network-explorer \
    ping -c 2 172.17.0.1  # Default bridge gateway

# Check network policies
docker exec network-explorer \
    iptables -L -n -v

# Display routing table
docker exec network-explorer \
    ip route show
```

### Network Cleanup Operations
```bash
# Graceful container disconnect
docker network disconnect \
    --force user-network2 network-explorer

# Remove networks (ensure no connected containers)
docker network rm user-network user-network2

# Verify cleanup
docker network ls --filter name=user-network
```

## Troubleshooting Guide

### Connectivity Issues
```bash
# Check DNS resolution
docker exec network-explorer \
    nslookup google.com

# Test network latency
docker exec network-explorer \
    ping -c 4 8.8.8.8

# Trace network path
docker exec network-explorer \
    traceroute 8.8.8.8
```

### Network Performance Analysis
```bash
# Monitor network interfaces
docker exec network-explorer \
    ip -s link show

# Check network statistics
docker exec network-explorer \
    netstat -i

# Monitor real-time network usage
docker exec network-explorer \
    iftop -i eth0  # If iftop is installed
```

## Learning Notes

1. Bridge networks provide container isolation while enabling controlled communication
2. Multi-network attachments require careful IP range planning
3. Network analysis tools are essential for troubleshooting and optimization
4. Container capabilities affect network analysis capabilities
5. Regular network maintenance prevents resource conflicts

---

> 💡 **Best Practice**: Always verify network connectivity after configuration changes

> ⚠️ **Warning**: Network modifications can impact running containers

> 📝 **Note**: Keep track of custom network configurations for documentation