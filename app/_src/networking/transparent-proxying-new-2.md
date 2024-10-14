{% tabs transparent-proxy-config-reference useUrlFragment=false %}
{% tab transparent-proxy-config-reference Simplified Reference %}

```yaml
# The username or UID of the user that will run kuma-dp
kumaDPUser: string
# The IP family mode used for configuring traffic redirection in the transparent proxy
ipFamilyMode: enum # default: "dualstack"
redirect:
  inbound:
    # Enables inbound traffic redirection
    enabled: bool # default: true
    # Port used for redirecting inbound traffic
    port: Port # default: 15006
    # List of ports to include in inbound traffic redirection
    includePorts: Ports
    # List of ports to exclude from inbound traffic redirection
    excludePorts: Ports
    # List of IP addresses to exclude from inbound traffic redirection for specific ports
    excludePortsForIPs: []string
    # List of UIDs to exclude from inbound traffic redirection for specific ports
    excludePortsForUIDs: []string
    # Inserts the redirection rule at the beginning of the chain instead of appending it
    insertRedirectInsteadOfAppend: bool
  outbound:
    # Enables outbound traffic redirection
    enabled: bool # default: true
    # Port used for redirecting outbound traffic
    port: Port # default: 15001
    # List of ports to include in outbound traffic redirection
    includePorts: []Port
    # List of ports to exclude from outbound traffic redirection
    excludePorts: []Port
    # List of IP addresses to exclude from outbound traffic redirection for specific ports
    excludePortsForIPs: []string
    # List of UIDs to exclude from outbound traffic redirection for specific ports
    excludePortsForUIDs: []string
    # Inserts the redirection rule at the beginning of the chain instead of appending it
    insertRedirectInsteadOfAppend: bool
  dns:
    # Enables DNS redirection in the transparent proxy
    enabled: bool
    # The port on which the DNS server listens
    port: Port # default: 15053
    # Redirect all DNS queries
    captureAll: bool
    # Disables conntrack zone splitting, which can prevent potential DNS issues
    skipConntrackZoneSplit: bool
    # Path to the system's resolv.conf file
    resolvConfigPath: string # default: "/etc/resolv.conf"
  vnet:
    # Specifies virtual networks using the format interfaceName:CIDR
    networks: []string
ebpf:
  # Enables eBPF support for handling traffic redirection in the transparent proxy
  enabled: bool
  instanceIP: string
  # The name of the environment variable containing the IP address of the instance (pod/vm) where transparent proxy will be installed
  instanceIPEnvVarName: string
  # The path of the BPF filesystem
  bpffsPath: string # default: "/run/kuma/bpf"
  # The path of cgroup2
  cgroupPath: string # default: "/sys/fs/cgroup"
  # Path where compiled eBPF programs and other necessary files for eBPF mode can be found
  programsSourcePath: string # default: "tmp/kuma-ebpf"
  # The network interface for TC eBPF programs to bind to
  tcAttachIface: string
retry:
  # The maximum number of retry attempts for operations
  maxRetries: uint # default: 4
  # The time duration to wait between retry attempts
  sleepBetweenRetries: Duration # default: "2s"
iptablesExecutables:
  # Custom path for the iptables executable (IPv4)
  iptables: string
  # Custom path for the iptables-save executable (IPv4)
  iptables-save: string
  # Custom path for the iptables-restore executable (IPv4)
  iptables-restore: string
  # Custom path for the ip6tables executable (IPv6)
  ip6tables: string
  # Custom path for the ip6tables-save executable (IPv6)
  ip6tables-save: string
  # Custom path for the ip6tables-restore executable (IPv6)
  ip6tables-restore: string
log:
  # Specifies the log level for iptables logging as defined by netfilter
  level: enum # default: 7
  # Enables logging of iptables rules for diagnostics and monitoring
  enabled: bool
comments:
  # Disables comments in the generated iptables rules
  disabled: bool
# Time in seconds to wait for acquiring the xtables lock before failing
wait: uint # default: 5
# Time interval between retries to acquire the xtables lock in seconds
waitInterval: uint
# Drops invalid packets to avoid connection resets in high-throughput scenarios
dropInvalidPackets: bool
# Enables firewalld support to store iptables rules
storeFirewalld: bool
cniMode: bool
dryRun: bool
# Enables verbose mode with longer argument/flag names and additional comments
verbose: bool
```

{% endtab %}
{% tab transparent-proxy-config-reference Environment Variables %}

