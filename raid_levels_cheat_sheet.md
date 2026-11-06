# RAID Levels Explained: Optimizing Storage for Performance & Redundancy

RAID (Redundant Array of Independent Disks) is a data storage virtualization technology that combines multiple physical disk drive components into one or more logical units for the purposes of data redundancy, performance improvement, or both.

---

## 01. RAID 0 (Striping)

*   **Description:** Data is split into blocks and written across multiple drives.
*   **Minimum Drives:** 2
*   **Capacity:** Sum of all drive capacities.
*   **Performance:** Excellent read/write speed, as data is accessed in parallel.
*   **Redundancy:** NONE. If any drive fails, all data is lost.
*   **Use Case:** Temporary data, scratch disks, video editing where speed is critical and data loss is acceptable.

---

## 02. RAID 1 (Mirroring)

*   **Description:** Data is exactly duplicated on two or more drives.
*   **Minimum Drives:** 2
*   **Capacity:** Capacity of a single drive (e.g., two 1TB drives yield 1TB usable).
*   **Performance:** Good read performance (can read from either drive), write performance is similar to a single drive.
*   **Redundancy:** HIGH. Can tolerate the failure of all but one drive in the array.
*   **Use Case:** Operating system drives, critical application data, small data sets requiring high availability.

---

## 03. RAID 5 (Striping with Parity)

*   **Description:** Data is striped across drives, and parity information (for recovery) is distributed among all drives.
*   **Minimum Drives:** 3
*   **Capacity:** (N-1) * (smallest drive capacity), where N is the number of drives.
*   **Performance:** Good read performance, decent write performance (parity calculation overhead).
*   **Redundancy:** MODERATE. Can tolerate a single drive failure.
*   **Use Case:** General-purpose storage, file servers, media servers where a balance of capacity, performance, and redundancy is needed.

---

## 04. RAID 6 (Striping with Dual Parity)

*   **Description:** Similar to RAID 5, but includes a second independent distributed parity block.
*   **Minimum Drives:** 4
*   **Capacity:** (N-2) * (smallest drive capacity), where N is the number of drives.
*   **Performance:** Good read performance, slightly slower write performance than RAID 5 (more parity calculation).
*   **Redundancy:** HIGH. Can tolerate two simultaneous drive failures.
*   **Use Case:** Large storage arrays, critical data, environments where long rebuild times increase the risk of a second drive failure.

---

## 05. RAID 10 (RAID 1+0 - Stripe of Mirrors)

*   **Description:** Combines RAID 1 (mirroring) and RAID 0 (striping). Data is mirrored in pairs, and then these mirrored pairs are striped.
*   **Minimum Drives:** 4 (in pairs of 2)
*   **Capacity:** (N/2) * (smallest drive capacity), where N is the number of drives.
*   **Performance:** Excellent read/write performance, similar to RAID 0.
*   **Redundancy:** HIGH. Can tolerate multiple drive failures, as long as no two failed drives are in the same mirrored pair.
*   **Use Case:** High-performance databases, mission-critical applications requiring both speed and high availability.

---

## CLI Example (Linux `mdadm` for Software RAID)

**Check RAID status:**
```bash
sudo mdadm --detail --scan
sudo cat /proc/mdstat
```

**Create a RAID 5 array (e.g., with /dev/sdb, /dev/sdc, /dev/sdd):**
```bash
sudo mdadm --create /dev/md0 --level=5 --raid-devices=3 /dev/sdb1 /dev/sdc1 /dev/sdd1
```

**Add a spare drive:**
```bash
sudo mdadm --add /dev/md0 /dev/sde1
```

**Replace a failed drive (assuming /dev/sdb1 failed and /dev/sde1 is the new drive):**
```bash
sudo mdadm --manage /dev/md0 --fail /dev/sdb1
sudo mdadm --manage /dev/md0 --remove /dev/sdb1
sudo mdadm --manage /dev/md0 --add /dev/sde1 # If not already added as a spare
```

**Monitor rebuild progress:**
```bash
sudo cat /proc/mdstat
```

---

## Practical Troubleshooting Workflow

1.  **Monitor Regularly:** Use tools like `mdadm --detail --scan` or `smartctl` to check drive health and RAID status.
2.  **Identify Failure:** A drive failure will often be indicated by LED lights on hardware RAID, or `mdadm --detail` will show a drive as `[F]` or `[S]` (failed/spare).
3.  **Replace Drive:** Physically replace the failed drive with a new, healthy drive of equal or greater size.
4.  **Add to Array:** If not an automatic hot-spare, use `sudo mdadm --add /dev/mdX /dev/sdY1` to add the new drive. The rebuild process will start automatically.
5.  **Verify Rebuild:** Monitor `cat /proc/mdstat` until the array is fully rebuilt and healthy.
6.  **Backup Strategy:** RAID is *not* a backup. Always have a separate, off-site backup strategy in place.
