# pihole-dns-filtering-lab
Pi-hole DNS filtering home lab on Ubuntu and VMware

This project documents my deployment of **Pi-hole** on an **Ubuntu Server VM in VMware** to build a centralized **DNS filtering solution** for a home lab environment. The final design uses **Pi-hole as the client-facing DNS server** and **Cloudflare Family** as the upstream resolver for additional content and malicious-domain filtering.

This project was valuable not just because I installed Pi-hole, but because I had to troubleshoot and validate the full path between **clients, DHCP, DNS, virtualization, and upstream resolvers**. It became a strong hands-on networking and systems lab that helped me better understand how DNS filtering works in a real environment.

---

## Project Overview

The goal of this project was to build a working DNS filtering service for multiple devices on a network while strengthening my practical skills in:

- Linux server administration
- VMware networking
- DNS and DHCP configuration
- network troubleshooting
- layered filtering design
- validating client-to-resolver traffic flow

The final solution allows client devices to send DNS queries to a Pi-hole VM, which then applies local filtering rules and forwards allowed requests to Cloudflare Family for additional upstream filtering.

---

## Final Architecture

### DNS Flow

Client Device → Pi-hole VM → Cloudflare Family DNS

### Final Design Roles

- **Router**
  - provides internet access and remains the default gateway
- **Pi-hole VM**
  - provides DNS filtering
  - provides DHCP leases to clients
- **Cloudflare Family**
  - acts as the upstream DNS resolver for Pi-hole

---

## Technologies Used

- **Ubuntu Server**
- **VMware Workstation**
- **Pi-hole**
- **Cloudflare Family DNS**
- **DHCP**
- **DNS**
- **Windows client testing tools**
  - `ipconfig /all`
  - `nslookup`
  - `ping`

---

## Why I Built This

I wanted a project that combined:

- networking
- Linux
- virtualization
- troubleshooting
- practical cybersecurity concepts
- and fun

Pi-hole was a strong fit because it allowed me to work with DNS from both the server side and the client side. It also made it easier to observe how addressing, DHCP, and upstream DNS decisions affect actual end-user connectivity.

This project also helped me move beyond theory and into configuration validation, fault isolation, and step-by-step troubleshooting.

---

## Deployment Summary

### 1. Ubuntu VM deployment
I created an Ubuntu Server virtual machine in VMware and configured it for bridged network access so the VM could function as a reachable service on the local network.

### 2. Pi-hole installation
Pi-hole was installed successfully and the admin interface was brought online.

### 3. Upstream DNS configuration
Pi-hole was configured to use Cloudflare Family as its upstream resolver.

### 4. Static IP assignment
A stable static IP was assigned to the VM so clients could consistently use Pi-hole as their DNS server.

### 5. DHCP migration
DHCP service was eventually moved from the router to Pi-hole so clients would reliably receive the correct DNS server without router-side interference.

---

## Key Challenges and Troubleshooting Process

This project became much more valuable because it required real troubleshooting rather than only installation.

## 1. VMware bridged networking issues

### Problem
The VM did not always behave like a true device on the local LAN. At different stages, it had an IP that looked correct but was not fully reachable from the host or from the network.

### Cause
VMware networking was not consistently mapped to the correct physical adapter. In addition, the bridged network configuration was not always visible until VMware’s network editor was opened with administrative privileges.

### Resolution
I corrected the bridged setup by:
- opening VMware’s network editor with elevated privileges
- restoring and validating the bridged network
- binding the bridged network to the correct physical wireless adapter
- confirming the VM adapter itself was set to **Bridged**
- validating host-to-VM and VM-to-gateway connectivity

This restored reliable LAN communication for the VM.

---

## 2. Static IP and reservation conflicts

### Problem
During testing, the VM address changed unexpectedly, which created confusion when referencing the Pi-hole service from clients and from the router.

### Cause
This happened because multiple approaches were being mixed during setup, including DHCP-based leasing, reservation behavior, and manual addressing.

### Resolution
I simplified the design by moving to a single clear method:
- manually assigning a static IP inside Ubuntu
- keeping the addressing plan consistent during all further testing

This gave the Pi-hole server a stable identity on the network and reduced confusion during later troubleshooting.

