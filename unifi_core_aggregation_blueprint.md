# Optimized UniFi Core: UDM Pro Max & Aggregation Switch Blueprint

This blueprint outlines a high-performance UniFi network core centered around the UniFi Dream Machine Pro Max (UDM Pro Max) and a UniFi Aggregation Switch. This architecture provides a scalable, secure, and high-bandwidth foundation suitable for demanding homelabs or small business environments, emphasizing 10GbE capabilities and intelligent power delivery.

## Hardware Bill of Materials

| Component                          | Model / Description                                | Quantity | Notes                                                                   |
| :--------------------------------- | :------------------------------------------------- | :------- | :---------------------------------------------------------------------- |
| **Network Core**                   |                                                    |          |                                                                         |
| UniFi Dream Machine Pro Max        | UDM-Pro-Max                                        | 1        | Central gateway, firewall, controller, NVR.                             |
| UniFi Aggregation Switch           | USW-Aggregation                                    | 1        | 8x 10GbE SFP+ ports for high-speed inter-switch links and server connectivity. |
| UniFi Switch Pro 24 PoE            | USW-Pro-24-PoE                                     | 1-2      | 24x GbE PoE+ ports, 2x 10GbE SFP+ uplinks for APs, cameras, endpoints. |
| UniFi Access Point                 | U6 Enterprise / U6 Pro                             | 2-4      | Wi-Fi 6E/6 for high-density wireless coverage.                          |
| **Rack & Power**                   |                                                    |          |                                                                         |
| StarTech 12U Rack                  | RK12WALHM                                          | 1        | Wall-mount or floor-standing, depending on space.                       |
| APC Smart-UPS 1500VA               | SMT1500RM2U                                        | 1        | Rack-mount UPS for network core.                                        |
| UniFi SmartPower PDU Pro           | USP-PDU-Pro                                        | 1        | Managed PDU with outlet control and power monitoring.                   |
| **Cabling**                        |                                                    |          |                                                                         |
| Ubiquiti UniFi DAC Cable           | UDC-0.5M (0.5m) / UDC-1M (1m)                      | 2-4      | For 10GbE links between UDM Pro Max and Aggregation Switch.             |
| Cat6A Patch Cables                 | Various lengths (0.5m, 1m, 2m)                     | 10-20    | For device connectivity to PoE switches.                                |
| Fiber Patch Cables (LC-LC)         | OM3 Multimode (1m, 2m)                             | 2-4      | If extending 10GbE to other racks/devices via fiber.                    |
| **Accessories**                    |                                                    |          |                                                                         |
| Blanking Panels                    | 1U / 2U                                            | 4-6      | For airflow management and aesthetics.                                  |
| Cable Management Panels            | 1U with D-rings                                    | 2-3      | Essential for clean routing.                                            |

## Rack Layout Diagram (ASCII)

```
+-------------------------------------------------------------+
| [   1U Blank Panel                                      ] |
|-------------------------------------------------------------|
| [   1U Cable Management Panel                           ] |
|-------------------------------------------------------------|
| [   1U UniFi Dream Machine Pro Max (UDM-Pro-Max)        ] |
|-------------------------------------------------------------|
| [   1U UniFi Aggregation Switch (USW-Aggregation)       ] |
|-------------------------------------------------------------|
| [   1U Cable Management Panel                           ] |
|-------------------------------------------------------------|
| [   1U UniFi Switch Pro 24 PoE (USW-Pro-24-PoE)         ] |
|-------------------------------------------------------------|
| [   1U UniFi Switch Pro 24 PoE (USW-Pro-24-PoE)         ] |
|-------------------------------------------------------------|
| [   1U Cable Management Panel                           ] |
|-------------------------------------------------------------|
| [   2U APC Smart-UPS 1500VA (SMT1500RM2U)               ] |
|-------------------------------------------------------------|
| [   1U UniFi SmartPower PDU Pro (USP-PDU-Pro)           ] |
|-------------------------------------------------------------|
| [   1U Blank Panel                                      ] |
+-------------------------------------------------------------+
```

## Cabling Guide

1.  **UDM Pro Max to Aggregation Switch:**
    *   Connect UDM Pro Max's 10GbE SFP+ LAN port (e.g., Port 10) to a 10GbE SFP+ port on the UniFi Aggregation Switch using a Ubiquiti DAC cable (0.5m or 1m). This provides the high-speed backbone for inter-VLAN routing and server connectivity.

2.  **Aggregation Switch to PoE Switches:**
    *   Connect 10GbE SFP+ ports on the Aggregation Switch to the 10GbE SFP+ uplinks on the UniFi Switch Pro 24 PoE units using DAC cables or fiber (if switches are further apart). This ensures high-bandwidth connectivity to all downstream wired devices and access points.

3.  **PoE Switches to End Devices:**
    *   Use Cat6A patch cables to connect UniFi Access Points, UniFi Protect cameras, desktops, servers, and other network devices to the Gigabit PoE+ ports on the UniFi Switch Pro 24 PoE.
    *   Organize cables using the cable management panels, routing them cleanly to avoid clutter and improve airflow.

4.  **Power:**
    *   Connect the UDM Pro Max, Aggregation Switch, PoE switches, and any other rack-mounted devices to the UniFi SmartPower PDU Pro.
    *   Connect the SmartPower PDU Pro to the APC Smart-UPS.
    *   Connect the Smart-UPS to a dedicated wall outlet.

## VLAN / Network Configuration Tips

1.  **Default Network (VLAN 1):** Keep this for management traffic only (UniFi devices, core infrastructure). Assign a small subnet (e.g., 192.168.1.0/24).
2.  **IoT Network (VLAN 10):** Isolate all smart home devices. Create a new network in UniFi Controller, assign VLAN ID 10, and enable client device isolation. Block inter-VLAN routing to other sensitive networks.
3.  **Guest Network (VLAN 20):** Completely isolated network for guests. Use UniFi's built-in guest portal features.
4.  **Main/Trusted Network (VLAN 30):** For personal devices, desktops, laptops.
5.  **Server Network (VLAN 40):** For self-hosted applications, NAS, Proxmox hosts. Configure firewall rules on the UDM Pro Max to control access between this network and others.
6.  **UniFi Protect Network (VLAN 50):** Dedicated network for UniFi Protect cameras and NVR (if not using UDM Pro Max's internal NVR).
7.  **Inter-VLAN Routing:** The UDM Pro Max will handle all inter-VLAN routing. Leverage its firewall capabilities to define granular access control rules between each VLAN based on the principle of least privilege.
8.  **SSID to VLAN Mapping:** Map each Wi-Fi SSID to its corresponding VLAN in the UniFi Controller (e.g., "MyHome_IoT" to VLAN 10, "MyHome_Guest" to VLAN 20).
9.  **Link Aggregation (LAG) / LACP:** While the Aggregation Switch provides 10GbE, consider LACP for NAS or servers with multiple NICs connected to the Aggregation Switch to increase bandwidth and provide redundancy.
