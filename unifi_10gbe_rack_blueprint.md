# Anatomy of an All-UniFi 10GbE Network Rack

## Description
A detailed blueprint for a high-performance, privacy-focused UniFi 10GbE network rack, emphasizing robust infrastructure and meticulous organization. This setup prioritizes speed, security, and the seamless integration of the Ubiquiti UniFi ecosystem.

## Hardware Bill-of-Materials
*   **Network Gateway**: UniFi Dream Machine Special Edition (UDM-SE)
*   **Core Switching**: UniFi Switch Aggregation (USW-Aggregation) - for 10GbE fiber backbone
*   **PoE & Edge Switching**: UniFi Switch Enterprise 8 PoE (USW-Enterprise-8-PoE) - for 2.5GbE/PoE devices
*   **Surveillance NVR**: UniFi Network Video Recorder Pro (UNVR-Pro)
*   **Wireless Access Points**: UniFi Access Point U6 Enterprise (U6-Enterprise) - (Multiple, deployed throughout location, not rack-mounted)
*   **Uninterruptible Power Supply (UPS)**: Rackmount UPS (e.g., Eaton 5P 1500VA)
*   **Power Distribution**: Rackmount PDU (Power Distribution Unit)
*   **Storage**: Rackmount Shelf (for a Synology/TrueNAS Mini or similar NAS)
*   **Patch Panels**: Fiber Optic Patch Panel, Ethernet Cat6a Patch Panel
*   **Rack Enclosure**: 12U Rack Cabinet (e.g., StarTech, NavePoint)
*   **Cables**: SFP+ DACs, Fiber Optic Patch Cables, Cat6a Ethernet Patch Cables (various lengths)

## Rack Layout Diagram (ASCII)
```text
+---------------------------------+
| 12U Rack Cabinet                |
+---------------------------------+
| U12: Empty / Future Expansion   |
|---------------------------------|
| U11: UniFi NVR Pro              |
|---------------------------------|
| U10: UniFi Switch Aggregation   |
|---------------------------------|
| U09: UniFi Switch Enterprise 8 PoE|
|---------------------------------|
| U08: UniFi Dream Machine SE     |
|---------------------------------|
| U07: Fiber Patch Panel          |
|---------------------------------|
| U06: Ethernet Patch Panel       |
|---------------------------------|
| U05: Rackmount Shelf (NAS/MiniPC)|
|---------------------------------|
| U04: Rackmount PDU              |
|---------------------------------|
| U03: UPS (Rackmount)            |
|---------------------------------|
| U02: UPS (Rackmount)            |
|---------------------------------|
| U01: Blank Panel                |
+---------------------------------+
```

## Cabling Guide
1.  **Power Distribution**: All active rack components connect to the Rackmount PDU. The PDU is fed by the rackmount UPS, which should be connected to a dedicated electrical circuit if possible.
2.  **Network Backbone (Fiber)**: Connect UDM-SE's SFP+ LAN port to the USW-Aggregation's SFP+ port using a 10GbE SFP+ DAC. Connect other SFP+ ports on the USW-Aggregation to the Fiber Patch Panel to distribute 10GbE fiber links to other switches or high-bandwidth devices.
3.  **Network Data (Ethernet)**: Connect UDM-SE's RJ45 LAN ports to the Ethernet Patch Panel. Connect the USW-Enterprise-8-PoE's RJ45 ports (including 2.5GbE PoE ports) to the Ethernet Patch Panel. Use short, color-coded patch cables from the patch panels to the active equipment.
4.  **Storage/Surveillance**: Connect the UNVR-Pro and the NAS (on the shelf) to the USW-Enterprise-8-PoE for data and power (if PoE-capable).
5.  **Cable Management**: Utilize vertical cable raceways along the sides of the rack and horizontal cable managers between patch panels and switches. Label every cable clearly at both ends. Consider color-coding cables by function (e.g., blue for data, yellow for PoE, orange for fiber).

## VLAN & Network Configuration Tips
*   **VLAN Structure**:
    *   `VLAN 10: Management` (For UniFi devices, NAS management interfaces, critical infrastructure)
    *   `VLAN 20: Trusted` (Main LAN for desktops, laptops, personal devices)
    *   `VLAN 30: IoT` (Isolated for smart home devices, strict firewall rules preventing IoT-to-Trusted communication)
    *   `VLAN 40: Guest` (Fully isolated, bandwidth-limited network for visitors)
    *   `VLAN 50: Surveillance` (Dedicated for UniFi Protect cameras and UNVR-Pro)
    *   `VLAN 60: Servers` (For Docker hosts, self-hosted applications, VMs)
*   **Firewall Rules**: Implement default deny-all inter-VLAN routing and create specific allow rules for necessary communication (e.g., Management VLAN to UniFi devices, Trusted VLAN to Servers VLAN for app access). Enable Geo-blocking and Intrusion Detection/Prevention System (IDS/IPS) on the UDM-SE.
*   **DNS**: Configure UDM-SE to use a self-hosted DNS sinkhole (e.g., AdGuard Home, Pi-hole) or privacy-focused public DNS servers for all VLANs.
*   **DHCP**: Utilize the UDM-SE's built-in DHCP server for all configured VLANs.
*   **10GbE Optimization**: Ensure all critical backbone links (UDM-SE to USW-Aggregation, USW-Aggregation to high-demand servers/NAS) utilize 10GbE SFP+ DACs or fiber for maximum throughput.