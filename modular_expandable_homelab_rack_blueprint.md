# Anatomy of a Modular & Expandable Homelab Rack

This blueprint outlines a versatile homelab rack architecture designed for growth, balancing storage, compute, and high-performance networking. It prioritizes expandability, efficient power management, and robust data handling, making it ideal for a burgeoning self-hosting environment.

## Hardware Bill of Materials (BoM)

*   **Rack Enclosure:** 12U or 15U Wall-Mount/Floor-Standing Rack (e.g., StarTech.com, NavePoint)
*   **Uninterruptible Power Supply (UPS):** Rack-mount Sine Wave UPS (e.g., APC Smart-UPS 1500VA)
*   **Power Distribution Unit (PDU):** 1U Rack-mount PDU with surge protection (e.g., Tripp Lite, CyberPower)
*   **Compute Nodes (2x):** Mini-PCs (e.g., Intel NUC, Beelink) or 1U/2U Rack Servers (e.g., Dell R210/R220, HP Microserver Gen10+)
*   **Network Attached Storage (NAS):** 4-8 Bay NAS (e.g., Synology DS1821+, TrueNAS Mini XL+) or DIY NAS with a small form factor PC and HBA card.
*   **Managed Switch:** 10GbE Managed Switch (e.g., Mikrotik CRS305/CRS309, QNAP QSW-M2108-2S)
*   **Patch Panel:** 24-port Cat6/6a Pass-Through or Punch-Down Patch Panel
*   **Rack Shelves:** 1-2x Vented Rack Shelf (for Mini-PCs, NAS, etc.)
*   **Cabling:** Cat6a/Cat7 Ethernet cables (various lengths), SFP+ DAC cables (0.5m-3m), Fiber Optic patch cables (if applicable)
*   **Accessories:** Rack Screws & Cage Nuts, Velcro Cable Ties, Horizontal/Vertical Cable Managers

## Rack Layout Diagram (Top-Down View)

```text
+---------------------------------------+
| 1U - Patch Panel (Front)              |
|---------------------------------------|
| 1U - Managed 10GbE Switch (Front)     |
|---------------------------------------|
| 2U - NAS / Storage Server (Shelf)     |
|---------------------------------------|
| 2U - Compute Server 1 (Shelf/Rackmount)|
|---------------------------------------|
| 2U - Compute Server 2 (Shelf/Rackmount)|
|---------------------------------------|
| 1U - Rack PDU (Rear or Front, depending on model)|
|---------------------------------------|
| 2U - Rack-mount UPS (Bottom)          |
+---------------------------------------+
```

## Cabling Guide & Best Practices

1.  **Power Cabling:**
    *   Connect UPS to wall outlet. Connect PDU to UPS. Route all device power cables to the PDU. 
    *   Bundle power cables neatly using velcro ties, keeping them separate from data cables to minimize interference.
2.  **Network Data Cabling:**
    *   **Patch Panel Integration:** Run longer Ethernet cables from the rear of your servers, NAS, and other fixed devices to the back of the patch panel. Use short, color-coded patch cables from the front of the patch panel to the switch.
    *   **High-Speed Links (10GbE):** For direct connections between the switch and servers/NAS, utilize SFP+ DAC (Direct Attach Copper) cables for short runs (under 5m) for cost-effectiveness and low latency. For longer runs or inter-rack connectivity, use fiber optic transceivers and cables.
    *   **Cable Management:** Employ horizontal and vertical cable managers to route cables cleanly. Use velcro ties for bundles, allowing for easy additions or changes. Avoid zip ties for bundles that might change frequently.
3.  **Management Network (Optional but Recommended):**
    *   If devices have dedicated management interfaces (IPMI, iLO, BMC), connect them to a separate port on your switch or a dedicated small switch, and assign them to a dedicated management VLAN.

## VLAN & Network Configuration Tips

Implementing VLANs is crucial for security, performance, and network segmentation in a growing homelab.

*   **VLAN 10 - Management:**
    *   **Purpose:** Dedicated for network infrastructure devices (switch, PDU, UPS, IPMI ports). 
    *   **IP Range:** `192.168.10.0/24`
    *   **Security:** Restrict access to this VLAN heavily from other segments.
*   **VLAN 20 - Servers/Compute:**
    *   **Purpose:** For hypervisors (Proxmox, ESXi), Docker hosts, Kubernetes nodes, and core application servers.
    *   **IP Range:** `192.168.20.0/24`
    *   **Configuration:** Enable Link Aggregation (LACP) for servers with multiple NICs to the switch for redundancy and increased throughput.
*   **VLAN 30 - Storage (iSCSI/NFS/SMB):**
    *   **Purpose:** Dedicated high-bandwidth network for NAS/SAN traffic, isolated from general network traffic.
    *   **IP Range:** `192.168.30.0/24`
    *   **Performance:** Use 10GbE connections for NAS to the switch, and ensure compute nodes accessing storage are also on 10GbE if possible. Enable LACP if your NAS supports it.
*   **VLAN 40 - IoT/Guest (Future Expansion):**
    *   **Purpose:** Isolated network for smart home devices, guest Wi-Fi, or other potentially less secure devices.
    *   **IP Range:** `192.168.40.0/24`
    *   **Security:** Strict firewall rules preventing communication with other internal VLANs.

**Firewall Rules:** Configure your router/firewall to control inter-VLAN routing. By default, deny all traffic between VLANs and only explicitly permit necessary services (e.g., SSH from Management to Servers, Storage access from Servers to Storage VLAN).