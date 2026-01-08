# Linux Debugging Techniques Deep Dive - Complete Understanding

## Table of Contents
1. [What is Linux Debugging?](#what-is-linux-debugging)
2. [Why Debugging Matters](#why-debugging-matters)
3. [Debugging Hanging Processes](#debugging-hanging-processes)
4. [Debugging iptables Rules](#debugging-iptables-rules)
5. [Process Debugging](#process-debugging)
6. [Network Debugging](#network-debugging)
7. [System Debugging](#system-debugging)
8. [Best Practices](#best-practices)

---

## What is Linux Debugging?

### Definition

**Linux Debugging**: Process of identifying and resolving issues in Linux systems.

**Key Characteristics:**
- **Problem identification**: Identify problems
- **Root cause analysis**: Find root causes
- **Issue resolution**: Resolve issues
- **System understanding**: Understand system behavior

### Real-World Analogy

**Debugging = Detective Work:**
- **Crime**: System issue
- **Evidence**: Logs, metrics
- **Investigation**: Debugging process
- **Solution**: Fix

**System Administration:**
- **Issue**: System problem
- **Debugging**: Investigation process
- **Tools**: Debugging tools
- **Resolution**: Problem fix

---

## Why Debugging Matters?

### Benefits

**1. Issue Resolution:**
```
Debugging
  ↓
Problem identification
  ↓
Issue resolution
```

**2. System Understanding:**
```
Debugging
  ↓
System analysis
  ↓
Better understanding
```

**3. Prevention:**
```
Debugging
  ↓
Root cause analysis
  ↓
Prevent recurrence
```

---

## Debugging Hanging Processes

### What is a Hanging Process?

**Hanging Process**: Process that appears to be running but is not responding.

**Characteristics:**
- **Not responding**: No response to signals
- **Consuming resources**: May consume CPU/memory
- **Blocked**: May be blocked on I/O or locks
- **Deadlock**: May be in deadlock

### Identifying Hanging Processes

**Signs:**
- Process doesn't respond to signals
- High CPU usage (spinning)
- High I/O wait
- No output
- Stuck in system call

**Commands:**
```bash
# Check process status
ps aux | grep <process_name>

# Check process state
ps -o pid,state,cmd <pid>
# States: R (running), S (sleeping), D (uninterruptible sleep), Z (zombie), T (stopped)

# Check process threads
ps -T -p <pid>

# Check process file descriptors
lsof -p <pid>

# Check process system calls
strace -p <pid>
```

### Debugging Techniques

**1. Check Process State:**
```bash
# Process state
ps -o pid,state,cmd <pid>

# D state: Uninterruptible sleep (usually I/O wait)
# S state: Interruptible sleep (waiting for event)
# R state: Running
# Z state: Zombie
```

**2. Check System Calls:**
```bash
# Trace system calls
strace -p <pid>

# Common issues:
# - Blocked on read/write
# - Waiting for lock
# - Waiting for signal
```

**3. Check Stack Trace:**
```bash
# Get stack trace
gdb -p <pid>
(gdb) thread apply all bt

# Or using gcore
gcore <pid>
gdb <executable> core.<pid>
(gdb) bt
```

**4. Check File Descriptors:**
```bash
# Check open files
lsof -p <pid>

# Check for file locks
lsof -p <pid> | grep -i lock
```

**5. Check Network Connections:**
```bash
# Check network connections
netstat -anp | grep <pid>
ss -anp | grep <pid>

# Check for hanging connections
netstat -an | grep TIME_WAIT
netstat -an | grep CLOSE_WAIT
```

### Common Causes

**1. I/O Wait:**
- **Symptom**: Process in D state
- **Cause**: Waiting for disk I/O
- **Fix**: Check disk I/O, optimize I/O

**2. Deadlock:**
- **Symptom**: Process stuck, no CPU usage
- **Cause**: Waiting for lock
- **Fix**: Identify deadlock, fix locking

**3. Network Wait:**
- **Symptom**: Process waiting for network
- **Cause**: Network timeout
- **Fix**: Check network, set timeouts

**4. Signal Wait:**
- **Symptom**: Process waiting for signal
- **Cause**: Blocked on signal
- **Fix**: Check signal handling

### Solutions

**1. Kill Process:**
```bash
# Try SIGTERM first
kill <pid>

# If doesn't work, use SIGKILL
kill -9 <pid>
```

**2. Restart Service:**
```bash
# Systemd service
systemctl restart <service>

# Init script
service <service> restart
```

**3. Fix Root Cause:**
- Fix I/O issues
- Fix deadlocks
- Fix network issues
- Fix signal handling

---

## Debugging iptables Rules

### What is iptables?

**iptables**: Linux firewall and packet filtering tool.

**Components:**
- **Tables**: filter, nat, mangle, raw
- **Chains**: INPUT, OUTPUT, FORWARD
- **Rules**: Packet matching rules
- **Targets**: ACCEPT, DROP, REJECT

### Checking iptables Rules

**Commands:**
```bash
# List all rules
iptables -L -n -v

# List rules with line numbers
iptables -L -n -v --line-numbers

# List specific chain
iptables -L INPUT -n -v

# List rules in specific table
iptables -t nat -L -n -v

# Save rules
iptables-save > /etc/iptables/rules.v4
```

### Debugging Techniques

**1. Check Rule Order:**
```bash
# Rules are processed in order
iptables -L -n -v --line-numbers

# First matching rule applies
# Check if rules are in correct order
```

**2. Test Rules:**
```bash
# Test packet matching
iptables -C INPUT -s 192.168.1.0/24 -j ACCEPT

# Check if rule matches
iptables -L -n -v | grep <rule>
```

**3. Use Logging:**
```bash
# Add logging rule
iptables -I INPUT -j LOG --log-prefix "IPTABLES: " --log-level 4

# Check logs
tail -f /var/log/kern.log
dmesg | grep IPTABLES
```

**4. Trace Packets:**
```bash
# Use tcpdump
tcpdump -i <interface> -n

# Use wireshark
wireshark -i <interface>
```

### Common Issues

**1. Rules Not Applied:**
- **Cause**: Rules not saved
- **Fix**: Save rules with iptables-save

**2. Wrong Rule Order:**
- **Cause**: Rules in wrong order
- **Fix**: Reorder rules

**3. Default Policy:**
- **Cause**: Default policy blocks
- **Fix**: Set default policy or add allow rules

**4. Interface Issues:**
- **Cause**: Wrong interface
- **Fix**: Check interface name

### Debugging Workflow

**1. List Rules:**
```bash
iptables -L -n -v --line-numbers
```

**2. Check Logs:**
```bash
tail -f /var/log/kern.log
dmesg | tail
```

**3. Test Connectivity:**
```bash
# Test from source
ping <destination>

# Test with telnet
telnet <host> <port>
```

**4. Add Logging:**
```bash
# Add logging rule at beginning
iptables -I INPUT 1 -j LOG --log-prefix "INPUT: "
```

**5. Analyze and Fix:**
- Identify blocking rule
- Fix rule or order
- Test again

---

## Process Debugging

### Process Information

**Commands:**
```bash
# Process details
ps aux | grep <process>

# Process tree
pstree -p <pid>

# Process limits
cat /proc/<pid>/limits

# Process memory
cat /proc/<pid>/status

# Process file descriptors
ls -l /proc/<pid>/fd
```

### Process Tracing

**strace:**
```bash
# Trace system calls
strace -p <pid>

# Trace with timestamps
strace -t -p <pid>

# Trace specific system calls
strace -e trace=open,read,write -p <pid>
```

**ltrace:**
```bash
# Trace library calls
ltrace -p <pid>
```

**gdb:**
```bash
# Attach to process
gdb -p <pid>

# Get stack trace
(gdb) bt

# Check variables
(gdb) print <variable>
```

---

## Network Debugging

### Network Tools

**tcpdump:**
```bash
# Capture packets
tcpdump -i <interface> -n

# Capture specific port
tcpdump -i <interface> port 80

# Capture to file
tcpdump -i <interface> -w capture.pcap
```

**netstat:**
```bash
# Network connections
netstat -an

# Listening ports
netstat -tlnp

# Process connections
netstat -anp | grep <pid>
```

**ss:**
```bash
# Socket statistics
ss -tan

# Listening sockets
ss -tlnp

# Process sockets
ss -anp | grep <pid>
```

**traceroute:**
```bash
# Trace route
traceroute <host>
```

---

## System Debugging

### System Logs

**Commands:**
```bash
# System logs
journalctl

# Recent logs
journalctl -n 100

# Follow logs
journalctl -f

# Logs for service
journalctl -u <service>

# Logs since boot
journalctl -b
```

### System Information

**Commands:**
```bash
# System info
uname -a

# CPU info
cat /proc/cpuinfo

# Memory info
free -h
cat /proc/meminfo

# Disk info
df -h
iostat -x
```

---

## Best Practices

### 1. Use Right Tools

**Why:**
- **Efficiency**: More efficient debugging
- **Accuracy**: More accurate results
- **Time**: Save time

**Guidelines:**
- **strace**: System call tracing
- **gdb**: Process debugging
- **tcpdump**: Network debugging
- **iptables**: Firewall debugging

### 2. Check Logs First

**Why:**
- **Quick**: Quick problem identification
- **History**: Historical information
- **Context**: System context

**Guidelines:**
- **System logs**: Check system logs
- **Application logs**: Check application logs
- **Kernel logs**: Check kernel logs
- **Service logs**: Check service logs

### 3. Reproduce Issue

**Why:**
- **Understanding**: Better understanding
- **Testing**: Test fixes
- **Verification**: Verify resolution

**Guidelines:**
- **Reproduce**: Reproduce issue
- **Document**: Document steps
- **Test**: Test solutions

### 4. Document Findings

**Why:**
- **Knowledge**: Knowledge sharing
- **Reference**: Future reference
- **Learning**: Learning opportunity

**Guidelines:**
- **Document**: Document findings
- **Share**: Share knowledge
- **Learn**: Learn from issues

---

## Summary

Linux debugging techniques help identify and resolve system issues. Understanding debugging hanging processes, iptables rules, process debugging, network debugging, system debugging, and best practices is crucial for system administration.

**Key Takeaways:**
- **Linux debugging**: Process of identifying and resolving issues (problem identification, root cause analysis, issue resolution, system understanding)
- **Debugging hanging processes**: Identifying (signs, commands), techniques (check state, system calls, stack trace, file descriptors, network), common causes (I/O wait, deadlock, network wait, signal wait), solutions (kill, restart, fix root cause)
- **Debugging iptables rules**: Checking rules (iptables -L), techniques (check order, test rules, use logging, trace packets), common issues (rules not applied, wrong order, default policy, interface issues), workflow (list, check logs, test, add logging, analyze)
- **Process debugging**: Process information (ps, pstree, /proc), tracing (strace, ltrace, gdb)
- **Network debugging**: Tools (tcpdump, netstat, ss, traceroute)
- **System debugging**: System logs (journalctl), system information (uname, /proc)
- **Best practices**: Use right tools, check logs first, reproduce issue, document findings

**Debugging Tools:**
- **strace**: System call tracing
- **gdb**: Process debugging
- **tcpdump**: Network debugging
- **iptables**: Firewall debugging

**Best Practices:**
- Use right tools
- Check logs first
- Reproduce issue
- Document findings

**Next Steps:**
- Learn tools
- Practice debugging
- Build experience
- Share knowledge

