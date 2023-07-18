# List of Packages Installed
After installing the base system from the install iso, and after running `dnf update -y`, we installed the following using `dnf`.

  1. tmux
  2. https://dl.fedoraproject.org/pub/epel/epel-release-latest-9.noarch.rpm
  3. snapd
     1. Brave Browser
     2. 1Password
  4. wireshark
  5. xrdp
  6. vdo
  7. kmod-kvdo
  8. dkms

We started and enabled each of the systemd usits, where possible.
`vdo` and `kmod-kvdo` are not part of the UEK kernel from Oracle, so we need to change to the RHEL compatible kernel version. 
Use `grubby --default-kernel` to see the current default kernel, or `grubby --info=ALL` to see information about all installed kernels.
Once the correct kernel is identified, use`grubby --set-default={kernel name}` to update the setting.

We have setup our partitions as follows

| System | Disk | Partitions                           |
|    -   |  -   |                 -                    |
| Dalek  | sda  | not partitioned (wholly used by lvm) |
| Dalek  | sdb  | not partitioned (wholly used by lvm) |
| Dalek  | sdc  | not partitioned (wholly used by lvm) |
| Dalek  | sdd  | Partition 1: Boot (768 MB)<br>Partition 2: Swap (24 GB)<br>Partition 3: "Data" cache data (248 GB)<br>Partition 4: Extended<br>Partition 5: "Data" cache metadata (2.2 GB)<br>Partition 6: "Backup" cache data (100 GB)<br>Partition 7: "Backup" cache metadata (2.2 GB)
| Dalek  | sde  | not partitioned (wholly used by lvm) |

To setup VDO volumes we need to first create cache metadata and cache data volumes.

```bash
# Add physical volumes
pvcreate /dev/sd[abce]
pvcreate /dev/sdd[3567]

# Create volume groups
vgcreate iff /dev/sd[abce] /dev/sdd[3567]

# Create logical volumes
# Data LV:
lvcreate -n data_cache -l 100%FREE iff /dev/sdd3
lvcreate -n data_meta -l 100%FREE iff /dev/sdd5
lvconvert --type cache-pool --poolmetadata iff/data_meta iff/data_cache
lvcreate --type vdo -n data -l 100%FREE -V 16T iff/data_pool /dev/sda /dev/sdb /dev/sdc
lvvonvert --type cache --cachepool iff/data_cache iff/data
# Backup LV:
lvcreate -n backup_cache -l 100%FREE iff /dev/sdd6
lvcreate -n backup_meta -l 100%FREE iff /dev/sdd7
lvconvert --type cache-pool --poolmetadata iff/backup_meta iff/backup_cache
lvcreate --type vdo -n backup -l 100%FREE -V 1.33333T iff/backup_pool /dev/sde
lvconvert --type cache --cachepool iff/backup_cache iff/backup
```

Convert them to a cache-pool.

Create a new logical volume of type cache
