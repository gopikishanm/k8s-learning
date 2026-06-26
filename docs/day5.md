## Day 5

### Why Grafana is slow

Grafana uses SqLite as its default database. A single-file embedded database that lives at /var/lib/grafana/grafana.db

SQLite is a single-writer database. It uses file-level locking to ensure only one process writes at a time. When you put that file on shared network storage like EFS and point multiple pods at it, you’re asking SQLite to coordinate writes across the network which breaks it.

Amazon EFS is essentially managed NFS (NFSv4.1). It’s excellent for many use cases — shared configuration, media files, log aggregation. But SQLite is not one of them.

In the context of NFS (Network File System), fsync is a system call that ensures all modified file data and metadata on a client are safely flushed and committed to the server's stable storage (e.g., a physical disk) before the operation completes.

On EBS, fsync takes less than 1ms while for EFS it is 10-50 ms. 

When we use multiple replicas for Grafana and all try to write to EFS, all pods are fighting for locking the sqlite db and update it. 

#### Infrequent Access Penalty

Amazon EFS has a cost-optimization feature called Infrequent Access (IA). Files not accessed within a configurable period (7 days in our case) are automatically moved to a cheaper storage tier. 

But IA-tier reads add an extra 100–200ms of latency on first access. This is on top of the already-slow NFS operations. 

#### Quick Fix

One Grafana replica with EBS backend which can be expanded as required.

#### Longterm Fix

Use a proper PostgreSQL or MySql database.

### References
- [Why Grafana is slow](https://medium.com/@j.aslanov94/why-your-grafana-is-slow-on-kubernetes-and-3-replicas-wont-fix-it-f375527de85a)