```yaml
kumaDPUser: KUMA_TRANSPARENT_PROXY_KUMA_DP_USER
ipFamilyMode: KUMA_TRANSPARENT_PROXY_IP_FAMILY_MODE
redirect:
  inbound:
    enabled: KUMA_TRANSPARENT_PROXY_REDIRECT_INBOUND_ENABLED
    port: KUMA_TRANSPARENT_PROXY_REDIRECT_INBOUND_PORT
    includePorts: KUMA_TRANSPARENT_PROXY_REDIRECT_INBOUND_INCLUDE_PORTS
    excludePorts: KUMA_TRANSPARENT_PROXY_REDIRECT_INBOUND_EXCLUDE_PORTS
    excludePortsForIPs: KUMA_TRANSPARENT_PROXY_REDIRECT_INBOUND_EXCLUDE_PORTS_FOR_IPS
    excludePortsForUIDs: KUMA_TRANSPARENT_PROXY_REDIRECT_INBOUND_EXCLUDE_PORTS_FOR_UIDS
    insertRedirectInsteadOfAppend: KUMA_TRANSPARENT_PROXY_REDIRECT_INBOUND_INSERT_REDIRECT_INSTEAD_OF_APPEND
  outbound:
    enabled: KUMA_TRANSPARENT_PROXY_REDIRECT_OUTBOUND_ENABLED
    port: KUMA_TRANSPARENT_PROXY_REDIRECT_OUTBOUND_PORT
    includePorts: KUMA_TRANSPARENT_PROXY_REDIRECT_OUTBOUND_INCLUDE_PORTS
    excludePorts: KUMA_TRANSPARENT_PROXY_REDIRECT_OUTBOUND_EXCLUDE_PORTS
    excludePortsForIPs: KUMA_TRANSPARENT_PROXY_REDIRECT_OUTBOUND_EXCLUDE_PORTS_FOR_IPS
    excludePortsForUIDs: KUMA_TRANSPARENT_PROXY_REDIRECT_OUTBOUND_EXCLUDE_PORTS_FOR_UIDS
    insertRedirectInsteadOfAppend: KUMA_TRANSPARENT_PROXY_REDIRECT_OUTBOUND_INSERT_REDIRECT_INSTEAD_OF_APPEND
  dns:
    enabled: KUMA_TRANSPARENT_PROXY_REDIRECT_DNS_ENABLED
    port: KUMA_TRANSPARENT_PROXY_REDIRECT_DNS_PORT
    captureAll: KUMA_TRANSPARENT_PROXY_REDIRECT_DNS_CAPTURE_ALL
    skipConntrackZoneSplit: KUMA_TRANSPARENT_PROXY_REDIRECT_DNS_SKIP_CONNTRACK_ZONE_SPLIT
    resolvConfigPath: KUMA_TRANSPARENT_PROXY_REDIRECT_DNS_RESOLV_CONFIG_PATH
  vnet:
    networks: KUMA_TRANSPARENT_PROXY_REDIRECT_VNET_NETWORKS
ebpf:
  enabled: KUMA_TRANSPARENT_PROXY_EBPF_ENABLED
  instanceIP: KUMA_TRANSPARENT_PROXY_EBPF_INSTANCE_IP
  instanceIPEnvVarName: KUMA_TRANSPARENT_PROXY_EBPF_INSTANCE_IP_ENV_VAR_NAME
  bpffsPath: KUMA_TRANSPARENT_PROXY_EBPF_BPFFS_PATH
  cgroupPath: KUMA_TRANSPARENT_PROXY_EBPF_CGROUP_PATH
  programsSourcePath: KUMA_TRANSPARENT_PROXY_EBPF_PROGRAMS_SOURCE_PATH
  tcAttachIface: KUMA_TRANSPARENT_PROXY_EBPF_TC_ATTACH_IFACE
retry:
  maxRetries: KUMA_TRANSPARENT_PROXY_RETRY_MAX_RETRIES
  sleepBetweenRetries: KUMA_TRANSPARENT_PROXY_RETRY_SLEEP_BETWEEN_RETRIES
iptablesExecutables: KUMA_TRANSPARENT_PROXY_IPTABLES_EXECUTABLES
log:
  enabled: KUMA_TRANSPARENT_PROXY_LOG_ENABLED
  level: KUMA_TRANSPARENT_PROXY_LOG_LEVEL
comments:
  disabled: KUMA_TRANSPARENT_PROXY_COMMENTS_DISABLED
wait: KUMA_TRANSPARENT_PROXY_WAIT
waitInterval: KUMA_TRANSPARENT_PROXY_WAIT_INTERVAL
dropInvalidPackets: KUMA_TRANSPARENT_PROXY_DROP_INVALID_PACKETS
storeFirewalld: KUMA_TRANSPARENT_PROXY_STORE_FIREWALLD
cniMode: KUMA_TRANSPARENT_PROXY_CNI_MODE
dryRun: KUMA_TRANSPARENT_PROXY_DRY_RUN
verbose: KUMA_TRANSPARENT_PROXY_VERBOSE
```

{% endtab %}
{% tab transparent-proxy-config-reference CLI Flags %}

