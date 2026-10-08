# Anatomy of a Silent 10GbE Homelab Rack

This blueprint outlines the essential components and best practices for constructing a high-performance, low-noise 10 Gigabit Ethernet (10GbE) homelab rack. The focus is on robust infrastructure, efficient cooling, and meticulous organization.

## Hardware Bill-of-Materials

### Core Infrastructure
- **Rack Enclosure**: 12U-15U Soundproof/Quiet Rack Cabinet (e.g., StarTech.com 12U Soundproof Cabinet)
- **UPS**: APC Smart-UPS SMT1500C (1500VA/1000W) or similar line-interactive/pure sine wave UPS
- **PDU**: Rack-mountable PDU with individual outlet control (e.g., CyberPower PDU15M10F)
- **Rack Fans**: Noctua NF-A12x25 PWM (for quiet active cooling, if rack has fan mounts)
- **Cable Management**: Vertical and Horizontal Cable Managers, Velcro cable ties

### Networking
- **10GbE Switch**: Managed L3 10GbE Switch (e.g., MikroTik CRS309-1G-8S+IN or a fanless SFP+ switch)
- **Router/Firewall**: Mini-PC with pfSense/OPNsense (e.g., Protectli Vault, Qotom i7)
- **Wireless Access Point**: Ceiling-mount Wi-Fi 6/6E AP (e.g., TP-Link Omada EAP670, Aruba Instant On AP22)

### Compute
- **Server 1 (Proxmox Host)**: Dell OptiPlex Micro/HP Elitedesk Mini (i5/i7, 32GB RAM, 2x NVMe for OS/VMs)
  - *Network Card*: Intel X520-DA1/DA2 10GbE SFP+ NIC (if not built-in)
- **Server 2 (Backup/Secondary)**: Similar Mini-PC or Raspberry Pi 4/5 Cluster (for lightweight services)

### Storage
- **NAS (Network Attached Storage)**: Synology DS1821+ / DS1621+ or custom TrueNAS Scale build
  - *Drives*: 4-8x Western Digital Red Pro / Seagate IronWolf Pro HDDs (4TB-16TB)
  - *Cache*: 2x NVMe SSDs for read/write cache (e.g., Samsung 970 EVO Plus)
  - *Network Card*: 10GbE SFP+ NIC (if not built-in, for TrueNAS)

## Rack Layout Diagram (ASCII)

```
+-------------------------------------+
| [ ] Front Panel                     |
|                                     |
| 1U - Cable Management               |
| 1U - PDU (Power Distribution Unit)  |
| 2U - UPS (Uninterruptible Power)    |
| 1U - Router/Firewall (Mini-PC)      |
| 1U - 10GbE Managed Switch           |
| 2U - Server 1 (Proxmox Host)        |
| 1U - Server 2 (Lightweight Services)|
| 3U - NAS (Network Attached Storage) |
| 1U - Cable Management               |
|                                     |
| [ ] Rear Panel                      |
+-------------------------------------+
```

## Cabling Guide

1.  **Power**:
    *   All devices connect to the PDU.
    *   PDU connects to the UPS.
    *   UPS connects to a dedicated wall outlet.
    *   Use appropriately sized C13/C14 cables; keep lengths minimal to reduce clutter.

2.  **Networking (Data)**:
    *   **Router/Firewall**: WAN port to ISP modem. LAN port(s) to 10GbE switch.
    *   **10GbE Switch**: Central hub for all wired devices.
        *   SFP+ ports: Use DAC cables for direct connections to servers/NAS where possible (less latency, lower power).
        *   RJ45 ports: Use Cat6A/Cat7 patch cables for 10GbE devices, Cat6 for 1GbE devices.
    *   **Servers/NAS**: Connect 10GbE NICs to SFP+ ports on the switch using DACs or fiber transceivers + fiber patch cables.
    *   **Wireless AP**: Connect to a PoE port on the 10GbE switch (if switch supports PoE, otherwise use PoE injector).
    *   **Cable Management**:
        *   Bundle cables loosely with Velcro ties.
        *   Route power cables on one side of the rack and data cables on the other to minimize interference.
        *   Utilize horizontal and vertical cable managers for clean routing.
        *   Label *all* cables at both ends.

## VLAN/Network Configuration Tips

This setup assumes a robust VLAN strategy for enhanced security and traffic management.

1.  **Main Network (VLAN 10)**: 192.168.10.0/24 - General purpose, trusted devices, management interfaces.
2.  **IoT Network (VLAN 20)**: 192.168.20.0/24 - Isolated network for smart home devices, restricted internet access, no LAN access.
3.  **Guest Network (VLAN 30)**: 192.168.30.0/24 - Fully isolated, internet-only access.
4.  **Servers Network (VLAN 40)**: 192.168.40.0/24 - Dedicated for server VMs, NAS, and infrastructure services. Strict firewall rules.
5.  **Storage Network (VLAN 50)**: 192.168.50.0/24 - *Optional, if using iSCSI/NFS between compute and storage*. Dedicated high-speed network for storage traffic only, no internet access.

### Configuration Steps (General)

*   **Router/Firewall (pfSense/OPNsense)**:
    *   Create VLAN interfaces for each network.
    *   Configure DHCP servers for each VLAN.
    *   Set up firewall rules:
        *   Default deny all between VLANs.
        *   Allow specific traffic: e.g., VLAN 10 to VLAN 40 (management), VLAN 40 to VLAN 50 (storage), VLANs 10, 20, 30 to WAN.
        *   Implement DNS filtering (e.g., Pi-hole/AdGuard Home in VLAN 40, accessible from other VLANs via firewall rules).
*   **10GbE Switch**:
    *   Configure switch ports as 'trunk' for connections to the router/firewall and servers, allowing all necessary VLANs.
    *   Configure 'access' ports for individual devices on their respective VLANs (e.g., AP port as trunk for multiple SSIDs, or specific IoT devices on VLAN 20).
    *   Enable Jumbo Frames (MTU 9000) on 10GbE ports for improved performance if supported by all devices.
*   **Servers (Proxmox/TrueNAS)**:
    *   Configure network interfaces to use VLAN tagging for their respective virtual networks.
    *   Ensure VMs/containers are assigned to the correct virtual bridges/interfaces associated with the VLANs.

This blueprint provides a foundation for a powerful, quiet, and secure homelab environment. Adapt components and configurations to your specific needs and budget.