---

## 3. Router DNS behavior was misleading

### Problem
When I pointed the router’s DNS setting directly to the Pi-hole VM, client internet access became unreliable.

### Cause
The router DNS setting behaved more like an upstream DNS setting for the router itself than a clean client-side DNS enforcement setting. This created confusion between:
- router DNS
- client DNS
- Pi-hole upstream DNS

### Resolution
I stopped relying on the router DNS page as the main filtering mechanism.

Instead, I designed the environment so that:
- clients use Pi-hole directly for DNS
- Pi-hole forwards to Cloudflare Family
- the router remains the default gateway only

This made the design much cleaner and easier to validate.

---

## 4. DHCP handoff had to be done carefully

### Problem
When moving DHCP responsibilities away from the router, connectivity could break if the transition was done in the wrong order.

### Cause
If router DHCP was disabled before Pi-hole DHCP was fully configured, clients could fail to receive:
- valid IP addresses
- a gateway
- a correct DNS server

### Resolution
The successful migration order was:
1. keep router DHCP enabled during Pi-hole DHCP configuration
2. fully configure Pi-hole DHCP
3. validate Pi-hole reachability and DNS operation
4. disable router DHCP only after Pi-hole was ready
5. reconnect or renew client leases

This prevented unnecessary loss of network access during migration.

---

## 5. Clients were not always using the intended DNS path

### Problem
Even after Pi-hole began issuing DHCP leases, client testing showed that DNS behavior was not always matching the expected design.

### Cause
The main issue was separating several different DNS roles:
- client-facing DNS
- router DNS
- the VM’s own operating system DNS
- Pi-hole’s upstream DNS

At different points, cached settings and previous client-side configuration also made testing more confusing.

### Resolution
I validated the design by checking:
- DHCP server values on clients
- active DNS servers on clients
- direct DNS responses from the Pi-hole server
- Pi-hole service status
- route and interface state inside Ubuntu

This made it possible to isolate where the DNS path was breaking and confirm when clients were finally using the intended resolver.

---

## What I Learned

This project gave me a much stronger understanding of how DNS filtering actually works in practice.

Key takeaways included:

- installing a service is only part of the job; validating the traffic path matters just as much
- DHCP and DNS need to be designed together
- router DNS settings do not always behave the way people assume
- virtualization adds another troubleshooting layer that can affect network services
- stable addressing is critical when a VM is acting as a shared infrastructure service
- upstream DNS, client DNS, and system DNS are separate concepts and need to be treated that way

The most important lesson from this lab was:

**A DNS filtering environment only works reliably when you clearly understand who is handing out DNS, who is forwarding DNS, and what each client is actually using.**

---

## Final Configuration Summary

### Pi-hole VM
- Ubuntu Server running in VMware
- static IP assigned manually
- Pi-hole admin interface reachable on the LAN

### Pi-hole DHCP
- DHCP enabled on Pi-hole
- lease pool assigned for client devices
- router retained as default gateway

### Pi-hole DNS
- clients use Pi-hole as their DNS server
- Pi-hole forwards allowed queries to Cloudflare Family

### Upstream Resolver
- Cloudflare Family DNS used for additional filtering

---

## Validation Performed

To confirm the environment was stable and working, I used:

### On Ubuntu
- `ip a`
- `ip route`
- `ping`
- `dig`
- `systemctl status pihole-FTL`
- `ss -tulpn | grep :53`

### On Windows client
- `ping`
- `nslookup`
- `ipconfig /all`
- `ipconfig /release`
- `ipconfig /renew`
- `ipconfig /flushdns`

### Functional validation
I confirmed:
- the Pi-hole admin page was reachable
- the Pi-hole VM responded to DNS queries
- clients could resolve normal domains
- filtered domains were blocked as expected
- DHCP leases were being served by the Pi-hole VM

---

## Project Value

This project is relevant to networking, systems administration, and cybersecurity because it demonstrates:

- Linux VM deployment
- DNS and DHCP implementation
- service troubleshooting
- network path validation
- layered filtering architecture
- the ability to diagnose issues across router, client, VM, and application layers

It also reflects a practical, hands-on approach to learning rather than only conceptual study.

---
