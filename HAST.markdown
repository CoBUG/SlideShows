# FreeBSD's HAST
HAST or "Highly Available STorage", is a mechanism built into FreeBSD that allows
for transparent syncing of block data between two physical hosts (connected via a network).

HAST is a "primary/secondary" cluster, meaning that there will always be a single source
of truthieness (the master/primary)

---

### Advantages of a HAST Cluster ###
1. If you have an NFS export of a HAST'd volume and the ability to read the local disk vanishes,
HAST will automatically read the data from the secondary server (assuming it's available!)
2. RAID-like data consistency can be achieved

### caveats ###
1. Only two machines are supported. (Primary and Master)
2. If you are operating in a "degraded" state (local disk on primary is missing), read/write speeds
will be reduced as the TCP overhead / network latency will come into play (this overhead will change depending on the sync method used)

---

# Replication Modes

1. **memsync** - This mode will report a successful write when the following are met:
   1. After the local write is successful.
   2. Remote node sends acknowledgement of data receipt. (Receipt is sent prior to writing data)
2. **fullsync** - The safest of the replication modes, fullsync waits for write of the data on *both* hosts prior to marking a write successful.
3. **async** - This is the most dangerous mode. Once data is written to the local disk, the write operation is considered successful. If you are on a high latency network, this is the option you want to used.

---

# Example Config

For the sake of the example, assume we have two machines (fbsdA and fbsdB) and both have a `/dev/da0` disk which contains the data we want to sync.

The config file (/etc/hast.conf) must be the same on both the primary and secondary machines!

```
listen tcp://0.0.0.0

resource shared {
    local /dev/ada0p4

    on fbsd_h1 {
        remote tcp://10.0.0.231
    }
    on fbsd_h2 {
         remote tcp://10.0.0.233
    }
}
```

---

Now we can start up hastd on each client and tell the cluster which host is the master!

```
fbsd_h1# hastctl create shared
fbsd_h1# hastd
fbsd_h1# hastctl role primary shared
fbsd_h1# newfs -U /dev/hast/shared
fbsd_h1# mount -o noatime /dev/hast/shared /data
```

```
fbsd_h2# hastctl create shared
fbsd_h2# hastd
fbsd_h2# hastctl role secondary shared
```

---
If all goes to plan when you run `hastctl status` you should see something like this:
```
fbsd_h1# hastctl status
Name	 Status		Role	Components
shared	 complete	primary	/dev/ada0p4	tcp://10.0.1.233
```

```
fbsd_h2# hastctl status
Name	 Status		Role		Components
shared	 complete	secondary	/dev/ada0p4	tcp://10.0.1.231
```
---