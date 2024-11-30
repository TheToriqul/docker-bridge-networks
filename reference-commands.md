# Docker Bridge Networks Reference Commands

### Project Content Table
- [Section 1: Core Project Workflow](#section-1-core-project-workflow)
- [Section 2: Advanced Operations](#section-2-advanced-operations)
- [Section 3: Production Guide](#section-3-production-guide)

> **Author**: [Md Toriqul Islam](https://linkedin.com/TheToriqul)  
> **Description**: Exploring Bridge Networks in Docker
> **Learning Focus**: Docker custom bridge networks, attaching containers to multiple networks, using `ip` and `nmap` for network examination and discovery
> **Note**: This is a reference guide. Do not execute commands directly without understanding their implications.

## Section 1: Core Project Workflow

### Step 1: Create a Custom Bridge Network
```bash
docker network create \
    --driver bridge \
    --label project=dockerinaction \
    --label chapter=5 \
    --attachable \
    --scope local \
    --subnet 10.0.42.0/24 \
    --ip-range 10.0.42.128/25 \
    user-network
```

### Step 2: Launch a Container in the Custom Network
```bash
docker run -it \
  --network user-network \
  --name network-explorer \
  alpine:3.8 \
    sh
```

### Step 3: Examine Network Interfaces Inside the Container
```bash
# Inside the container
ip -f inet -4 -o addr
```

### Step 4: Create Another Custom Bridge Network
```bash
docker network create \
  --driver bridge \
  --label project=dockerinaction \
  --label chapter=5 \
  --attachable \
  --scope local \
  --subnet 10.0.43.0/24 \
  --ip-range 10.0.43.128/25 \
  user-network2
```

### Step 5: Attach the Container to the New Network
```bash
docker network connect \
  user-network2 \
  network-explorer
```

### Step 6: Install nmap Inside the Container
```bash
docker exec -it network-explorer sh -c "apk update && apk add nmap"
```

### Step 7: Scan Subnets Using nmap
```bash
docker exec -it network-explorer sh -c "nmap -sn 10.0.42.* 10.0.43.* -oG - | grep Status"
```

### Verification Commands
```bash
# List all networks
docker network ls

# Inspect network details
docker network inspect user-network

# List containers attached to a network
docker network inspect --format='{{range .Containers}}{{.Name}}{{end}}' user-network
```

### Final Step: Cleanup
```bash
docker rm -f network-explorer
docker network rm user-network user-network2
```

## Section 2: Advanced Operations

### Assign Static IP to a Container
```bash
docker run -dit \
  --network user-network \
  --ip 10.0.42.100 \
  --name static-ip-container \
  alpine:latest
```

### Create a Network with a Gateway
```bash
docker network create \
  --driver bridge \
  --subnet 192.168.0.0/24 \
  --gateway 192.168.0.1 \
  gateway-network
```

### Connect a Container to a Network with Aliases
```bash
docker run -dit \
  --network user-network \
  --network-alias my-container \
  --name aliased-container \
  alpine:latest
```

### Disconnect a Container from a Network
```bash
docker network disconnect user-network2 network-explorer
```

### Remove Unused Networks
```bash
docker network prune
```

## Section 3: Production Guide

### Create an Overlay Network for Multi-Host Communication
```bash
docker network create \
  --driver overlay \
  --subnet 10.0.9.0/24 \
  prod-overlay-network
```

### Deploy a Stack with a Dedicated Network
```bash
docker stack deploy -c docker-compose.yml my-stack
```

### Implement Network Security Using Firewall Rules
```bash
docker network create \
  --driver bridge \
  --subnet 172.20.0.0/16 \
  --opt com.docker.network.bridge.enable_ip_masquerade=true \
  --opt com.docker.network.bridge.enable_icc=false \
  --opt com.docker.network.bridge.host_binding_ipv4=0.0.0.0 \
  secure-network
```

### Monitor Network Traffic Using sysdig
```bash
sudo docker run -it --rm \
  --name sysdig \
  --privileged \
  --volume /var/run/docker.sock:/host/var/run/docker.sock \
  --volume /dev:/host/dev \
  --volume /proc:/host/proc:ro \
  --volume /boot:/host/boot:ro \
  --volume /lib/modules:/host/lib/modules:ro \
  --volume /usr:/host/usr:ro \
  sysdig/sysdig
```

## Learning Notes

1. Custom bridge networks provide isolation and segregation for containers
2. Containers can be attached to multiple networks simultaneously
3. The `ip` command allows inspection of network interfaces and IP addresses inside containers
4. `nmap` is a powerful tool for network scanning and discovery
5. Docker network commands enable creation, inspection, and management of networks
6. Advanced network configurations like static IPs, gateways, and aliases can be achieved
7. Overlay networks enable multi-host container communication in production environments
8. Network security can be enhanced using firewall rules and traffic monitoring tools

---

> 💡 **Best Practice**: Use custom bridge networks to isolate and secure container communication.

> ⚠️ **Warning**: Executing `nmap` scans on networks without proper authorization may be illegal or against organizational policies.

> 📝 **Note**: Container network configuration should align with the application's requirements and security best practices.