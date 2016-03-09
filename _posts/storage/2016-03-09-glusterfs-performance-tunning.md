---
layout: post
title: GlusterFS 配置选项
modified:
categories: storage
excerpt: "GlusterFS选项说明及oVirt对GlusterFS的优化配置"
tags: [Gluster]
comments: true
image:
  feature:
date: 2016-03-09T21:33:44+08:00
---

## 选项说明

~~~ bash
$ gluster volume set help
~~~

| 选项 | 默认值 | 说明 
|:--------|:-------:|:--------
| changelog.changelog-barrier-timeout | 120 | After 'timeout' seconds since the time 'barrier' option was set to "on", unlink/rmdir/rename  operations are no longer blocked and previously blocked fops are allowed to go through
| cluster.enable-shared-storage | disable | Create and mount the shared storage volume(gluster_shared_storage) at /var/run/gluster/shared_storage on enabling this option. Unmount and delete the shared storage volume  on disabling this option.
| cluster.write-freq-threshold | 0 | Defines the number of writes, in a promotion/demotion cycle, that would mark a file HOT for promotion. Any file that has write hits less than this value will be considered as COLD and will be demoted.
| cluster.read-freq-threshold | 0 | Defines the number of reads, in a promotion/demotion cycle, that would mark a file HOT for promotion. Any file that has read hits less than this value will be considered as COLD and will be demoted.
| cluster.tier-pause | off | (null)
| cluster.tier-promote-frequency | 120 | Frequency to promote files to fast tier
| cluster.tier-demote-frequency | 120 | Frequency to demote files to slow tier
| cluster.watermark-hi | 90 | Upper % watermark for promotion. If hot tier fills above this percentage, no promotion will happen and demotion will happen with high probability.
| cluster.watermark-low | 75 | Lower % watermark. If hot tier is less full than this, promotion will happen and demotion will not happen. If greater than this, promotion/demotion will happen at a probability relative to how full the hot tier is.
| cluster.tier-mode | cache | Either 'test' or 'cache'. Test mode periodically demotes or promotes files automatically based on access. Cache mode does so based on whether the cache is full or not, as specified with watermarks.
| cluster.tier-max-mb | 10000 | The maximum number of MB that may be migrated in any direction in a given cycle by a single node.
| cluster.tier-max-files | 50000 | The maximum number of files that may be migrated in any direction in a given cycle by a single node.
| cluster.lookup-unhashed | on   | This option if set to ON, does a lookup through all the sub-volumes, in case a lookup didn't return any result from the hash subvolume. If set to OFF, it does not do a lookup on the remaining subvolumes.   |
| cluster.lookup-optimize | off  | This option if set to ON enables the optimization of -ve lookups, by not doing a lookup on non-hashed subvolumes for files, in case the hashed subvolume does not return any result. This option disregards the lookup-unhashed setting, when enabled.   |
| cluster.min-free-disk | 10% | Percentage/Size of disk space, after which the process starts balancing out the cluster, and logs will appear in log files
| cluster.min-free-inodes | 5% | after system has only N% of inodes, warnings starts to appear in log files
| cluster.rebalance-stats | off | This option if set to ON displays and logs the  time taken for migration of each file, during the rebalance process. If set to OFF, the rebalance logs will only display the time spent in each directory.
| cluster.subvols-per-directory | null | Specifies the directory layout spread. Takes number of subvolumes as default value.
| cluster.readdir-optimize | off | This option if set to ON enables the optimization that allows DHT to requests non-first subvolumes to filter out directory entries.
| cluster.rebal-throttle | normal | Sets the maximum number of parallel file migrations allowed on a node during the rebalance operation. The default value is normal and allows a max of [($(processing units) - 4) / 2), 2]  files to be migrated at a time. Lazy will allow only one file to be migrated at a time and aggressive will allow max of [($(processing units) - 4) / 2), 4]
| cluster.weighted-rebalance | on | When enabled, files will be allocated to bricks with a probability proportional to their size.  Otherwise, all bricks will have the same probability (legacy behavior).
| cluster.entry-change-log | on | Entry fops like create/unlink will not perform pre/post fop changelog operations in afr transaction if this option is disabled
| cluster.read-subvolume | null | inode-read fops happen only on one of the bricks in replicate. Afr will prefer the one specified using this option if it is not stale. Option value must be one of the xlator names of the children. Ex: <volname>-client-0 till <volname>-client-<number-of-bricks - 1>
| cluster.read-subvolume-index | -1 | inode-read fops happen only on one of the bricks in replicate. AFR will prefer the one specified using this option if it is not stale. allowed options include -1 till replica-count - 1
| cluster.read-hash-mode | 1 | inode-read fops happen only on one of the bricks in replicate. AFR will prefer the one computed using the method specified using this option0 = first up server, 1 = hash by GFID of file (all clients use same subvolume), 2 = hash by GFID of file and client PID
| cluster.background-self-heal-count | 16 | This specifies the number of self-heals that can be  performed in background without blocking the fop
| cluster.metadata-self-heal | on | Using this option we can enable/disable metadata i.e. Permissions, ownerships, xattrs self-heal on the file/directory.
| cluster.data-self-heal | on | Using this option we can enable/disable data self-heal on the file. "open" means data self-heal action will only be triggered by file open operations.
| cluster.entry-self-heal | on | Using this option we can enable/disable entry self-heal on the directory.
| cluster.self-heal-daemon | on | This option applies to only self-heal-daemon. Index directory crawl and automatic healing of files will not be performed if this option is turned off.
| cluster.heal-timeout | 600 | time interval for checking the need to self-heal in self-heal-daemon
| cluster.self-heal-window-size | 1 | Maximum number blocks per file for which self-heal process would be applied simultaneously.
| cluster.data-change-log | on | Data fops like write/truncate will not perform pre/post fop changelog operations in afr transaction if this option is disabled
| cluster.metadata-change-log | on | Metadata fops like setattr/setxattr will not perform pre/post fop changelog operations in afr transaction if this option is disabled
| cluster.data-self-heal-algorithm | null | Select between "full", "diff". The "full" algorithm copies the entire file from source to sink. The "diff" algorithm copies to sink only those blocks whose checksums don't match with those of source. If no option is configured the option is chosen dynamically as follows: If the file does not exist on one of the sinks or empty file exists or if the source file size is about the same as page size the entire file will be read and written i.e "full" algo, otherwise "diff" algo is chosen.
| cluster.eager-lock | on | Lock phase of a transaction has two sub-phases. First is an attempt to acquire locks in parallel by broadcasting non-blocking lock requests. If lock acquisition fails on any server, then the held locks are unlocked and revert to a blocking locked mode sequentially on one server after another.  If this option is enabled the initial broadcasting lock request attempt to acquire lock on the entire file. If this fails, we revert back to the sequential "regional" blocking lock as before. In the case where such an "eager" lock is granted in the non-blocking phase, it gives rise to an opportunity for optimization. i.e, if the next write transaction on the same FD arrives before the unlock phase of the first transaction, it "takes over" the full file lock. Similarly if yet another data transaction arrives before the unlock phase of the "optimized" transaction, that in turn "takes over" the lock as well. The actual unlock now happens at the end of the last "optimized" transaction.
| cluster.quorum-type | none | If value is "fixed" only allow writes if quorum-count bricks are present.  If value is "auto" only allow writes if more than half of bricks, or exactly half including the first, are present.
| cluster.quorum-count | null | If quorum-type is "fixed" only allow writes if this many bricks or present.  Other quorum types will OVERWRITE this value.
| cluster.choose-local | true | Choose a local subvolume (i.e. Brick) to read from if read-subvolume is not explicitly set.
| cluster.self-heal-readdir-size | 1KB | readdirp size for performing entry self-heal
| cluster.ensure-durability | on | Afr performs fsyncs for transactions if this option is on to make sure the changelogs/data is written to the disk
| cluster.consistent-metadata | no | If this option is enabled, readdirp will force lookups on those entries read whose read child is not the same as that of the parent. This will guarantee that all read operations on a file serve attributes from the same subvol as long as it holds  a good copy of the file/dir.
| cluster.stripe-block-size | 128KB | Size of the stripe unit that would be read from or written to the striped servers.
| cluster.stripe-coalesce | true | Enable/Disable coalesce mode to flatten striped files as stored on the server (i.e., eliminate holes caused by the traditional format).
| cluster.server-quorum-type | (null) | This feature is on the server-side i.e. in glusterd. Whenever the glusterd on a machine observes that the quorum is not met, it brings down the bricks to prevent data split-brains. When the network connections are brought back up and the quorum is restored the bricks in the volume are brought back up.
| cluster.server-quorum-ratio | (null) | Sets the quorum percentage for the trusted storage pool.
| cluster.quorum-reads | no | If quorum-reads is "true" only allow reads if quorum is met when quorum is enabled.
|----
| diagnostics.latency-measurement | off | If on stats related to the latency of each operation would be tracked inside GlusterFS data-structures. 
| diagnostics.dump-fd-stats | off | If on stats related to file-operations would be tracked inside GlusterFS data-structures.
| diagnostics.brick-log-level | INFO | Changes the log-level of the bricks
| diagnostics.client-log-level | INFO | Changes the log-level of the clients
| diagnostics.brick-sys-log-level | CRITICAL | Gluster's syslog log-level
| diagnostics.client-sys-log-level | CRITICAL | Gluster's syslog log-level
| diagnostics.brick-logger | null | null
| diagnostics.client-logger | null | null
| diagnostics.brick-log-format | null | null
| diagnostics.client-log-format | null | null
| diagnostics.brick-log-buf-size | 5 | null
| diagnostics.client-log-buf-size | 5 | null
| diagnostics.brick-log-flush-timeout | 20 | null
| diagnostics.client-log-flush-timeout | 20 | null
|----
| disperse.background-heals | 8 | This option can be used to control number of parallel heals
| disperse.heal-wait-qlength | 128 | This option can be used to control number of heals that can wait
| disperse.read-policy | round-robin | inode-read fops happen only on 'k' number of bricks in n=k+m disperse subvolume. 'round-robin' selects the read subvolume using round-robin algo. 'gfid-hash' selects read subvolume based on hash of the gfid of that file/directory.
|----
| dht.force-readdirp | on | This option if set to ON, forces the use of readdirp, and hence also displays the stats of the files.
|----
| performance.cache-max-file-size | 0 | Maximum file size which would be cached by the io-cache translator.
| performance.cache-min-file-size | 0 | Minimum file size which would be cached by the io-cache translator.
| performance.cache-refresh-timeout | 1 | The cached data for a file will be retained till 'cache-refresh-timeout' seconds, after which data re-validation is performed.
| performance.cache-priority | | Assigns priority to filenames with specific patterns so that when a page needs to be ejected out of the cache, the page of a file whose priority is the lowest will be ejected earlier
| performance.cache-size | 32MB | Size of the read cache.
| performance.io-thread-count | 16 | Number of threads in IO threads translator which perform concurrent IO operations
| performance.high-prio-threads | 16 | Max number of threads in IO threads translator which perform high priority IO operations at a given time
| performance.normal-prio-threads | 16 | Max number of threads in IO threads translator which perform normal priority IO operations at a given time 
| performance.low-prio-threads | 16 | Max number of threads in IO threads translator which perform low priority IO operations at a given time
| performance.least-prio-threads | 1 | Max number of threads in IO threads translator which perform least priority IO operations at a given time
| performance.enable-least-priority | on | Enable/Disable least priority
| performance.least-rate-limit | 0 | Max number of least priority operations to handle per-second
| performance.flush-behind | on | If this option is set ON, instructs write-behind translator to perform flush in background, by returning success (or any errors, if any of previous  writes were failed) to application even before flush FOP is sent to backend filesystem.
| performance.nfs.flush-behind | on | If this option is set ON, instructs write-behind translator to perform flush in background, by returning success (or any errors, if any of previous  writes were failed) to application even before flush FOP is sent to backend filesystem. 
| performance.write-behind-window-size | 1MB | Size of the write-behind buffer for a single file (inode).
| performance.nfs.write-behind-window-size | 1MB | Size of the write-behind buffer for a single file (inode).
| performance.strict-o-direct | off | This option when set to off, ignores the O_DIRECT flag.
| performance.nfs.strict-o-direct | off | This option when set to off, ignores the O_DIRECT flag. 
| performance.strict-write-ordering | off | Do not let later writes overtake earlier writes even if they do not overlap
| performance.nfs.strict-write-ordering | off | Do not let later writes overtake earlier writes even if they do not overlap
| performance.lazy-open | yes | Perform open in the backend only when a necessary FOP arrives (e.g writev on the FD, unlink of the file). When option is disabled, perform backend open right after unwinding open().
| performance.read-after-open | no | read is sent only after actual open happens and real fd is obtained, instead of doing on anonymous fd (similar to write)
| performance.read-ahead-page-count | 4 | Number of pages that will be pre-fetched
| performance.md-cache-timeout | 1 | Time period after which cache has to be refreshed
| performance.write-behind | on | enable/disable write-behind translator in the volume.
| performance.read-ahead | on | enable/disable read-ahead translator in the volume.
| performance.readdir-ahead | off | enable/disable readdir-ahead translator in the volume.
| performance.io-cache | on | enable/disable io-cache translator in the volume.
| performance.quick-read | on | enable/disable quick-read translator in the volume.
| performance.open-behind | on | enable/disable open-behind translator in the volume.
| performance.stat-prefetch | on | enable/disable meta-data caching translator in the volume.
| performance.client-io-threads | off | enable/disable io-threads translator in the client graph of volume.
| performance.nfs.write-behind | on | enable/disable write-behind translator in the volume
| performance.force-readdirp | true | Convert all readdir requests to readdirplus to collect stat info on each entry.
| features.encryption | off | enable/disable client-side encryption for the volume.
| encryption.master-key | null | Pathname of regular file which contains master volume key
| encryption.data-key-size | 256 | Data key size (bits)
| encryption.block-size | 4096 | Atom size (bits)
| nfs.enable-ino32 | no | For nfs clients or apps that do not support 64-bit inode numbers, use this option to make NFS return 32-bit inode numbers instead. Disabled by default, so NFS returns 64-bit inode numbers.
| nfs.mem-factor | 15 | Use this option to make NFS be faster on systems by using more memory. This option specifies a multiple that determines the total amount of memory used. Default value is 15. Increase to use more memory in order to improve performance for certain use cases.Please consult gluster-users list before using this option.
| nfs.export-dirs | on | By default, all subvolumes of nfs are exported as individual exports. There are cases where a subdirectory or subdirectories in the volume need to be exported separately. Enabling this option allows any directory on a volumes to be exported separately.Directory exports are enabled by default.
| nfs.export-volumes | on | Enable or disable exporting whole volumes, instead if used in conjunction with nfs3.export-dir, can allow setting up only subdirectories as exports. On by default.
| nfs.addr-namelookup | off | Users have the option of turning on name lookup for incoming client connections using this option. Use this option to turn on name lookups during address-based authentication. Turning this on will enable you to use hostnames in nfs.rpc-auth-\* filters. In some setups, the name server can take too long to reply to DNS queries resulting in timeouts of mount requests. By default, name lookup is off
| nfs.dynamic-volumes | off | Internal option set to tell gnfs to use a different scheme for encoding file handles when DVM is being used.
| nfs.register-with-portmap | on | For systems that need to run multiple nfs servers, only one registration is possible with portmap service. Use this option to turn off portmap registration for Gluster NFS. On by default
| nfs.outstanding-rpc-limit | 16 | Parameter to throttle the number of incoming RPC requests from a client. 0 means no limit (can potentially run out of memory)
| nfs.port | 2049 | Use this option on systems that need Gluster NFS to be associated with a non-default port number.
| nfs.rpc-auth-unix | on | Disable or enable the AUTH_UNIX authentication type for a particular exported volume overriding defaults and general setting for AUTH_UNIX scheme. Must always be enabled for better interoperability. However, can be disabled if needed. Enabled by default.
| nfs.rpc-auth-null | on | Disable or enable the AUTH_NULL authentication type for a particular exported volume overriding defaults and general setting for AUTH_NULL. Must always be enabled. This option is here only to avoid unrecognized option warnings.
| nfs.rpc-auth-allow | all | Allow a comma separated list of addresses and/or hostnames to connect to the server. By default, all connections are allowed. This allows users to define a rule for a specific exported volume.
| nfs.rpc-auth-reject | none | Reject a comma separated list of addresses and/or hostnames from connecting to the server. By default, all connections are allowed. This allows users to define a rule for a specific exported volume.
| nfs.ports-insecure | off | Allow client connections from unprivileged ports. By default only privileged ports are allowed. Use this option to enable or disable insecure ports for a specific subvolume and to override the global setting set by the previous option.
| nfs.transport-type | (null) | Specifies the nfs transport type. Valid transport types are 'tcp' and 'rdma'.
| nfs.trusted-sync | off | All writes and COMMIT requests are treated as async. This implies that no write requests are guaranteed to be on server disks when the write reply is received at the NFS client. Trusted sync includes  trusted-write behaviour. Off by default.
| nfs.trusted-write | off | On an UNSTABLE write from client, return STABLE flag to force client to not send a COMMIT request. In some environments, combined with a replicated GlusterFS setup, this option can improve write performance. This flag allows user to trust Gluster replication logic to sync data to the disks and recover when required. COMMIT requests if received will be handled in a default manner by fsyncing. STABLE writes are still handled in a sync manner. Off by default.
| nfs.volume-access | read-write | Type of access desired for this subvolume:  read-only, read-write(default)
| nfs.export-dir | | By default, all subvolumes of nfs are exported as individual exports. There are cases where a subdirectory or subdirectories in the volume need to be exported separately. This option can also be used in conjunction with nfs3.export-volumes option to restrict exports only to the subdirectories specified through this option. Must be an absolute path. Along with path allowed list of IPs/hostname can be associated with each subdirectory. If provided connection will allowed only from these IPs. By default connections from all IPs are allowed. Format: <dir>[(hostspec[\|hostspec\|...])][,...]. Where hostspec can be an IP address, hostname or an IP range in CIDR notation. e.g. /foo(192.168.1.0/24\|host1\|10.1.1.8),/host2. NOTE: Care must be taken while configuring this option as invalid entries and/or unreachable DNS servers can introduce unwanted delay in all the mount calls.
| nfs.disable | false | This option is used to start or stop the NFS server for individual volumes.
| nfs.nlm | on | This option, if set to 'off', disables NLM server by not registering the service with the portmapper. Set it to 'on' to re-enable it. Default value: 'on'
| nfs.acl | on | This option is used to control ACL support for NFS.
| nfs.mount-udp | off | set the option to 'on' to enable mountd on UDP. Required for some Solaris and AIX NFS clients. The need for enabling this option often depends on the usage of NLM.
| nfs.mount-rmtab | /var/lib/glusterd/nfs/rmtab | Set the location of the cache file that is used to list all the NFS-clients that have connected through the MOUNT protocol. If this is on shared storage, all GlusterFS servers will update and output (with 'showmount') the same list. Set to "/-" to disable. 
| nfs.drc | off | Enable Duplicate Request Cache in gNFS server to improve correctness of non-idempotent operations like write, delete, link, et al
| nfs.drc-size| 0x20000 | Sets the number of non-idempotent requests to cache in drc
| nfs.read-size | (1 * 1048576ULL) | Size in which the client should issue read requests to the Gluster NFSv3 server. Must be a multiple of 4KB (4096). Min and Max supported values are 4KB (4096) and 1MB (1048576) respectively. If the specified value is within the supported range but not a multiple of 4096, it is rounded up to the nearest multiple of 4096.
| nfs.write-size | (1 * 1048576ULL) | Size in which the client should issue write requests to the Gluster NFSv3 server. Must be a multiple of 1KB (1024). Min and Max supported values are 4KB (4096) and 1MB(1048576) respectively. If the specified value is within the supported range but not a multiple of 4096, it is rounded up to the nearest multiple of 4096.
| nfs.readdir-size | (1 * 1048576ULL) | Size in which the client should issue directory reading requests to the Gluster NFSv3 server. Must be a multiple of 1KB (1024). Min and Max supported values are 4KB (4096) and 1MB (1048576) respectively.If the specified value is within the supported range but not a multiple of 4096, it is rounded up to the nearest multiple of 4096.
| nfs.exports-auth-enable | (null) | Set the option to 'on' to enable exports/netgroup authentication in the NFS server and mount daemon.
| nfs.auth-refresh-interval-sec | (null) | Frequency in seconds that the daemon should check for changes in the exports/netgroups file.
| nfs.auth-cache-ttl-sec | (null) | Sets the TTL of an entry in the auth cache. Value is in seconds.
| ganesha.enable | off | export volume via NFS-Ganesha
|----
| network.frame-timeout | 1800 | Time frame after which the (file) operation would be declared as dead, if the server does not respond for a particular (file) operation.
| network.ping-timeout | 42 | Time duration for which the client waits to check if the server is responsive.
| network.tcp-window-size | null | Specifies the window size for tcp socket.
| network.remote-dio | disable | If enabled, in open() and creat() calls, O_DIRECT flag will be filtered at the client protocol level so server will still continue to cache the file. This works similar to NFS's behavior of O_DIRECT
| network.inode-lru-limit | 16384 | Specifies the maximum megabytes of memory to be used in the inode cache.
| network.compression | off | enable/disable network compression translator
| network.compression.window-size | -15 | Size of the zlib history buffer.
| network.compression.mem-level | 8 | Memory allocated for internal compression state. 1 uses minimum memory but is slow and reduces compression ratio; memLevel=9 uses maximum memory for optimal speed. The default value is 8.
| network.compression.min-size | 0 | Data is compressed only when its size exceeds this.
| network.compression.compression-level | -1 | Compression levels 0 \: no compression, 1 \: best speed, 9 \: best compression, -1 \: default compression 
|----
| features.lock-heal | off | When the connection to client is lost, server cleans up all the locks held by the client. After the connection is restored, the client reacquires (heals) the fcntl locks released by the server.
| features.grace-timeout | 10 | Specifies the duration for the lock state to be maintained on the client after a network disconnection. Range 10-1800 seconds.
| features.file-snapshot | off | enable/disable file-snapshot feature in the volume.
| features.uss | off | enable/disable User Serviceable Snapshots on the volume.
| features.snapshot-directory | .snaps | Entry point directory for entering snapshot world
| features.show-snapshot-directory | off | show entry point in readdir output of snapdir-entry-path which is set by samba
| features.quota-deem-statfs | off | If set to on, it takes quota limits intoconsideration while estimating fs size. (df command) (Default is off).
| features.read-only | off | When "on", makes a volume read-only. It is turned "off" by default.
| features.worm | off | When "on", makes a volume get write once read many  feature. It is turned "off" by default.
| features.barrier-timeout | 120 | After 'timeout' seconds since the time 'barrier' option was set to "on", acknowledgements to file operations are no longer blocked and previously blocked acknowledgements are sent to the application
| features.trash | off | Enable/disable trash translator
| features.trash-dir | .trashcan | Directory for trash files
| features.trash-eliminate-path | (null) | Eliminate paths to be excluded from trashing
| features.trash-max-filesize | 5MB | Maximum size of file that can be moved to trash
| features.trash-internal-op | off | Enable/disable trash translator for internal operations
| features.ctr-enabled | off | Enable CTR xlator
| features.record-counters | off | Its a Change Time Recorder Xlator option to enable recording write and read heat counters. The default is disabled. If enabled, "cluster.write-freq-threshold" and "cluster.read-freq-threshold" defined the number of writes (or reads) to a given file are needed before triggering migration.
| features.ctr-sql-db-cachesize | 1000 | Defines the cache size of the sqlite database of changetimerecorder xlator.The input to this option is in pages.Each page is 4096 bytes. Default value is 1000 pages i.e ~ 4 MB. The max value is 262144 pages i.e 1 GB and the min value is 1000 pages i.e ~ 4 MB. 
| features.ctr-sql-db-wal-autocheckpoint | 1000 | Defines the autocheckpoint of the sqlite database of  changetimerecorder. The input to this option is in pages. Each page is 4096 bytes. Default value is 1000 pages i.e ~ 4 MB.The max value is 262144 pages i.e 1 GB and the min value is 1000 pages i.e ~4 MB.
| features.shard-block-size | 4MB | The size unit used to break a file into multiple chunks
| features.cache-invalidation | off | When "on", sends cache-invalidation notifications.
| features.cache-invalidation-timeout | 60 | After 'timeout' seconds since the time client accessed any file, cache-invalidation notifications are no longer sent to that client.
|----
| client.event-threads | 2 | Specifies the number of event threads to execute in parallel. Larger values would help process responses faster, depending on available processing power. Range 1-32 threads.
| auth.allow | null | Allow a comma separated list of addresses and/or hostnames to connect to the server. Option auth.reject overrides this option. By default, all connections are allowed.
| auth.reject | null | Reject a comma separated list of addresses and/or hostnames to connect to the server. This option overrides the auth.allow option. By default, all connections are allowed.
|----
| server.root-squash | off | Map requests from uid/gid 0 to the anonymous uid/gid. Note that this does not apply to any other uids or gids that might be equally sensitive, such as user bin or group staff.
| server.anonuid | 65534 | value of the uid used for the anonymous user/nfsnobody when root-squash is enabled.
| server.anongid | 65534 | value of the gid used for the anonymous user/nfsnobody when root-squash is enabled.
| server.statedump-path | /var/run/gluster | Specifies directory in which gluster should save its statedumps.
| server.outstanding-rpc-limit | 64 | Parameter to throttle the number of incoming RPC requests from a client. 0 means no limit (can potentially run out of memory)
| server.manage-gids | off | Resolve groups on the server-side.
| server.dynamic-auth | on | When 'on' perform dynamic authentication of volume options in order to allow/terminate client transport connection immediately in response to \*.allow \| \*.reject volume set options.
| server.gid-timeout | 300 | Timeout in seconds for the cached groups to expire.
| server.event-threads | 2 | Specifies the number of event threads to execute in parallel. Larger values would help process responses faster, depending on available processing power. Range 1-32 threads.
|----
| ssl.own-cert | null | SSL certificate. Ignored if SSL is not enabled.
| ssl.private-key | null | SSL private key. Ignored if SSL is not enabled.
| ssl.ca-list | null | SSL CA list. Ignored if SSL is not enabled.
| ssl.crl-path | null | Path to directory containing CRL. Ignored if SSL is not enabled.
| ssl.certificate-depth | null | Maximum certificate-chain depth.  If zero, the peer's certificate itself must be in the local certificate list.  Otherwise, there may be up to N signing certificates between the peer's and the local list.  Ignored if SSL is not enabled.
| ssl.cipher-list | null | Allowed SSL ciphers. Ignored if SSL is not enabled.
| ssl.dh-param | null | DH parameters file. Ignored if SSL is not enabled.
| ssl.ec-curve | null | ECDH curve name. Ignored if SSL is not enabled.
|----
| storage.linux-aio | off | Support for native Linux AIO
| storage.batch-fsync-mode | reverse-fsync | Possible values: syncfs: Perform one syncfs() on behalf oa batchof fsyncs. syncfs-single-fsync: Perform one syncfs() on behalf of a batch of fsyncs and one fsync() per batch. syncfs-reverse-fsync: Preform one syncfs() on behalf of a batch of fsyncs and fsync() each file in the batch in reverse order. reverse-fsync: Perform fsync\(\) of each file in the batch in reverse order.
| storage.batch-fsync-delay-usec | 0 | Num of usecs to wait for aggregating fsync requests
| storage.owner-uid | -1 | Support for setting uid of brick's owner
| storage.owner-gid | -1 | Support for setting gid of brick's owner
| storage.node-uuid-pathinfo | off | return glusterd's node-uuid in pathinfo xattr string instead of hostname
| storage.health-check-interval | 30 | Interval in seconds for a filesystem health check, set to 0 to disable
| storage.build-pgfid | off | Enable placeholders for gfid to path conversion
| storage.bd-aio | off | Support for native Linux AIO


## oVirt针对GlusterFS的优化

| 选项 | 配置 | 说明
|:-|:-:|:-
|cluster.server-quorum-type|server|
|cluster.quorum-type|auto|
|network.remote-dio|enable|
|cluster.eager-lock|enable|
|performance.stat-prefetch|off|
|performance.io-cache|off|
|performance.read-ahead|off|
|performance.quick-read|off|
|performance.readdir-ahead|on|

## 参考文档

* [oVirt 3.4, Glusterized](http://community.redhat.com/blog/2014/05/ovirt-3-4-glusterized/)

