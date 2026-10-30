# Network Troubleshooting Commands Cheat Sheet

Diagnosing network issues is a fundamental skill for any sysadmin or homelab enthusiast. This cheat sheet provides a quick reference for essential command-line tools used to troubleshoot connectivity, routing, and DNS problems.

---

## 01. `ping` - Network Host Reachability

**Purpose:** Tests the reachability of a host on an Internet Protocol (IP) network and measures the round-trip time for messages sent from the originating host to a destination computer.

**Syntax:**
```bash
ping <destination_ip_or_hostname>
```

**Common Options:**
- `-c <count>`: Stop after sending `count` ECHO_REQUEST packets.
- `-i <interval>`: Wait `interval` seconds between sending packets.
- `-t <ttl>` (Windows): Set the Time To Live.
- `-q`: Quiet output.

**Practical Use Case:**
- Verify if a server or device is online and responsive.
- Measure basic network latency between two points.

**Example:**
```bash
ping google.com
ping 192.168.1.1 -c 4
```

---

## 02. `traceroute` / `tracert` - Packet Path Mapping

**Purpose:** Displays the route and measures transit delays of packets across an Internet Protocol (IP) network. It identifies all intermediate routers (hops) between the source and destination.

**Syntax (Linux/macOS):**
```bash
traceroute <destination_ip_or_hostname>
```

**Syntax (Windows):**
```bash
tracert <destination_ip_or_hostname>
```

**Common Options:**
- `-n`: Do not resolve IP addresses to hostnames.
- `-m <max_hops>`: Set the maximum number of hops.
- `-p <port>`: Set the destination port for UDP (Linux).

**Practical Use Case:**
- Pinpoint where network latency or a connectivity issue is occurring (e.g., which router is dropping packets).
- Identify the path your traffic takes to an external service.

**Example:**
```bash
traceroute example.com
tracert 8.8.8.8
```

---

## 03. `netstat` / `ss` - Network Connections & Statistics

**Purpose:**
- **`netstat` (Network Statistics):** Displays active TCP connections, listening ports, routing tables, and network interface statistics.
- **`ss` (Socket Statistics):** A faster, more efficient modern replacement for `netstat` for displaying socket information.

**Syntax:**
```bash
netstat [options]
ss [options]
```

**Common Options (`netstat` / `ss`):
- `-a` / `-a`: Display all sockets (listening and non-listening).
- `-t` / `-t`: Display TCP connections.
- `-u` / `-u`: Display UDP connections.
- `-l` / `-l`: Display listening sockets.
- `-n` / `-n`: Do not resolve service names (show port numbers).
- `-p` / `-p`: Show the PID and name of the program owning the socket.
- `-r` / `-r`: Display the kernel routing tables.

**Practical Use Case:**
- See what services are listening on which ports (e.g., troubleshooting a web server not starting).
- Identify active connections to and from your machine.
- Check open ports for security audits.

**Example (`ss` is preferred):**
```bash
ss -tuln # Show all TCP/UDP listening ports without resolving names
ss -antp # Show all active TCP connections with process info
```

---

## 04. `ip` - Show / Manipulate Routing, Devices, Interfaces

**Purpose:** A powerful utility to show and manipulate routing, network devices, interfaces, and tunnels. It replaces older tools like `ifconfig` and `route` on modern Linux systems.

**Syntax:**
```bash
ip [OPTIONS] OBJECT { COMMAND | help }
```

**Common Objects & Commands:**
- **`ip addr`**: Display / manipulate IP addresses.
  - `ip addr show`: Show IP addresses and interface details.
  - `ip addr add <ip/mask> dev <interface>`: Add an IP address.
- **`ip link`**: Display / manipulate network devices.
  - `ip link show`: Show network interfaces.
  - `ip link set <interface> up/down`: Bring an interface up/down.
- **`ip route`**: Display / manipulate the routing table.
  - `ip route show`: Show the routing table.
  - `ip route add default via <gateway_ip>`: Add a default route.
- **`ip neigh`**: Display / manipulate neighbor (ARP) entries.
  - `ip neigh show`: Show ARP cache.

**Practical Use Case:**
- Configure static IP addresses or change network interface states.
- View detailed statistics for network interfaces.
- Troubleshoot routing issues by inspecting the routing table.

**Example:**
```bash
ip addr show eth0
ip route show
ip link show
```

---

## 05. `dig` / `nslookup` - DNS Query Utility

**Purpose:**
- **`dig` (Domain Information Groper):** A flexible tool for interrogating DNS name servers. It performs DNS lookups and displays the answers that are returned from the name server.
- **`nslookup` (Name Server Lookup):** A network administration command-line tool for querying the Domain Name System (DNS) to obtain domain name or IP address mapping or for any other specific DNS record. `dig` is generally preferred for advanced use.

**Syntax:**
```bash
dig <hostname> [record_type] [@nameserver]
nslookup <hostname> [nameserver]
```

**Common Options (`dig`):**
- `A`: Address record (default).
- `MX`: Mail exchange record.
- `TXT`: Text record.
- `ANY`: Any record type.
- `+short`: Display only the answer.

**Practical Use Case:**
- Verify if DNS resolution is working correctly for a specific domain.
- Troubleshoot issues where a service is unreachable by hostname but accessible by IP.
- Check DNS records (e.g., MX for email, TXT for SPF/DKIM).

**Example:**
```bash
dig example.com A
dig example.com MX +short
nslookup my-server.lan 192.168.1.1 # Query specific DNS server
```

---

Mastering these commands will equip you with the essential toolkit to diagnose and resolve a wide array of network-related challenges in your homelab and beyond.