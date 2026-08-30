# RHCSA & RHCE — Real-World Networking Practice

A practical collection of **real-world networking scenarios** for Red Hat Enterprise Linux (RHEL), focused on **RHCSA (EX200)** and **RHCE (EX294)** preparation.

> **Practice rule:** Don't just make the service work. Make the configuration persistent, secure, and verifiable after reboot.

---

# RHCSA Networking

## 1. Static IP Configuration

A production RHEL server currently uses DHCP.

Configure:

```text
IP Address : 192.168.10.50/24
Gateway    : 192.168.10.1
DNS        : 192.168.10.10
```

### Requirements
- Configure the IP address.
- Configure the default gateway.
- Configure DNS.
- Ensure the configuration is persistent.
- Verify after restarting NetworkManager.

---

## 2. Network Interface Is Down

Interfaces:

```text
ens160
ens192
ens224
```

`ens160` is currently down.

### Tasks
- Identify the state of all interfaces.
- Bring `ens160` up.
- Identify its NetworkManager connection.
- Configure automatic activation at boot.
- Verify the IP configuration.

---

## 3. Server Cannot Reach the Internet

```text
IP      : 192.168.10.50/24
Gateway : 192.168.10.1
DNS     : 192.168.10.10
```

This works:

```bash
ping 192.168.10.1
```

But this fails:

```bash
ping 8.8.8.8
```

### Tasks
Determine whether the problem is related to routing, default gateway, network interface, firewall, or upstream connectivity. Fix and verify.

---

## 4. DNS Resolution Failure

This works:

```bash
ping 8.8.8.8
```

But this fails:

```bash
ping google.com
```

### Tasks
- Check DNS configuration.
- Identify the configured DNS server.
- Test DNS server reachability.
- Test DNS resolution.
- Correct the configuration.
- Verify hostname resolution.

---

## 5. Incorrect Default Gateway

The server can communicate with `192.168.10.0/24` but cannot reach `192.168.20.0/24`.

### Tasks
- Display the routing table.
- Identify the default route.
- Determine whether the gateway is correct.
- Correct the routing configuration.
- Verify connectivity.

---

## 6. Two Network Interfaces

```text
ens160 → 192.168.10.50/24
ens192 → 10.10.10.50/24
```

`ens160` is for management traffic and `ens192` for application traffic.

### Tasks
- Configure both interfaces.
- Configure appropriate routes.
- Configure the correct default route.
- Verify traffic routing.
- Ensure configuration persists after reboot.

---

## 7. Network Configuration Disappears After Reboot

An administrator manually configures an IP address. It works until reboot, after which the old DHCP address returns.

### Tasks
- Determine why the configuration is not persistent.
- Correct the NetworkManager configuration.
- Reboot.
- Verify.

---

## 8. Identify the Correct Interface

A server has `ens160`, `ens192`, and `ens224`. Only one is connected to production.

Determine the active interface, IP address, default gateway, MAC address, NetworkManager connection, and production interface.

---

# Services & Firewall

## 9. Apache Is Running but Inaccessible

Apache reports `active (running)`, but another server cannot access:

```text
http://192.168.10.50
```

Investigate network connectivity, listening ports, Apache configuration, firewall, SELinux, and service status.

---

## 10. Apache on Port 8080

Configure Apache to listen on TCP `8080`.

### Requirements
- Apache listens on 8080.
- SELinux permits the port.
- firewalld permits TCP 8080.
- Apache starts automatically after reboot.
- Verify from another server.

---

## 11. Open an Application Port

An application runs on TCP `8443`.

```bash
curl http://localhost:8443
```

works locally, but remote clients cannot connect.

### Tasks
- Verify the listening socket.
- Check firewall configuration.
- Allow TCP 8443 permanently.
- Verify remote connectivity.

---

## 12. Firewall Zone Problem

```text
ens160 → public
ens192 → internal
```

Port 8080 is allowed in `internal`, but users connecting through `ens160` cannot reach the application.

Diagnose and correct the zone configuration.

---

## 13. Find Listening Services

Display:
- Listening TCP ports
- Listening UDP ports
- Listening addresses
- Associated processes

Identify unexpected services.

---

# SSH Troubleshooting

## 14. SSH Connection Timeout

```bash
ssh user@192.168.10.50
```

returns:

```text
Connection timed out
```

Troubleshoot:

```text
Client
  ↓
Interface
  ↓
IP
  ↓
Routing
  ↓
Connectivity
  ↓
Firewall
  ↓
SSH Port
  ↓
sshd
```

---

## 15. SSH on a Non-Standard Port

Configure SSH to run on TCP `2222`.

### Requirements
- Configure sshd.
- Configure SELinux.
- Configure firewalld.
- Restart/reload correctly.
- Verify port 2222.
- Test from another server.

---

## 16. SSH Works Locally but Not Remotely

`ssh localhost` works, but another server cannot connect.

Investigate sshd, listening ports, firewall, SELinux, networking, and SSH configuration. Do not disable SELinux.

---

# Network Troubleshooting

## 17. Ping Gateway Fails