```yaml
kumaDPUser: --kuma-dp-user
ipFamilyMode: --ip-family-mode
redirect:
  dns:
    enabled: --redirect-dns
    port: --redirect-dns-port
    captureAll: --redirect-all-dns-traffic
    skipConntrackZoneSplit: --skip-dns-conntrack-zone-split
    resolvConfigPath: # no flag
  inbound:
    enabled: --redirect-inbound
    port: --redirect-inbound-port
    includePorts: # no flag
    excludePorts: --exclude-inbound-ports
    excludePortsForIPs: --exclude-inbound-ips
    excludePortsForUIDs: # no flag
    insertRedirectInsteadOfAppend: --redirect-inbound-insert-instead-of-append
  outbound:
    enabled: # no flag
    port: --redirect-outbound-port
    includePorts: # no flag
    excludePorts: --exclude-outbound-ports
    excludePortsForIPs: --exclude-outbound-ips
    excludePortsForUIDs: --exclude-outbound-ports-for-uids
    insertRedirectInsteadOfAppend: --redirect-outbound-insert-instead-of-append
  vnet:
    networks: --vnet
ebpf:
  enabled: --ebpf-enabled
  instanceIP: --ebpf-instance-ip
  instanceIPEnvVarName: # no flag
  bpffsPath: --ebpf-bpffs-path
  cgroupPath: --ebpf-cgroup-path
  programsSourcePath: --ebpf-programs-source-path
  tcAttachIface: --ebpf-tc-attach-iface
retry:
  maxRetries: --max-retries
  sleepBetweenRetries: --sleep-between-retries
iptablesExecutables: --iptables-executables
log:
  enabled: --iptables-logs
  level: # no flag
comments:
  disabled: --disable-comments
wait: --wait
waitInterval: --wait-interval
dropInvalidPackets: --drop-invalid-packets
storeFirewalld: --store-firewalld
cniMode: # no flag
dryRun: --dry-run
verbose: --verbose
```

{% endtab %}
{% tab transparent-proxy-config-reference Default Values %}

```yaml
ipFamilyMode: "dualstack" 
redirect:
  inbound:
    enabled: true
    port: 15006
  outbound:
    enabled: true
    port: 15001
  dns:
    port: 15053
    resolvConfigPath: "/etc/resolv.conf"
ebpf:
  bpffsPath: "/run/kuma/bpf"
  cgroupPath: "/sys/fs/cgroup"
  programsSourcePath: "/tmp/kuma-ebpf"
retry:
  maxRetries: 4
  sleepBetweenRetries: "2s"
log:
  level: 7
wait: 5
```

{% endtab %}
{% endtabs %}


      **Customization examples**

      - Regular file using `--config-file` flag

        ```yaml
        # config.yaml
        redirect:
          inbound:
            excludePorts: [22, 80, 443]
        ```

        ```sh
        kumactl install transparent-proxy --config-file config.yaml
        ```

      - **STDIN** using `--config-file` flag set to `-`

        ```sh
        echo "{redirect: {inbound: {excludePorts: [22, 80, 443]}}}" | kumactl install transparent-proxy --config-file -
        ```

      - Directly using `--config` flag

        ```sh
        kumactl install transparent-proxy --config '{"redirect":{"inbound":{"excludePorts":[22,80,443]}}}'
        ```

      - Environmental variable

        ```sh
        KUMA_TRANSPARENT_PROXY_REDIRECT_INBOUND_EXCLUDE_PORTS="22,80,443" kumactl install transparent-proxy
        ```

      - Specific flag

        ```sh
        kumactl install transparent-proxy --exclude-inbound-ports "22,80,443"
        ```


      **Customization examples**

      - Regular file using `--config-file` flag

        ```yaml
        # config.yaml
        redirect:
          inbound:
            excludePortsForIPs:
            - 10.0.0.1
            - 172.10.0.0/24
            - fe80::/10
        ```
  
        ```sh
        kumactl install transparent-proxy --config-file config.yaml
        ```

      - **STDIN** using `--config-file` flag set to `-`
  
        ```sh
        echo "{redirect: {inbound: {excludePortsForIPs: [10.0.0.1, 172.10.0.0/24, 'fe80::/10']}}}" | kumactl install transparent-proxy --config-file -
        ```

      - Directly using `--config` flag

        ```sh
        kumactl install transparent-proxy --config '{"redirect":{"inbound":{"excludePortsForIPs":["10.0.0.1","172.10.0.0/24", "fe80::/10"]}}}'
        ```

      - Environmental variable

        ```sh
        KUMA_TRANSPARENT_PROXY_REDIRECT_INBOUND_EXCLUDE_PORTS_FOR_IPS="10.0.0.1,172.10.0.0/24,fe80::/10" kumactl install transparent-proxy
        ```

      - Specific flag

        ```sh
        kumactl install transparent-proxy --exclude-inbound-ips "10.0.0.1,172.10.0.0/24,fe80::/10"
        ```
