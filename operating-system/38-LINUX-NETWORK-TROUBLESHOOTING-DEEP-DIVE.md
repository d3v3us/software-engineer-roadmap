# Linux Network Troubleshooting Deep Dive - Complete Understanding

## Table of Contents
1. [What is Network Troubleshooting?](#what-is-network-troubleshooting)
2. [Why Network Troubleshooting Matters](#why-network-troubleshooting-matters)
3. [Basic Connectivity Checks](#basic-connectivity-checks)
4. [Port and Service Checks](#port-and-service-checks)
5. [Routing Troubleshooting](#routing-troubleshooting)
6. [DNS Troubleshooting](#dns-troubleshooting)
7. [Firewall Troubleshooting](#firewall-troubleshooting)
8. [Performance Troubleshooting](#performance-troubleshooting)
9. [Best Practices](#best-practices)

---

## What is Network Troubleshooting?

### Definition

**Network Troubleshooting**: Process of identifying and resolving network issues.

**Key Characteristics:**
- **Problem identification**: Identify network problems
- **Root cause analysis**: Find root causes
- **Issue resolution**: Resolve issues
- **Network understanding**: Understand network behavior

### Real-World Analogy

**Network Troubleshooting = Traffic Investigation:**
- **Traffic jam**: Network issue
- **Investigation**: Troubleshooting process
- **Root cause**: Find cause
- **Solution**: Fix issue

**System Administration:**
- **Network issue**: Connectivity problem
- **Troubleshooting**: Investigation
- **Tools**: Network tools
- **Resolution**: Problem fix

---

## Why Network Troubleshooting Matters?

### Benefits

**1. Connectivity:**
```
Network troubleshooting
  ↓
Connectivity restoration
  ↓
Service availability
```

**2. Performance:**
```
Network troubleshooting
  ↓
Performance optimization
  ↓
Better user experience
```

**3. Reliability:**
```
Network troubleshooting
  ↓
Reliable network
  ↓
System reliability
```

---

## Basic Connectivity Checks

### Ping

**Ping Test:**
```bash
# Basic ping
ping google.com

# Ping with count
ping -c 4 google.com

# Ping with interval
ping -i 2 google.com

# Ping specific interface
ping -I eth0 google.com
```

**Interpreting Results:**
- **Success**: Connectivity OK
- **Timeout**: No connectivity
- **Packet loss**: Network issues

### Traceroute

**Traceroute:**
```bash
# Basic traceroute
traceroute google.com

# Traceroute with max hops
traceroute -m 30 google.com

# Traceroute with timeout
traceroute -w 5 google.com
```

**Interpreting Results:**
- **Hops**: Network path
- **Timeouts**: Network issues
- **Latency**: Network latency

---

## Port and Service Checks

### Checking Open Ports

**netstat:**
```bash
# Listening ports
netstat -tlnp

# All connections
netstat -an

# Specific port
netstat -an | grep 80
```

**ss:**
```bash
# Listening ports
ss -tlnp

# All connections
ss -an

# Specific port
ss -tlnp | grep :80
```

**lsof:**
```bash
# Ports by process
lsof -i :80

# All network connections
lsof -i
```

### Testing Port Connectivity

**telnet:**
```bash
# Test TCP port
telnet hostname 80

# Test with timeout
timeout 5 telnet hostname 80
```

**nc (netcat):**
```bash
# Test TCP port
nc -zv hostname 80

# Test UDP port
nc -zuv hostname 53

# Test with timeout
timeout 5 nc -zv hostname 80
```

**curl:**
```bash
# Test HTTP port
curl -v http://hostname:80

# Test with timeout
curl --connect-timeout 5 http://hostname:80
```

### Finding Process Using Port

**Commands:**
```bash
# Using lsof
lsof -i :80

# Using netstat
netstat -tlnp | grep :80

# Using ss
ss -tlnp | grep :80

# Using fuser
fuser 80/tcp
```

---

## Routing Troubleshooting

### Checking Routing Table

**ip route:**
```bash
# Show routing table
ip route show

# Show default route
ip route show default

# Add route
ip route add 192.168.1.0/24 via 192.168.0.1

# Delete route
ip route del 192.168.1.0/24
```

**route:**
```bash
# Show routing table
route -n

# Add route
route add -net 192.168.1.0/24 gw 192.168.0.1

# Delete route
route del -net 192.168.1.0/24
```

### Testing Routing

**Commands:**
```bash
# Traceroute
traceroute destination

# MTR (My Traceroute)
mtr destination

# Check specific route
ip route get destination
```

---

## DNS Troubleshooting

### DNS Resolution

**nslookup:**
```bash
# Basic lookup
nslookup google.com

# Specific DNS server
nslookup google.com 8.8.8.8

# Reverse lookup
nslookup 8.8.8.8
```

**dig:**
```bash
# Basic lookup
dig google.com

# Specific DNS server
dig @8.8.8.8 google.com

# Reverse lookup
dig -x 8.8.8.8

# All record types
dig google.com ANY
```

**host:**
```bash
# Basic lookup
host google.com

# Reverse lookup
host 8.8.8.8
```

### DNS Configuration

**Checking DNS:**
```bash
# /etc/resolv.conf
cat /etc/resolv.conf

# systemd-resolved
systemd-resolve --status

# Test DNS
dig @127.0.0.1 google.com
```

---

## Firewall Troubleshooting

### Checking iptables

**Commands:**
```bash
# List rules
iptables -L -n -v

# List with line numbers
iptables -L -n -v --line-numbers

# Check specific rule
iptables -C INPUT -p tcp --dport 80 -j ACCEPT

# Test packet
iptables -t filter -v -L
```

### Checking firewalld

**Commands:**
```bash
# Status
firewall-cmd --state

# List zones
firewall-cmd --list-all-zones

# List services
firewall-cmd --list-services

# Check port
firewall-cmd --query-port=80/tcp
```

---

## Performance Troubleshooting

### Network Statistics

**ifconfig:**
```bash
# Interface statistics
ifconfig eth0

# Check errors
ifconfig eth0 | grep errors
```

**ip:**
```bash
# Interface statistics
ip -s link show eth0

# Detailed statistics
ip -s -s link show eth0
```

**ethtool:**
```bash
# Interface information
ethtool eth0

# Statistics
ethtool -S eth0

# Link status
ethtool eth0 | grep Link
```

### Network Monitoring

**tcpdump:**
```bash
# Capture packets
tcpdump -i eth0

# Capture specific port
tcpdump -i eth0 port 80

# Capture to file
tcpdump -i eth0 -w capture.pcap
```

**iftop:**
```bash
# Network usage by connection
iftop -i eth0
```

**nethogs:**
```bash
# Network usage by process
nethogs
```

---

## Best Practices

### 1. Systematic Approach

**Why:**
- **Efficiency**: More efficient troubleshooting
- **Thoroughness**: Thorough investigation
- **Time**: Save time

**Guidelines:**
- **Start simple**: Start with basic checks
- **Layer by layer**: Check each layer
- **Document**: Document findings

### 2. Use Right Tools

**Why:**
- **Accuracy**: More accurate results
- **Efficiency**: More efficient
- **Information**: Better information

**Guidelines:**
- **ping**: Basic connectivity
- **traceroute**: Routing
- **netstat/ss**: Ports
- **tcpdump**: Packet analysis

### 3. Check Logs

**Why:**
- **History**: Historical information
- **Errors**: Error messages
- **Context**: System context

**Guidelines:**
- **System logs**: Check system logs
- **Application logs**: Check application logs
- **Network logs**: Check network logs

### 4. Document Solutions

**Why:**
- **Knowledge**: Knowledge sharing
- **Reference**: Future reference
- **Learning**: Learning opportunity

**Guidelines:**
- **Document**: Document solutions
- **Share**: Share knowledge
- **Learn**: Learn from issues

---

## Summary

Linux network troubleshooting helps identify and resolve network issues. Understanding basic connectivity checks, port and service checks, routing troubleshooting, DNS troubleshooting, firewall troubleshooting, performance troubleshooting, and best practices is crucial for system administration.

**Key Takeaways:**
- **Network troubleshooting**: Process of identifying and resolving network issues (problem identification, root cause analysis, issue resolution, network understanding)
- **Basic connectivity checks**: Ping (connectivity test), traceroute (path analysis)
- **Port and service checks**: Checking open ports (netstat, ss, lsof), testing port connectivity (telnet, nc, curl), finding process using port
- **Routing troubleshooting**: Checking routing table (ip route, route), testing routing (traceroute, mtr)
- **DNS troubleshooting**: DNS resolution (nslookup, dig, host), DNS configuration (/etc/resolv.conf, systemd-resolved)
- **Firewall troubleshooting**: Checking iptables, checking firewalld
- **Performance troubleshooting**: Network statistics (ifconfig, ip, ethtool), network monitoring (tcpdump, iftop, nethogs)
- **Best practices**: Systematic approach, use right tools, check logs, document solutions

**Troubleshooting Tools:**
- **ping**: Connectivity
- **traceroute**: Routing
- **netstat/ss**: Ports
- **tcpdump**: Packet analysis

**Best Practices:**
- Systematic approach
- Use right tools
- Check logs
- Document solutions

**Next Steps:**
- Learn tools
- Practice troubleshooting
- Build experience
- Share knowledge

