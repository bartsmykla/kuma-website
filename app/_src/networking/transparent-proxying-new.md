| Configuration                         | Type         | Environment Variable                                            | CLI Flag                          |
|---------------------------------------|--------------|-----------------------------------------------------------------|-----------------------------------|
| kumaDPUser                            | Owner        | KUMA_TRANSPARENT_PROXY_KUMA_DP_USER                             | --kuma-dp-user                    |
| redirect.inbound.enabled              | bool         | KUMA_TRANSPARENT_PROXY_REDIRECT_INBOUND_ENABLED                 | --redirect-inbound                |
| redirect.inbound.port                 | Port         | KUMA_TRANSPARENT_PROXY_REDIRECT_INBOUND_PORT                    | --redirect-inbound-port           |
| redirect.inbound.includePorts         | Ports        | KUMA_TRANSPARENT_PROXY_REDIRECT_INBOUND_INCLUDE_PORTS           |                                   |
| redirect.inbound.excludePorts         | Ports        | KUMA_TRANSPARENT_PROXY_REDIRECT_INBOUND_EXCLUDE_PORTS           | --exclude-inbound-ports           |
| redirect.inbound.excludePortsForUIDs  | []string     | KUMA_TRANSPARENT_PROXY_REDIRECT_INBOUND_EXCLUDE_PORTS_FOR_UIDS  | --exclude-inbound-ports-for-uids  |
| redirect.inbound.excludePortsForIPs   | []string     | KUMA_TRANSPARENT_PROXY_REDIRECT_INBOUND_EXCLUDE_PORTS_FOR_IPS   | --exclude-inbound-ips             |
| redirect.outbound.enabled             | bool         | KUMA_TRANSPARENT_PROXY_REDIRECT_OUTBOUND_ENABLED                |                                   |
| redirect.outbound.port                | Port         | KUMA_TRANSPARENT_PROXY_REDIRECT_OUTBOUND_PORT                   | --redirect-outbound-port          |
| redirect.outbound.includePorts        | Ports        | KUMA_TRANSPARENT_PROXY_REDIRECT_OUTBOUND_INCLUDE_PORTS          |                                   |
| redirect.outbound.excludePorts        | Ports        | KUMA_TRANSPARENT_PROXY_REDIRECT_OUTBOUND_EXCLUDE_PORTS          | --exclude-outbound-ports          |
| redirect.outbound.excludePortsForUIDs | []string     | KUMA_TRANSPARENT_PROXY_REDIRECT_OUTBOUND_EXCLUDE_PORTS_FOR_UIDS | --exclude-outbound-ports-for-uids |
| redirect.outbound.excludePortsForIPs  | []string     | KUMA_TRANSPARENT_PROXY_REDIRECT_OUTBOUND_EXCLUDE_PORTS_FOR_IPS  | --exclude-outbound-ips            |
| redirect.dns.enabled                  | bool         | KUMA_TRANSPARENT_PROXY_REDIRECT_DNS_ENABLED                     | --redirect-dns                    |
| redirect.dns.port                     | Port         | KUMA_TRANSPARENT_PROXY_REDIRECT_DNS_PORT                        | --redirect-dns-port               |
| redirect.dns.captureAll               | bool         | KUMA_TRANSPARENT_PROXY_REDIRECT_DNS_CAPTURE_ALL                 | --redirect-all-dns-traffic        |
| redirect.dns.skipConntrackZoneSplit   | bool         | KUMA_TRANSPARENT_PROXY_REDIRECT_DNS_SKIP_CONNTRACK_ZONE_SPLIT   | --skip-dns-conntrack-zone-split   |
| redirect.dns.resolvConfigPath         | string       | KUMA_TRANSPARENT_PROXY_REDIRECT_DNS_RESOLV_CONFIG_PATH          |                                   |
| redirect.vnet.networks                | []string     | KUMA_TRANSPARENT_PROXY_REDIRECT_VNET_NETWORKS                   | --vnet                            |
| ebpf.enabled                          | bool         | KUMA_TRANSPARENT_PROXY_EBPF_ENABLED                             | --ebpf-enabled                    |
| ebpf.instanceIP                       | string       | KUMA_TRANSPARENT_PROXY_EBPF_INSTANCE_IP                         | --ebpf-instance-ip                |
| ebpf.bpffsPath                        | string       | KUMA_TRANSPARENT_PROXY_EBPF_BPFFS_PATH                          | --ebpf-bpffs-path                 |
| ebpf.cgroupPath                       | string       | KUMA_TRANSPARENT_PROXY_EBPF_CGROUP_PATH                         | --ebpf-cgroup-path                |
| ebpf.programsSourcePath               | string       | KUMA_TRANSPARENT_PROXY_EBPF_PROGRAMS_SOURCE_PATH                | --ebpf-programs-source-path       |
| ebpf.tcAttachIface                    | string       | KUMA_TRANSPARENT_PROXY_EBPF_TC_ATTACH_IFACE                     | --ebpf-tc-attach-iface            |
| retry.maxRetries                      | int          | KUMA_TRANSPARENT_PROXY_RETRY_MAX_RETRIES                        | --max-retries                     |
| retry.sleepBetweenRetries             | duration     | KUMA_TRANSPARENT_PROXY_RETRY_SLEEP_BETWEEN_RETRIES              | --sleep-between-retries           |
| log.enabled                           | bool         | KUMA_TRANSPARENT_PROXY_LOG_ENABLED                              | --iptables-logs                   |
| log.level                             | uint16       | KUMA_TRANSPARENT_PROXY_LOG_LEVEL                                |                                   |
| comments.disabled                     | bool         | KUMA_TRANSPARENT_PROXY_COMMENTS_DISABLED                        | --disable-comments                |
| wait                                  | uint         | KUMA_TRANSPARENT_PROXY_WAIT                                     |                                   |
| waitInterval                          | uint         | KUMA_TRANSPARENT_PROXY_WAIT_INTERVAL                            | --wait-interval                   |
| dropInvalidPackets                    | bool         | KUMA_TRANSPARENT_PROXY_DROP_INVALID_PACKETS                     | --drop-invalid-packets            |
| storeFirewalld                        | bool         | KUMA_TRANSPARENT_PROXY_STORE_FIREWALLD                          | --store-firewalld                 |
| ipFamilyMode                          | IPFamilyMode | KUMA_TRANSPARENT_PROXY_IP_FAMILY_MODE                           | --ip-family-mode                  |
| cniMode                               | bool         | KUMA_TRANSPARENT_PROXY_CNI_MODE                                 |                                   |
| dryRun                                | bool         | KUMA_TRANSPARENT_PROXY_DRY_RUN                                  | --dry-run                         |
| verbose                               | bool         | KUMA_TRANSPARENT_PROXY_VERBOSE                                  | --verbose                         |
