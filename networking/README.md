# RHCSA & RHCE — Real-World Networking Practice

A practical collection of **real-world networking scenarios** for Red Hat Enterprise Linux (RHEL), focused on **RHCSA (EX200)** and **RHCE (EX294)** preparation.

> **Practice rule:** Don't just make the service work. Make the configuration persistent, secure, and verifiable after reboot.

---

## 📚 Contents

- [1. Static IP Configuration](#1-static-ip-configuration)
- [2. Network Interface Is Down](#2-network-interface-is-down)
- [3. Server Cannot Reach the Internet](#3-server-cannot-reach-the-internet)
- [4. DNS Resolution Failure](#4-dns-resolution-failure)
- [5. Incorrect Default Gateway](#5-incorrect-default-gateway)
- [6. Two Network Interfaces](#6-two-network-interfaces)
- [7. Network Configuration Disappears After Reboot](#7-network-configuration-disappears-after-reboot)
- [8. Identify the Correct Interface](#8-identify-the-correct-interface)
- [9. Apache Is Running but Inaccessible](#9-apache-is-running-but-inaccessible)
- [10. Apache on Port 8080](#10-apache-on-port-8080)
- [11. Open an Application Port](#11-open-an-application-port)
- [12. Firewall Zone Problem](#12-firewall-zone-problem)
- [13. Find Listening Services](#13-find-listening-services)
- [14. SSH Connection Timeout](#14-ssh-connection-timeout)
- [15. SSH on a Non-Standard Port](#15-ssh-on-a-non-standard-port)
- [16. SSH Works Locally but Not Remotely](#16-ssh-works-locally-but-not-remotely)
- [17. Ping Gateway Fails](#17-ping-gateway-fails)
- [18. Ping Works but Application Doesn't](#18-ping-works-but-application-doesnt)
- [19. Application Works Locally but Not Remotely](#19-application-works-locally-but-not-remotely)
- [20. Service Listening Only on Localhost](#20-service-listening-only-on-localhost)
- [21. Persistent Static Route](#21-persistent-static-route)
- [22. Multiple Routes](#22-multiple-routes)
- [23. Hostname Configuration](#23-hostname-configuration)
- [24. Network Configuration Verification](#24-network-configuration-verification)
- [25. Configure DNS with Ansible](#25-configure-dns-with-ansible)
- [26. Configure Firewall with Ansible](#26-configure-firewall-with-ansible)
- [27. Deploy an Application Port](#27-deploy-an-application-port)
- [28. Configure Apache with Ansible](#28-configure-apache-with-ansible)
- [29. Apache Configuration Using Jinja2](#29-apache-configuration-using-jinja2)
- [30. Network Troubleshooting with Ansible](#30-network-troubleshooting-with-ansible)
- [31. Complete Production Server Configuration](#31-complete-production-server-configuration)
- [Troubleshooting Methodology](#troubleshooting-methodology)
- [Essential Commands](#essential-commands)

---

# RHCSA Networking

## 1. Static IP Configuration

A production RHEL server currently uses DHCP.

Configure the server with:

```text
IP Address : 192.168.10.50/24
Gateway    : 192.168.10.1
DNS        : 192.168.10.10