```text
IP      : 192.168.10.50/24
Gateway : 192.168.10.1
```

`ping 192.168.10.1` fails.

Investigate interface state, addressing, subnet mask, ARP, VLAN/network connection, and physical connectivity.

---

## 18. Ping Works but Application Doesn't

```bash
ping 192.168.10.50
```

works, but:

```bash
curl http://192.168.10.50
```

fails.

Investigate service availability, listening port, firewall, SELinux, and application configuration.

---

## 19. Application Works Locally but Not Remotely

```bash
curl http://localhost:9000
```

works locally.

```bash
curl http://192.168.10.50:9000
```

fails remotely.

Investigate listening address, port, firewall, SELinux, and network connectivity.

---

## 20. Service Listening Only on Localhost

The service listens on:

```text
127.0.0.1:9000
```

instead of:

```text
0.0.0.0:9000
```

Explain the difference, correct the application configuration, and verify remote connectivity.

---

# Advanced Networking

## 21. Persistent Static Route

Configure a persistent route:

```text
10.20.30.0/24 via 192.168.10.254
```

Verify the route and connectivity.

---

## 22. Multiple Routes

Networks:

```text
Network A → 192.168.10.0/24
Network B → 10.10.10.0/24
Network C → 172.16.10.0/24
```

Traffic to Network C must use a specific gateway.

Configure and verify the required route.

---

## 23. Hostname Configuration

Change:

```text
localhost.localdomain
```

to:

```text
web01.example.com
```

Ensure the hostname persists and local resolution works.

---

## 24. Network Configuration Verification

Collect:
- Hostname
- IP address
- Subnet
- Default gateway
- DNS server
- Active interface
- MAC address
- Listening ports
- Firewall zone
- SELinux status

---

# RHCE — Ansible Networking

## 25. Configure DNS with Ansible

Inventory:

```ini
[servers]
web01
web02
web03
db01
db02
```

Configure all servers to use `192.168.10.10` as DNS.

The configuration must be automated, persistent, and verified.

---

## 26. Configure Firewall with Ansible

All `webservers` must allow:

```text
HTTP  → TCP 80
HTTPS → TCP 443
```

Create a playbook that configures firewalld persistently and verifies the result.

---

## 27. Deploy an Application Port

Application servers use TCP `9000`.

Using Ansible:
1. Configure the application.
2. Start it.
3. Enable it at boot.
4. Configure SELinux.
5. Open TCP 9000.
6. Verify accessibility.

---

## 28. Configure Apache with Ansible

Configure all `[webservers]` to:
- Install httpd.
- Start and enable httpd.
- Deploy index.html.
- Configure the firewall.
- Verify the website.

---

## 29. Apache Configuration Using Jinja2

Create `httpd.conf.j2` using variables for:
- ServerName
- Listen
- DocumentRoot

Deploy it and restart Apache only when the configuration changes.

---

## 30. Network Troubleshooting with Ansible

You manage 50 RHEL servers. Several cannot reach the application network.

Use Ansible to collect:
- IP addresses
- Routing tables
- DNS configuration
- Default gateways
- Listening ports
- Firewall status
- NetworkManager status

Identify incorrectly configured systems.

---

# 31. Complete Production Server Challenge

Configure:

```text
Hostname : web01.example.com
IP       : 192.168.10.50/24
Gateway  : 192.168.10.1
DNS      : 192.168.10.10
HTTP     : 80
HTTPS    : 443
```

### Requirements
- Static IP
- Persistent hostname
- Correct DNS
- Correct default route
- Apache installed
- Apache enabled at boot
- Test website
- HTTP/HTTPS firewall access
- SELinux enforcing
- Configuration survives reboot

### Verify After Reboot

```text
✓ Hostname
✓ IP address
✓ Default route
✓ DNS resolution
✓ Apache status
✓ Listening ports
✓ Firewall
✓ SELinux
✓ Website accessibility
```

---

# Troubleshooting Methodology

```text
Interface
   ↓
IP Address
   ↓
Subnet
   ↓
Routing
   ↓
Gateway
   ↓
DNS
   ↓
Listening Port
   ↓
Firewall
   ↓
SELinux
   ↓
Service / Application
```

---

# Essential Commands

```bash
ip addr
ip link
ip route
ip route get <IP>
ip neigh

nmcli device status
nmcli connection show
nmcli connection show <connection>

ping <IP>
tracepath <IP>

ss -tulnp

resolvectl status
cat /etc/resolv.conf

firewall-cmd --list-all
firewall-cmd --get-active-zones
firewall-cmd --list-services
firewall-cmd --list-ports

getenforce
sestatus
semanage port -l

systemctl status NetworkManager
systemctl status <service>

journalctl -u NetworkManager
journalctl -u <service>
```

---

# Practice Philosophy

For every networking task, ask:

- Does it work now?
- Will it survive reboot?
- Is it secure?
- Is SELinux still enforcing?
- Is the firewall correctly configured?
- Can I verify the result?
- Can I automate it with Ansible?

> **Configure → Verify → Reboot → Verify Again**
