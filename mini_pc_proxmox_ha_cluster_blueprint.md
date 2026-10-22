### Mini-PC Proxmox High-Availability Cluster Blueprint

This blueprint outlines the components and configuration for building a compact, power-efficient Proxmox High-Availability cluster using mini-PCs. It focuses on redundancy, performance, and scalability within a small footprint.

#### 1. Hardware Bill-of-Materials

*   **Compute Nodes (x3-5):** Intel NUCs, HP Elitedesk/ProDesk Mini, or Dell Optiplex Micro series (e.g., i5/i7 8th Gen+, 32-64GB RAM, 250GB NVMe for OS, 1-2TB SSD for VMs/Containers). Ensure dual NICs or a USB-C/Thunderbolt to Ethernet adapter for dedicated cluster/management networks.
*   **Network Switch:** Managed Gigabit Ethernet Switch (e.g., UniFi Lite 8 PoE, TP-Link Omada) with VLAN support and ideally some PoE ports for accessories. Consider 2.5GbE or 10GbE for node interconnect if budget allows.
*   **Shared Storage (Optional, for advanced setups):** External USB 3.0 SSD/NVMe enclosure or a small NAS (e.g., Synology DS220+) for NFS/iSCSI shares. For true HA, consider a Ceph cluster across the mini-PCs.
*   **UPS:** Small Line-Interactive UPS (e.g., APC Back-UPS BX series) to protect the cluster during power fluctuations.
*   **Rack/Mounting:** VESA mounts for mini-PCs, small rack shelf or custom 3D-printed mounts for a cleaner setup.
*   **Cables:** Short, high-quality Cat6 Ethernet cables, power cables, and optionally fiber/DAC for faster interconnects.

#### 2. Rack Layout Diagram (Conceptual)

```text
+---------------------------------+
|                                 |
|  [  UPS  ] (Power Protection)   |
|                                 |
+---------------------------------+
|                                 |
|  [  Managed GbE Switch   ]     |
|  (VLANs, Inter-node Comm)       |
|                                 |
+---------------------------------+
|                                 |
|  [ Mini-PC Node 1 (Proxmox) ]   |
|  [ Mini-PC Node 2 (Proxmox) ]   |
|  [ Mini-PC Node 3 (Proxmox) ]   |
|  (VESA mounted / Stacked)       |
+---------------------------------+
|                                 |
|  [ External Storage / NAS ]     |
|  (NFS/iSCSI or Ceph OSDs)       |
|                                 |
+---------------------------------+
```

#### 3. Cabling Guide

*   **Power:** Each Mini-PC, the switch, and the optional NAS connected to the UPS. UPS connected to wall power.
*   **Network (Node 1):**
    *   `eth0` (Management/VM Bridge): Connect to Managed Switch (VLAN for management & VMs).
    *   `eth1` (Cluster/Ceph Private): Connect to Managed Switch (Dedicated VLAN for Proxmox cluster communication and Ceph traffic, if applicable).
*   **Network (Node 2, 3, etc.):** Mirror Node 1 cabling.
*   **Shared Storage:** If using an external NAS, connect its network port to the Managed Switch.

#### 4. VLAN/Network Configuration Tips

*   **VLAN 10 - Management:** For Proxmox Web UI, SSH access to nodes.
*   **VLAN 20 - Cluster Private:** Dedicated for Proxmox HA synchronization and Ceph traffic (if used). This should be a high-bandwidth, low-latency network if possible.
*   **VLAN 30 - VM/Container Network:** For VMs and LXCs to access your LAN/WAN.
*   **VLAN 40+ - Service Specific:** Create additional VLANs for specific services (e.g., IoT, Guest, Media) to enhance security and isolation.
*   **IP Addressing:** Use static IPs for all Proxmox nodes and the managed switch.
*   **DNS:** Configure reliable DNS servers (e.g., Pi-hole, AdGuard Home) on all nodes.
*   **Time Sync (NTP):** Ensure all nodes are synchronized to an NTP server for cluster stability.

#### 5. Proxmox HA Configuration Notes

*   Ensure at least 3 nodes for robust HA quorum.
*   Configure Corosync on a dedicated private network (VLAN 20).
*   Use shared storage (NFS, iSCSI, or Ceph) for VM/LXC disk images to enable live migration and HA failover.
*   Set up QDevice if running a 2-node cluster to prevent split-brain scenarios (though 3+ nodes is recommended).
