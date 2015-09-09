# FreeNAS

Storage liberated!

---

## Features

- ZFS
  * Replication
  * Data integrity
  * Encryption
  * Snapshots
- Easy web manager
- Jailed plugins
- Lots more!

---

## ZFS

- Automatic striping across devices
- Fault-tolerant / self healing
- Copy-on-write (COW)/ snapshots
- Machine independent
- Volume manager

---

### ZFS COW

- On write, copy block and write to the copy
- When copy finishes, pointers are updated to reference block copy + freshly written data
- Want snapshots? 

---

### ZFS Snapshots

COW allows for a few different kinds of "snapshots" to be created:
- **snapshots**: point in time reference to a set of blocks.
- **clones**: filesystem that references blocks at a point in time.

Think of **clones** like a directory that contains old data symlinked in.

---

### ZFS Self Healing

- Every block has a 256-bit checksum associated with it.
- Validated every read.
- Data can be *healed* from another copy if checksums do not match.
- **scrub** reads everything, should be used regularly on low access data!

---

### ZFS Partitions?

*We don't need no stinking partitions?*

- **vdev**: files, hard drive partitions or entire disks.
  - Usage of entire drive is recommended as it simplifies things greatly.
- **zpool**: a collection of one or more **vdevs** (think of a zvol as a raw disk device).
- **datasets**: chop up a zpool to allow differing permissions, quotas and compression per dataset. 

---

# Web Manager

- The web manager covers a very broad range of tasks

*Word of caution*: The web ui does not use https by default. Only access it from a trusted network during initial setup!

---

# Jailed Plugins

- Utilizes FreeBSD's jail(8) and PC-BSD's Push Button Installer (PBI).
- Allows for isolated applications.
- Fine grain access control to zpools can be leveraged.

---

# Setup

[if you don't see a live demo - the presenter needs more beer!]
---

# Uses

What can't you use it for? :P

