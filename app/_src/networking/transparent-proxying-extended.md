---
title: Transparent Proxying
---

## What is the Transparent Proxy?

## Transparent Proxy Configuration

During transparent proxy installation {{site.mesh_product_name}} under the hood is using a common structure, which can be modified in multiple different ways. By modifying it you are able to for example exclude some ports or IPs from transparent proxy redirection, configure if it should handle both IPv4 and IPv6 or just IPv4 traffic and more.

### Kubernetes

In Kubernetes environment transparent proxy is required to be enabled so it's not possible to disable it

#### (experimental) Configuration from ConfigMap

{% warning %}
This feature is experimental. Use it with caution.{% if site.mesh_product_name == "Kuma" %} If you experience any unexpected behavior or issues, please [**contact us**](/community) and [**submit an issue on GitHub**](https://github.com/kumahq/kuma/issues/new/choose). Your feedback is important in helping us improve this feature.{% endif %}
{% endwarning %}

Until {{site.mesh_product_name}} 2.9, customizing the transparent proxy configuration in Kubernetes was often complicated and difficult to manage due to the following reasons:

- Some settings could only be changed in the [control plane runtime configuration](#kubernetes-control-plane-runtime-configuration).
- Other settings had to be adjusted using annotations.
- Certain settings could be configured in both places, adding to the confusion.

This approach had several limitations. Settings applied in the [control plane runtime configuration](#kubernetes-control-plane-runtime-configuration) were enforced across all Kubernetes workloads with injected data planes, with no way to selectively apply them to specific workloads. On the other hand, settings configurable via annotations had to be applied individually to each workload, making it difficult to configure multiple workloads with the same values at once.

##### ConfigMap Auto-Creation and Configuration

When this feature is enabled, Kuma will automatically create a ConfigMap in the <code>{{site.mesh_namespace}}</code> namespace with the name specified in the <code>{{site.set_flag_values_prefix}}transparentProxy.configMap.name</code> setting. The contents of the ConfigMap will be based on the configuration defined in <code>{{site.set_flag_values_prefix}}transparentProxy.configMap.config</code>, which is a YAML representation of the transparent proxy configuration.

{% warning %}
If <code>{{site.set_flag_values_prefix}}transparentProxy.configMap.config</code> is empty, it is treated as if <code>{{site.set_flag_values_prefix}}transparentProxy.configMap.enabled: false</code>, and the feature will be disabled.
{% endwarning %}

{% tip %}
For a detailed structure of this configuration, refer to the documentation at [Kuma Control Plane Helm Values](/docs/{{ page.version }}/reference/kuma-cp/#helm-valuesyaml) under the <code>{{site.set_flag_values_prefix}}transparentProxy.configMap.config</code> path or see the [Configuration Reference](#configuration-reference) section below.
{% endtip %}

**Examples:**

{% tabs transparent-proxy-config-configmap-auto-creation-examples useUrlFragment=false %}
{% tab transparent-proxy-config-configmap-auto-creation-examples Helm %}

```sh
helm install \
  --create-namespace \
  --namespace {{site.mesh_namespace}} \
  --set "{{site.set_flag_values_prefix}}transparentProxy.configMap.config.redirect.outbound.excludePortsForIPs=169\.254\.169\.254\,169\.254\.169\.252" \
  --set "{{site.set_flag_values_prefix}}transparentProxy.configMap.config.redirect.verbose=false" \
  {{ site.mesh_helm_install_name }} {{ site.mesh_helm_repo }}
```

{% endtab %}
{% tab transparent-proxy-config-configmap-auto-creation-examples kumactl %}

```sh
kumactl install control-plane \
  --set "{{site.set_flag_values_prefix}}transparentProxy.configMap.config.redirect.outbound.excludePortsForIPs=169\.254\.169\.254\,169\.254\.169\.252" \
  --set "{{site.set_flag_values_prefix}}transparentProxy.configMap.config.redirect.verbose=false"
```

{% endtab %}
{% endtabs %}

##### ConfigMap Lookup Strategy

{{site.mesh_product_name}} follows a specific order when searching for the ConfigMap containing the transparent proxy configuration:

- If the workload is annotated with <code>traffic.kuma.io/transparent-proxy-configmap-name</code>, {{site.mesh_product_name}} first looks for the ConfigMap with the specified name in the workload’s namespace. If not found, it will then check the <code>{{site.mesh_namespace}}</code> namespace for the same ConfigMap.

- If the workload is not annotated, or the ConfigMap specified in the annotation is not found, {{site.mesh_product_name}} will search for the ConfigMap named in the <code>{{site.set_flag_values_prefix}}transparentProxy.configMap.name</code> setting. It first looks in the workload's namespace, and if not found, it checks the <code>{{site.mesh_namespace}}</code> namespace.

{% warning %}
The ConfigMap in the <code>{{site.mesh_namespace}}</code> namespace is required for proper operation, so it must be present even if a custom ConfigMap is used in individual workload namespaces.
{% endwarning %}

##### Enabling the Feature

To enable this feature, set the <code>{{site.set_flag_values_prefix}}transparentProxy.configMap.enabled</code> option during installation:

{% warning %}
If <code>{{site.set_flag_values_prefix}}transparentProxy.configMap.config</code> is empty, it is treated as if <code>{{site.set_flag_values_prefix}}transparentProxy.configMap.enabled: false</code>, and the feature will be disabled.
{% endwarning %}

{% tabs transparent-proxy-config-configmap-enabling useUrlFragment=false %}
{% tab transparent-proxy-config-configmap-enabling Helm %}

```sh
helm install \
  --create-namespace \
  --namespace {{site.mesh_namespace}} \
  --set "{{site.set_flag_values_prefix}}transparentProxy.configMap.enabled=true" \
  {{ site.mesh_helm_install_name }} {{ site.mesh_helm_repo }}
```

{% endtab %}
{% tab transparent-proxy-config-configmap-enabling kumactl %}

```sh
kumactl install control-plane \
  --set "{{site.set_flag_values_prefix}}transparentProxy.configMap.enabled=true" \
  | kubectl apply -f-
```

{% endtab %}
{% endtabs %}

##### Custom ConfigMap Name

The name of the ConfigMap containing the transparent proxy configuration is specified by the <code>{{site.set_flag_values_prefix}}transparentProxy.configMap.name</code> setting. By default, it is set to <code>kuma-transparent-proxy-config</code>.

{% tabs transparent-proxy-config-configmap-custom-name useUrlFragment=false %}
{% tab transparent-proxy-config-configmap-custom-name All Workloads %}

To specify a different ConfigMap name for all workloads, use the <code>{{site.set_flag_values_prefix}}transparentProxy.configMap.name</code> setting during installation:

{% tabs transparent-proxy-config-configmap-custom-name-globally useUrlFragment=false %}
{% tab transparent-proxy-config-configmap-custom-name-globally Helm %}

```sh
helm install \
  --create-namespace \
  --namespace {{site.mesh_namespace}} \
  --set "{{site.set_flag_values_prefix}}transparentProxy.configMap.name=custom-name" \
  {{ site.mesh_helm_install_name }} {{ site.mesh_helm_repo }}
```

{% endtab %}
{% tab transparent-proxy-config-configmap-custom-name-globally kumactl %}

```sh
kumactl install control-plane \
  --set "{{site.set_flag_values_prefix}}transparentProxy.configMap.name=custom-name" \
  | kubectl apply -f-
```

{% endtab %}
{% endtabs %}
{% endtab %}
{% tab transparent-proxy-config-configmap-custom-name Specific Workloads %}

If you want specific workloads to use a different ConfigMap, you can apply the <nobr><code>traffic.kuma.io/transparent-proxy-configmap-name</code></nobr> annotation to those workloads:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: workload
  annotations:
    traffic.kuma.io/transparent-proxy-configmap-name: "custom-name"
...
```

{% endtab %}
{% endtabs %}

### How to Customize the Configuration

Depending on your environment (Universal or Kubernetes), there are multiple ways to customize the Transparent Proxy configuration. These methods can be used individually or combined as needed.

{% warning %}
It is recommended to modify the configuration using only one method at a time whenever possible. Mixing multiple methods can lead to confusion and make troubleshooting difficult, as it may become unclear where specific configuration values are originating. If combining methods adds significant value to your use case, ensure you are familiar with the [**order of precedence**](#order-of-precedence) to understand how the final configuration is determined.
{% endwarning %}

#### (Universal) YAML or JSON Configuration

You can provide the configuration in <code>YAML</code> or <code>JSON</code> format using the <code>--config</code> or <code>--config-file</code> flags. For example:

Using the <code>--config-file</code> flag to specify a file with configuration:

{% tabs transparent-proxy-config-yaml-or-json-config-file useUrlFragment=false %}
{% tab transparent-proxy-config-yaml-or-json-config-file YAML %}

```yaml
# config.yaml
kumaDPUser: dataplane
verbose: true
```

```sh
kumactl install transparent-proxy --config-file config.yaml
```

{% endtab %}
{% tab transparent-proxy-config-yaml-or-json-config-file JSON %}

```json
{
  "kumaDPUser": "dataplane",
  "verbose": true
}
```

```sh
kumactl install transparent-proxy --config-file config.json
```

{% endtab %}
{% endtabs %}

Providing raw configuration directly via the <code>--config</code> flag:

{% tabs transparent-proxy-config-yaml-or-json-config useUrlFragment=false %}
{% tab transparent-proxy-config-yaml-or-json-config YAML %}

```sh
kumactl install transparent-proxy --config "{ kumaDPUser: dataplane, verbose: true }"
```

{% endtab %}
{% tab transparent-proxy-config-yaml-or-json-config JSON %}

```sh
kumactl install transparent-proxy --config '{"kumaDPUser":"dataplane","verbose":true}'
```

{% endtab %}
{% endtabs %}

Passing configuration via [STDIN](https://en.wikipedia.org/wiki/Standard_streams#Standard_input_(stdin)) using the <code>--config-file</code> flag with <code>-</code>:

{% tabs transparent-proxy-config-yaml-or-json-config-file-stdin useUrlFragment=false %}
{% tab transparent-proxy-config-yaml-or-json-config-file-stdin YAML %}

```sh
echo '
kumaDPUser: dataplane
verbose: true
' | kumactl install transparent-proxy --config-file -
```

{% endtab %}
{% tab transparent-proxy-config-yaml-or-json-config-file-stdin JSON %}

```sh
echo '{
  "kumaDPUser": "dataplane",
  "verbose": true
}' | kumactl install transparent-proxy --config-file -
```

{% endtab %}
{% endtabs %}

#### (Universal) Environment Variables

You can customize configuration settings by using environment variables. For example:

{% tabs transparent-proxy-config-env-vars useUrlFragment=false %}
{% tab transparent-proxy-config-env-vars Universal %}

```sh
KUMA_TRANSPARENT_PROXY_IP_FAMILY_MODE="ipv4" kumactl install transparent-proxy
```

{% endtab %}
{% tab transparent-proxy-config-env-vars Kubernetes %}

{% warning %}
In Kubernetes environments, it's technically possible to configure the transparent proxy using environment variables, but this is **strongly discouraged**. The Kubernetes implementation of the transparent proxy is not designed to be configured this way, meaning it hasn't been tested and may lead to unexpected behavior or confusion during debugging.

Here’s an example for illustration purposes only. **Please do not use this method**:

```sh
kumactl install control-plane \
  --set {{site.set_flag_values_prefix}}controlPlane.envVars.KUMA_RUNTIME_KUBERNETES_INJECTOR_SIDECAR_CONTAINER_ENV_VARS="KUMA_TRANSPARENT_PROXY_IP_FAMILY_MODE:ipv4"
```
{% endwarning %}

{% endtab %}
{% endtabs %}

{% tip %}
Refer to the [Configuration Reference](#configuration-reference) for the complete list of environment variables.
{% endtip %}

#### (Universal) CLI Flags

Most configuration values can also be specified directly through CLI flags. For example:

```sh
kumactl install transparent-proxy --kuma-dp-user dataplane --verbose
```

{% warning %}
The following settings cannot be modified directly via CLI flags (corresponding flags are not available):

- <code>redirect.dns.resolvConfigPath</code>
- <code>redirect.inbound.includePorts</code>
- <code>redirect.inbound.excludePortsForUIDs</code>
- <code>redirect.outbound.enabled</code>
- <code>redirect.outbound.includePorts</code>
- <code>ebpf.instanceIPEnvVarName</code>
- <code>log.level</code>
- <code>cniMode</code>
{% endwarning %}

{% tip %}
For a full list of available CLI flags, refer to the [Configuration Reference](#configuration-reference).
{% endtip %}

#### (Kubernetes) Control Plane Runtime Configuration

#### [(Kubernetes) ConfigMap](#experimental-configuration-from-configmap) <sup>(experimental)</sup> 

#### (Kubernetes) Annotations

There is a list of annotations which can be used on workloads to modify the configuration.

- <code>kuma.io/transparent-proxying-ip-family-mode</code> [<sup>[automatically set]</sup>](#notes)

  desc

- <code>traffic.kuma.io/exclude-inbound-ips</code>

  desc

- <code>traffic.kuma.io/exclude-inbound-ports</code>

  desc

- <code>traffic.kuma.io/exclude-outbound-ips</code>

  desc

- <code>traffic.kuma.io/exclude-outbound-ports</code>

  desc

- <code>traffic.kuma.io/exclude-outbound-ports-for-uids</code>

  desc

- <code>traffic.kuma.io/drop-invalid-packets</code>

  desc

- <code>traffic.kuma.io/iptables-logs</code>

  desc

- <code>kuma.io/transparent-proxying-ebpf</code> [<sup>[automatically set]</sup>](#notes)

  desc

- <code>kuma.io/transparent-proxying-ebpf-bpf-fs-path</code> [<sup>[automatically set]</sup>](#notes)

  desc

- <code>kuma.io/transparent-proxying-ebpf-cgroup-path</code> [<sup>[automatically set]</sup>](#notes)

  desc

- <code>kuma.io/transparent-proxying-ebpf-tc-attach-iface</code> [<sup>[automatically set]</sup>](#notes)

  desc

- <code>kuma.io/transparent-proxying-ebpf-instance-ip-env-var-name</code> [<sup>[automatically set]</sup>](#notes)

  desc

- <code>kuma.io/transparent-proxying-ebpf-programs-source-path</code> [<sup>[automatically set]</sup>](#notes)

  desc

The following annotations also impact the transparent proxy configuration. However, their values are automatically set by {{site.mesh_product_name}} and cannot be manually modified. These annotations are not only used to configure the transparent proxy but also play a role in other internal mechanisms, such as generating [<code>Dataplane</code>](/docs/{{ page.version }}/production/dp-config/dpp/) resources from workloads and creating the [Kubernetes sidecar container with <code>kuma-dp</code>](/docs/{{ page.version }}/production/dp-config/dpp-on-kubernetes/), which is injected alongside workloads:

- <code>kuma.io/sidecar-uid</code>

- <code>kuma.io/transparent-proxying-inbound-port</code>

- <code>kuma.io/transparent-proxying-outbound-port</code>

deprecated:
[kuma.io/builtin-dns](https://kuma.io/docs/2.8.x/reference/kubernetes-annotations/#kumaiobuiltindns)
[kuma.io/builtin-dns-port](https://kuma.io/docs/2.8.x/reference/kubernetes-annotations/#kumaiobuiltindnsport)

kuma.io/transparent-proxying
traffic.kuma.io/transparent-proxy-config
traffic.kuma.io/transparent-proxy-configmap-name

please don't use kuma.io/sidecar-env-vars

kuma.io/application-probe-proxy-port


#### Order of Precedence

When using multiple configuration methods, it's important to understand the order in which they are applied to avoid conflicts and ensure the correct settings are used.

{% tabs transparent-proxy-config-order-of-precedence useUrlFragment=false %}
{% tab transparent-proxy-config-order-of-precedence Universal %}

1. **Default Values**
2. **Values from** <code>--config</code> / <code>--config-file</code> **flags**
3. **Environment Variables**
4. **CLI Flags**

---

**Example**

```yaml
# config.yaml
redirect:
  dns:
    port: 10001
```

```sh
KUMA_TRANSPARENT_PROXY_REDIRECT_DNS_PORT="10002" kumactl install transparent-proxy \
  --config-file config.yaml \
  --redirect-dns-port 10003
```

In the example above, the available values are:

- <code>15053</code> (Default Value) 
- <code>10001</code> (Config File)
- <code>10002</code> (Environment Variable)
- <code>10003</code> (Specific CLI Flag)

Since the <code>--redirect-dns-port</code> CLI flag has the highest precedence, its value (<code>10003</code>) will be used for <code>redirect.dns.port</code>

{% endtab %}
{% tab transparent-proxy-config-order-of-precedence Kubernetes %}

1. Default Values
2. Control Plane Runtime Configuration
3. (optional) [ConfigMap <sup>(experimental)</sup>](#experimental-configuration-from-configmap)
4. Annotations

{% endtab %}
{% endtabs %}

### Configuration Reference

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
    includePorts: []Port
    # List of ports to exclude from inbound traffic redirection
    excludePorts: []Port
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

#### Full Reference

- <strong><code>kumaDPUser</code></strong>

  The username or UID of the user that will run <code>kuma-dp</code>

  <p class="custom-block tip">
    If this value is not provided, the system will default to using the UID <code>5678</code> or the username <code>kuma-dp</code>
  </p>

  | Property                 | Value                                            |
  |--------------------------|--------------------------------------------------|
  | **Type**                 | <code>string</code>                              |
  | **CLI Flag**             | <nobr><code>--kuma-dp-user</code></nobr>         |
  | **Environment Variable** | <code>KUMA_TRANSPARENT_PROXY_KUMA_DP_USER</code> |

   **Examples**
    
   ```sh
   kumactl install transparent-proxy --kuma-dp-user bob
   ```
    
   ```sh
   KUMA_TRANSPARENT_PROXY_KUMA_DP_USER="5679" kumactl install transparent-proxy
   ```

- <strong><code>ipFamilyMode</code></strong>

  The IP family mode used for configuring traffic redirection in the transparent proxy

  | Property                  | Value                                                    |
  |---------------------------|----------------------------------------------------------|
  | **Type**                  | <code>enum</code>                                        |
  | **Default Value**         | <code>"dualstack"</code>                                 |
  | **Allowed Values**        | <code>"dualstack"</code><br /><code>"ipv4"</code>        |
  | **CLI Flag**              | <nobr><code>--ip-family-mode</code></nobr>               |
  | **Environment Variable**  | <code>KUMA_TRANSPARENT_PROXY_IP_FAMILY_MODE</code>       |
  | **Kubernetes Annotation** | <code>kuma.io/transparent-proxying-ip-family-mode</code> |

- <strong><code>redirect</code></strong>

  - <strong><code>inbound</code></strong>

    - <strong><code>enabled</code></strong>

      Enables inbound traffic redirection

      | Property                 | Value                                                        |
      |--------------------------|--------------------------------------------------------------|
      | **Type**                 | <code>bool</code>                                            |
      | **Default Value**        | <code>true</code>                                            |
      | **CLI Flag**             | <nobr><code>--redirect-inbound</code></nobr>                 |
      | **Environment Variable** | <code>KUMA_TRANSPARENT_PROXY_REDIRECT_INBOUND_ENABLED</code> |

    - <strong><code>port</code></strong>

      Port used for redirecting inbound traffic

      | Property                  | Value                                                     |
      |---------------------------|-----------------------------------------------------------|
      | **Type**                  | <code>Port</code> [<sup>[type information]</sup>](#notes) |
      | **Default Value**         | <code>15006</code>                                        |
      | **CLI Flag**              | <nobr><code>--redirect-inbound-port</code></nobr>         |
      | **Environment Variable**  | <code>KUMA_TRANSPARENT_PROXY_REDIRECT_INBOUND_PORT</code> |
      | **Kubernetes Annotation** | <code>kuma.io/transparent-proxying-inbound-port</code>    |

    - <strong><code>includePorts</code></strong>

      List of ports to include in inbound traffic redirection

      <p class="custom-block warning">
        This option cannot be used together with <code>redirect.inbound.excludePorts</code>. If both are specified, <code>redirect.inbound.includePorts</code> will take precedence
      </p>

      | Property                 | Value                                                              |
      |--------------------------|--------------------------------------------------------------------|
      | **Type**                 | <code>[]Port</code> [<sup>[type information]</sup>](#notes)        |
      | **Environment Variable** | <code>KUMA_TRANSPARENT_PROXY_REDIRECT_INBOUND_INCLUDE_PORTS</code> |

    - <strong><code>excludePorts</code></strong>

      List of ports to exclude from inbound traffic redirection

      <p class="custom-block warning">
        This option cannot be used together with <code>redirect.inbound.includePorts</code>. If both are specified, <code>redirect.inbound.includePorts</code> will take precedence
      </p>

      | Property                  | Value                                                              |
      |---------------------------|--------------------------------------------------------------------|
      | **Type**                  | <code>[]Port</code> [<sup>[type information]</sup>](#notes)        |
      | **CLI Flag**              | <nobr><code>--exclude-inbound-ports</code></nobr>                  |
      | **Environment Variable**  | <code>KUMA_TRANSPARENT_PROXY_REDIRECT_INBOUND_EXCLUDE_PORTS</code> |
      | **Kubernetes Annotation** | <code>traffic.kuma.io/exclude-inbound-ports</code>                 |

    - <strong><code>excludePortsForIPs</code></strong>

      List of IP addresses to exclude from inbound traffic redirection for specific ports

      | Property                  | Value                                                                      |
      |---------------------------|----------------------------------------------------------------------------|
      | **Type**                  | <code>[]string</code>                                                      |
      | **CLI Flag**              | <nobr><code>--exclude-inbound-ips</code></nobr>                            |
      | **Environment Variable**  | <code>KUMA_TRANSPARENT_PROXY_REDIRECT_INBOUND_EXCLUDE_PORTS_FOR_IPS</code> |
      | **Kubernetes Annotation** | <code>traffic.kuma.io/exclude-inbound-ips</code>                           |
      | **Format**                | <code>ip[,...]</code>                                                      |

    - <strong><code>excludePortsForUIDs</code></strong>

      List of UIDs to exclude from inbound traffic redirection for specific ports

      | Property                 | Value                                                                       |
      |--------------------------|-----------------------------------------------------------------------------|
      | **Type**                 | <code>[]string</code>                                                       |
      | **Environment Variable** | <code>KUMA_TRANSPARENT_PROXY_REDIRECT_INBOUND_EXCLUDE_PORTS_FOR_UIDS</code> |

    - <strong><code>insertRedirectInsteadOfAppend</code></strong>

      Inserts the redirection rule at the beginning of the chain instead of appending it

      **Details**: For inbound traffic, by default, the last applied iptables rule in the <code>PREROUTING</code> chain of the <code>nat</code> table redirects traffic to our custom chain (<code>KUMA_MESH_INBOUND_REDIRECT</code>) for handling transparent proxying. If there is an existing rule in this chain that redirects traffic to another chain, our default behavior of appending the rule would cause it to be added after the existing one, making our rule ineffective. Specifying this flag changes the behavior to insert the rule at the beginning of the chain, ensuring our rule takes precedence

      <p class="custom-block tip">
        Note that if the <code>redirect.vnet</code> setting is also specified, the default behavior is already to insert the rule, so using this setting will not change that behavior
      </p>

      | Property                 | Value                                                                                  |
      |--------------------------|----------------------------------------------------------------------------------------|
      | **Type**                 | <code>bool</code>                                                                      |
      | **Default Value**        | <code>false</code>                                                                     |
      | **CLI Flag**             | <nobr><code>--redirect-inbound-insert-instead-of-append</code></nobr>                  |
      | **Environment Variable** | <code>KUMA_TRANSPARENT_PROXY_REDIRECT_INBOUND_INSERT_REDIRECT_INSTEAD_OF_APPEND</code> |

  - <strong><code>outbound</code></strong>

    - <strong><code>enabled</code></strong>

      Enables outbound traffic redirection

      | Property                 | Value                                                         |
      |--------------------------|---------------------------------------------------------------|
      | **Type**                 | <code>bool</code>                                             |
      | **Default Value**        | <code>true</code>                                             |
      | **Environment Variable** | <code>KUMA_TRANSPARENT_PROXY_REDIRECT_OUTBOUND_ENABLED</code> |

    - <strong><code>port</code></strong>

      Port used for redirecting outbound traffic

      | Property                  | Value                                                      |
      |---------------------------|------------------------------------------------------------|
      | **Type**                  | <code>Port</code> [<sup>[type information]</sup>](#notes)  |
      | **Default Value**         | <code>15001</code>                                         |
      | **CLI Flag**              | <nobr><code>--redirect-outbound-port</code></nobr>         |
      | **Environment Variable**  | <code>KUMA_TRANSPARENT_PROXY_REDIRECT_OUTBOUND_PORT</code> |
      | **Kubernetes Annotation** | <code>kuma.io/transparent-proxying-outbound-port</code>    |

    - <strong><code>includePorts</code></strong>

      List of ports to include in outbound traffic redirection

      <p class="custom-block warning">
        This option cannot be used together with <code>redirect.outbound.excludePorts</code>. If both are specified, <code>redirect.outbound.includePorts</code> will take precedence
      </p>

      | Property                 | Value                                                               |
      |--------------------------|---------------------------------------------------------------------|
      | **Type**                 | <code>[]Port</code> [<sup>[type information]</sup>](#notes)         |
      | **Environment Variable** | <code>KUMA_TRANSPARENT_PROXY_REDIRECT_OUTBOUND_INCLUDE_PORTS</code> |

    - <strong><code>excludePorts</code></strong>

      List of ports to exclude from outbound traffic redirection

      <p class="custom-block warning">
        This option cannot be used together with <code>redirect.outbound.includePorts</code>. If both are specified, <code>redirect.outbound.includePorts</code> will take precedence
      </p>

      | Property                  | Value                                                               |
      |---------------------------|---------------------------------------------------------------------|
      | **Type**                  | <code>[]Port</code> [<sup>[type information]</sup>](#notes)         |
      | **CLI Flag**              | <nobr><code>--exclude-outbound-ports</code></nobr>                  |
      | **Environment Variable**  | <code>KUMA_TRANSPARENT_PROXY_REDIRECT_OUTBOUND_EXCLUDE_PORTS</code> |
      | **Kubernetes Annotation** | <code>traffic.kuma.io/exclude-outbound-ports</code>                 |
      | **Format**                | <code>port[,...]</code>                                             |

    - <strong><code>excludePortsForIPs</code></strong>

      List of IP addresses to exclude from outbound traffic redirection for specific ports

      | Property                  | Value                                                                            |
      |---------------------------|----------------------------------------------------------------------------------|
      | **Type**                  | <code>[]string</code>                                                            |
      | **CLI Flag**              | <nobr><code>--exclude-outbound-ips</code></nobr> [<sup>[repeated]</sup>](#notes) |
      | **Environment Variable**  | <code>KUMA_TRANSPARENT_PROXY_REDIRECT_OUTBOUND_EXCLUDE_PORTS_FOR_IPS</code>      |
      | **Kubernetes Annotation** | <code>traffic.kuma.io/exclude-outbound-ips</code>                                |
      | **Format**                | <code>ip[,...]</code>                                                            |

    - <strong><code>excludePortsForUIDs</code></strong>

      List of UIDs to exclude from outbound traffic redirection for specific ports

      | Property                  | Value                                                                          |
      |---------------------------|--------------------------------------------------------------------------------|
      | **Type**                  | <code>[]string</code>                                                          |
      | **Environment Variable**  | <code>KUMA_TRANSPARENT_PROXY_REDIRECT_OUTBOUND_EXCLUDE_PORTS_FOR_UIDS</code>   |
      | **CLI Flag**              | <code>--exclude-outbound-ports-for-uids</code> [<sup>[repeated]</sup>](#notes) |
      | **Kubernetes Annotation** | <code>traffic.kuma.io/exclude-outbound-ports-for-uids</code>                   |
      | **Format**                | <code>[[protocol:][ports:]uids][;...]</code>                                   |

      **Examples**
      
      - Exclude outbound **TCP** and **UDP** traffic to all ports for processes owned by user with **UID** <code>1000</code>:
      
        ```sh
        kumactl install transparent-proxy \
          --exclude-outbound-ports-for-uids "1000"
        ```
      
      - Exclude outbound **UDP** traffic to all ports for processes owned by user with **UID** <code>1000</code>:
      
        ```sh
        kumactl install transparent-proxy \
          --exclude-outbound-ports-for-uids "udp:*:1000"
        ```
      
      - Exclude outbound **TCP** traffic to port <code>22</code> and ports <code>80–88</code> for processes owned by users with **UIDs** in the range <code>1000–1002</code>:
      
        ```sh
        kumactl install transparent-proxy \
          --exclude-outbound-ports-for-uids "tcp:22,80-88:1000-1002"
        ```
      
      - Exclude outbound **TCP** and **UDP** traffic to all ports for processes owned by users with **UIDs** in the range <code>1000–1100</code>, and exclude outbound **UDP** traffic to all ports for processes owned by user with **UID** <code>2000</code>:
      
        ```sh
        kumactl install transparent-proxy \
          --exclude-outbound-ports-for-uids "1000-1100;udp:*:2000"
        ```
      
        ```sh
        kumactl install transparent-proxy \
          --exclude-outbound-ports-for-uids "1000-1100" \
          --exclude-outbound-ports-for-uids "udp:*:2000"
        ```

    - <strong><code>insertRedirectInsteadOfAppend</code></strong>

      Inserts the redirection rule at the beginning of the chain instead of appending it

      **Details**: For outbound traffic, by default, the last applied iptables rule in the <code>OUTPUT</code> chain of the <code>nat</code> table redirects traffic to our custom chain (<code>KUMA_MESH_OUTBOUND_REDIRECT</code>), where it is processed for transparent proxying. However, if there is an existing rule in this chain that already redirects traffic to another chain, our default behavior of appending the rule will cause our rule to be added after the existing one, effectively ignoring it. When this flag is specified, it changes the behavior from appending to inserting the rule at the beginning of the chain, ensuring that our iptables rule takes precedence

      | Property                 | Value                                                                                   |
      |--------------------------|-----------------------------------------------------------------------------------------|
      | **Type**                 | <code>bool</code>                                                                       |
      | **Default Value**        | <code>false</code>                                                                      |
      | **CLI Flag**             | <nobr><code>--redirect-outbound-insert-instead-of-append</code></nobr>                  |
      | **Environment Variable** | <code>KUMA_TRANSPARENT_PROXY_REDIRECT_OUTBOUND_INSERT_REDIRECT_INSTEAD_OF_APPEND</code> |

  - <strong><code>dns</code></strong>

    - <strong><code>enabled</code></strong>

      Enables redirection of DNS queries to the DNS server managed by {{site.mesh_product_name}}, listening on the port specified in the <code>redirect.dns.port</code> setting

      <p class="custom-block tip">
        When <code>redirect.dns.captureAll</code> is disabled, only queries directed to servers listed in the file specified via <code>redirect.dns.resolvConfigPath</code>) will be redirected. If <code>redirect.dns.captureAll</code> is enabled, all DNS queries will be redirected, regardless of the target DNS server
      </p>

      | Property                  | Value                                                    |
      |---------------------------|----------------------------------------------------------|
      | **Type**                  | <code>bool</code>                                        |
      | **Default Value**         | <code>true</code>                                        |
      | **Environment Variable**  | <code>KUMA_TRANSPARENT_PROXY_REDIRECT_DNS_ENABLED</code> |

    - <strong><code>port</code></strong>

      The port where the DNS server managed by {{site.mesh_product_name}} is listening

      | Property                 | Value                                                     |
      |--------------------------|-----------------------------------------------------------|
      | **Type**                 | <code>Port</code> [<sup>[type information]</sup>](#notes) |
      | **Default Value**        | <code>15053</code>                                        |
      | **CLI Flag**             | <nobr><code>--redirect-dns-port</code></nobr>             |
      | **Environment Variable** | <code>KUMA_TRANSPARENT_PROXY_REDIRECT_DNS_PORT</code>     |

    - <strong><code>captureAll</code></strong>

      Redirect all DNS traffic to the DNS server managed by {{site.mesh_product_name}}, listening on the port specified in the <code>redirect.dns.port</code> setting

      <p class="custom-block warning">
        This setting requires <nobr><code>redirect.dns.enabled</code></nobr>, which is disabled by default. However, using the <nobr><code>--redirect-all-dns-traffic</code></nobr> flag automatically enables it. Note that combining <nobr><code>--redirect-all-dns-traffic</code></nobr> with <nobr><code>--redirect-dns</code></nobr> is incorrect and will result in an error. In all other cases, ensure <nobr><code>redirect.dns.enabled</code></nobr> is explicitly enabled via the appropriate environment variable or in the <code>JSON</code> / <code>YAML</code> configuration
      </p>

      | Property                 | Value                                                        |
      |--------------------------|--------------------------------------------------------------|
      | **Type**                 | <code>bool</code>                                            |
      | **Default Value**        | <code>false</code>                                           |
      | **CLI Flag**             | <nobr><code>--redirect-all-dns-traffic</code></nobr>         |
      | **Environment Variable** | <code>KUMA_TRANSPARENT_PROXY_REDIRECT_DNS_CAPTURE_ALL</code> |

    - <strong><code>skipConntrackZoneSplit</code></strong>

      Disables conntrack zone splitting, which can prevent potential DNS issues

      **Details**: The conntrack zone splitting feature is used to avoid DNS resolution errors when applications make numerous DNS UDP requests. Normally, we separate conntrack zones to ensure proper handling of DNS traffic: Zone 2 handles DNS packets between the application and the local proxy, while Zone 1 manages packets between the proxy and upstream DNS resolvers. Disabling this feature should only be done if necessary, for example, in environments where custom iptables rules are already manipulating DNS traffic (e.g., inside Docker containers in custom networks when redirecting all DNS traffic \[<code>redirect.dns.captureAll</code> is enabled\])

      | Property                 | Value                                                                      |
      |--------------------------|----------------------------------------------------------------------------|
      | **Type**                 | <code>bool</code>                                                          |
      | **Default Value**        | <code>false</code>                                                         |
      | **CLI Flag**             | <nobr><code>--skip-dns-conntrack-zone-split</code></nobr>                  |
      | **Environment Variable** | <code>KUMA_TRANSPARENT_PROXY_REDIRECT_DNS_SKIP_CONNTRACK_ZONE_SPLIT</code> |

    - <strong><code>resolvConfigPath</code></strong>

      Specifies the path to the <code>resolv.conf</code> file used to parse the DNS servers for redirecting DNS queries

      <p class="custom-block tip">
        This setting is taken into account only when <code>redirect.dns.captureAll</code> is not enabled
      </p>

      | Property                 | Value                                                               |
      |--------------------------|---------------------------------------------------------------------|
      | **Type**                 | <code>string</code>                                                 |
      | **Default Value**        | <code>"/etc/resolv.conf"</code>                                     |
      | **Environment Variable** | <code>KUMA_TRANSPARENT_PROXY_REDIRECT_DNS_RESOLV_CONFIG_PATH</code> |

  - <strong><code>vnet</code></strong>

    - <strong><code>networks</code></strong>

      Specifies virtual networks using the format <code>interfaceName:CIDR</code> Allows matching traffic on specific network interfaces

      **Examples**:

      - <code>docker0:172.17.0.0/16</code>
      - <code>br+:172.18.0.0/16</code> (matches any interface with name starting with <code>br</code>)
      - <code>iface:::1/64</code> (for IPv6)

      | Property                 | Value                                                      |
      |--------------------------|------------------------------------------------------------|
      | **Type**                 | <code>[]string</code>                                      |
      | **CLI Flag**             | <nobr><code>--vnet</code></nobr>                           |
      | **Environment Variable** | <code>KUMA_TRANSPARENT_PROXY_REDIRECT_VNET_NETWORKS</code> |

- <strong><code>ebpf</code></strong>

  <p class="custom-block warning">
    eBPF implementation is experimental. Use with caution
  </p>

  - <strong><code>enabled</code></strong>

    Enables eBPF support for handling traffic redirection in the transparent proxy

    | Property                  | Value                                            |
    |---------------------------|--------------------------------------------------|
    | **Type**                  | <code>bool</code>                                |
    | **Default Value**         | <code>false</code>                               |
    | **CLI Flag**              | <nobr><code>--ebpf-enabled</code></nobr>         |
    | **Environment Variable**  | <code>KUMA_TRANSPARENT_PROXY_EBPF_ENABLED</code> |
    | **Kubernetes Annotation** | <code>kuma.io/transparent-proxying-ebpf</code>   |

  - <strong><code>instanceIP</code></strong>

    IP address of the instance (pod/vm) where transparent proxy will be installed

    <p class="custom-block warning">
      Mutually exclusive with <code>ebpf.instanceIPEnvVarName</code>
    </p>

    | Property                 | Value                                                |
    |--------------------------|------------------------------------------------------|
    | **Type**                 | <code>string</code>                                  |
    | **CLI Flag**             | <nobr><code>--ebpf-instance-ip</code></nobr>         |
    | **Environment Variable** | <code>KUMA_TRANSPARENT_PROXY_EBPF_INSTANCE_IP</code> |

  - <strong><code>instanceIPEnvVarName</code></strong>

    The name of the environment variable containing the IP address of the instance (pod/vm) where transparent proxy will be installed

    <p class="custom-block warning">
      Mutually exclusive with <code>ebpf.instanceIP</code>
    </p>

    | Property                  | Value                                                                                |
    |---------------------------|--------------------------------------------------------------------------------------|
    | **Type**                  | <code>string</code>                                                                  |
    | **Environment Variable**  | <code>KUMA_TRANSPARENT_PROXY_EBPF_INSTANCE_IP_ENV_VAR_NAME</code>                    |
    | **Kubernetes Annotation** | <nobr><code>kuma.io/transparent-proxying-ebpf-instance-ip-env-var-name</code></nobr> |

  - <strong><code>bpffsPath</code></strong>

    The path of the BPF filesystem

    | Property                  | Value                                                      |
    |---------------------------|------------------------------------------------------------|
    | **Type**                  | <code>string</code>                                        |
    | **Default Value**         | <code>"/run/kuma/bpf"</code>                               |
    | **CLI Flag**              | <nobr><code>--ebpf-bpffs-path</code></nobr>                |
    | **Environment Variable**  | <code>KUMA_TRANSPARENT_PROXY_EBPF_BPFFS_PATH</code>        |
    | **Kubernetes Annotation** | <code>kuma.io/transparent-proxying-ebpf-bpf-fs-path</code> |

  - <strong><code>cgroupPath</code></strong>

    The path of cgroup2

    | Property                  | Value                                                      |
    |---------------------------|------------------------------------------------------------|
    | **Type**                  | <code>string</code>                                        |
    | **Default Value**         | <code>"/sys/fs/cgroup"</code>                              |
    | **CLI Flag**              | <nobr><code>--ebpf-cgroup-path</code></nobr>               |
    | **Environment Variable**  | <code>KUMA_TRANSPARENT_PROXY_EBPF_CGROUP_PATH</code>       |
    | **Kubernetes Annotation** | <code>kuma.io/transparent-proxying-ebpf-cgroup-path</code> |

  - <strong><code>programsSourcePath</code></strong>

    Path where compiled eBPF programs and other necessary files for eBPF mode can be found

    | Property                  | Value                                                               |
    |---------------------------|---------------------------------------------------------------------|
    | **Type**                  | <code>string</code>                                                 |
    | **Default Value**         | <code>"/tmp/kuma-ebpf"</code>                                       |
    | **CLI Flag**              | <nobr><code>--ebpf-programs-source-path</code></nobr>               |
    | **Environment Variable**  | <code>KUMA_TRANSPARENT_PROXY_EBPF_PROGRAMS_SOURCE_PATH</code>       |
    | **Kubernetes Annotation** | <code>kuma.io/transparent-proxying-ebpf-programs-source-path</code> |

  - <strong><code>tcAttachIface</code></strong>

    The network interface for TC eBPF programs to bind to. If not provided, it will be automatically determined

    | Property                  | Value                                                          |
    |---------------------------|----------------------------------------------------------------|
    | **Type**                  | <code>string</code>                                            |
    | **CLI Flag**              | <nobr><code>--ebpf-tc-attach-iface</code></nobr>               |
    | **Environment Variable**  | <code>KUMA_TRANSPARENT_PROXY_EBPF_TC_ATTACH_IFACE</code>       |
    | **Kubernetes Annotation** | <code>kuma.io/transparent-proxying-ebpf-tc-attach-iface</code> |

- <strong><code>retry</code></strong>

  - <strong><code>maxRetries</code></strong>

    The maximum number of retry attempts for operations

    | Property                 | Value                                                 |
    |--------------------------|-------------------------------------------------------|
    | **Type**                 | <code>uint</code>                                     |
    | **Default Value**        | <code>4</code>                                        |
    | **CLI Flag**             | <nobr><code>--max-retries</code></nobr>               |
    | **Environment Variable** | <code>KUMA_TRANSPARENT_PROXY_RETRY_MAX_RETRIES</code> |

  - <strong><code>sleepBetweenRetries</code></strong>

    The time duration to wait between retry attempts

    | Property                 | Value                                                           |
    |--------------------------|-----------------------------------------------------------------|
    | **Type**                 | <code>Duration</code> [<sup>[type information]</sup>](#notes)   |
    | **Default Value**        | <code>"2s"</code>                                               |
    | **CLI Flag**             | <nobr><code>--sleep-between-retries</code></nobr>               |
    | **Environment Variable** | <code>KUMA_TRANSPARENT_PROXY_RETRY_SLEEP_BETWEEN_RETRIES</code> |

- <strong><code>iptablesExecutables</code></strong>

  Specifies custom paths for iptables executables

  <p class="custom-block warning">
    You must provide all three executables for each IP version you want to customize (IPv4 or IPv6), meaning if you configure one for IPv6 (e.g., <code>ip6tables</code>), you must also specify <code>ip6tables-save</code> and <code>ip6tables-restore</code>. Partial configurations for either IPv4 or IPv6 are not allowed
  </p>

  <p class="custom-block warning">
    Provided paths are not extensively validated, so ensure you specify correct paths and that the executables are actual iptables binaries to avoid misconfigurations and unexpected behavior
  </p>

  <p class="custom-block tip">
    Configuration values can be set through a combination of sources: config file (via <nobr><code>--config</code></nobr> or <nobr><code>--config-file</code></nobr>), environment variables, and the <nobr><code>--iptables-executables</code></nobr> flag. For example, you can specify <code>ip6tables</code> in the config file, <nobr><code>ip6tables-save</code></nobr> as an environment variable, and <nobr><code>ip6tables-restore</code></nobr> via the <nobr><code>--iptables-executables</code></nobr> flag
  </p>

  | Property                                     | Value                                                                                                                                                                                                                                                      |
  |----------------------------------------------|------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
  | **Type**                                     | <code>object</code>                                                                                                                                                                                                                                        |
  | **Allowed Object Keys**                      | <code>iptables</code><br /><nobr><code>iptables-save</code></nobr><br /><nobr><code>iptables-restore</code></nobr><br /><nobr><code>ip6tables</code></nobr><br /><nobr><code>ip6tables-save</code></nobr><br /><nobr><code>ip6tables-restore</code></nobr> |
  | **CLI Flag**                                 | <nobr><code>--iptables-executables</code></nobr>                                                                                                                                                                                                           |
  | **Environment Variable**                     | <code>KUMA_TRANSPARENT_PROXY_IPTABLES_EXECUTABLES</code>                                                                                                                                                                                                   |
  | **CLI Flag and Environment Variable Format** | <code>key:path[,key:path...]</code>                                                                                                                                                                                                                        |

- <strong><code>log</code></strong>

  - <strong><code>enabled</code></strong>

    Determines whether iptables rules logging is activated. When <code>true</code>, each packet matching an iptables rule will have its details logged, aiding in diagnostics and monitoring of packet flows

    | Property                  | Value                                           |
    |---------------------------|-------------------------------------------------|
    | **Type**                  | <code>bool</code>                               |
    | **Default Value**         | <code>false</code>                              |
    | **CLI Flag**              | <nobr><code>--iptables-logs</code></nobr>       |
    | **Environment Variable**  | <code>KUMA_TRANSPARENT_PROXY_LOG_ENABLED</code> |
    | **Kubernetes Annotation** | <code>traffic.kuma.io/iptables-logs</code>      |

  - <strong><code>level</code></strong>

    Specifies the log level for iptables logging as defined by netfilter. This level controls the verbosity and detail of the log entries for matching packets. Higher values increase the verbosity. The exact behavior can depend on the system's syslog configuration
    
    **Available log levels**:

    - <code>0</code> - emergency (system is unusable)
    - <code>1</code> - alert (action must be taken immediately)
    - <code>2</code> - critical (critical conditions)
    - <code>3</code> - error (error conditions)
    - <code>4</code> - warning (warning conditions)
    - <code>5</code> - notice (normal but significant condition)
    - <code>6</code> - info (informational)
    - <code>7</code> - debug (debug-level messages)

    | Property                 | Value                                                                                                                          |
    |--------------------------|--------------------------------------------------------------------------------------------------------------------------------|
    | **Type**                 | <code>enum</code>                                                                                                              |
    | **Default Value**        | <code>7</code>                                                                                                                 |
    | **Allowed Values**       | <code>0</code>, <code>1</code>, <code>2</code>, <code>3</code>, <code>4</code>, <code>5</code>, <code>6</code>, <code>7</code> |
    | **Environment Variable** | <code>KUMA_TRANSPARENT_PROXY_LOG_LEVEL</code>                                                                                  |

- <strong><code>comments</code></strong>

  - <strong><code>disabled</code></strong>

    Disables the addition of comments to iptables rules

    <p class="custom-block warning">
      Disabling comments is strongly discouraged, as they are essential for properly uninstalling the transparent proxy. If comments are disabled, the <code>kumactl uninstall transparent-proxy</code> command will not function, and you'll need to manually remove the related iptables rules when necessary
    </p>

    | Property                 | Value                                                 |
    |--------------------------|-------------------------------------------------------|
    | **Type**                 | <code>bool</code>                                     |
    | **Default Value**        | <code>false</code>                                    |
    | **CLI Flag**             | <nobr><code>--disable-comments</code></nobr>          |
    | **Environment Variable** | <code>KUMA_TRANSPARENT_PROXY_COMMENTS_DISABLED</code> |

- <strong><code>wait</code></strong>

  Time in seconds to wait for acquiring the xtables lock before failing. Value <code>0</code> means wait indefinitely

  | Property                 | Value                                    |
  |--------------------------|------------------------------------------|
  | **Type**                 | <code>uint</code>                        |
  | **Default Value**        | <code>5</code>                           |
  | **CLI Flag**             | <nobr><code>--wait</code></nobr>         |
  | **Environment Variable** | <code>KUMA_TRANSPARENT_PROXY_WAIT</code> |

- <strong><code>waitInterval</code></strong>

  Time interval between retries to acquire the xtables lock in seconds

  | Property                 | Value                                             |
  |--------------------------|---------------------------------------------------|
  | **Type**                 | <code>uint</code>                                 |
  | **Default Value**        | <code>0</code>                                    |
  | **CLI Flag**             | <nobr><code>--wait-interval</code></nobr>         |
  | **Environment Variable** | <code>KUMA_TRANSPARENT_PROXY_WAIT_INTERVAL</code> |

- <strong><code>dropInvalidPackets</code></strong>

  Drops invalid packets to avoid connection resets in high-throughput scenarios

  **Details**: This setting enables dropping of packets in invalid states, improving application stability by preventing them from reaching the backend. This is particularly beneficial during high-throughput requests where out-of-order packets might bypass DNAT

  <p class="custom-block warning">
    Note that enabling this flag may introduce slight performance overhead. Weigh the trade-off between connection stability and performance before enabling it
  </p>

  | Property                  | Value                                                    |
  |---------------------------|----------------------------------------------------------|
  | **Type**                  | <code>bool</code>                                        |
  | **Default Value**         | <code>false</code>                                       |
  | **CLI Flag**              | <nobr><code>--drop-invalid-packets</code></nobr>         |
  | **Environment Variable**  | <code>KUMA_TRANSPARENT_PROXY_DROP_INVALID_PACKETS</code> |
  | **Kubernetes Annotation** | <code>traffic.kuma.io/drop-invalid-packets</code>        |

- <strong><code>storeFirewalld</code></strong>

  Enables firewalld support to store iptables rules

  | Property                 | Value                                               |
  |--------------------------|-----------------------------------------------------|
  | **Type**                 | <code>bool</code>                                   |
  | **Default Value**        | <code>false</code>                                  |
  | **CLI Flag**             | <nobr><code>--store-firewalld</code></nobr>         |
  | **Environment Variable** | <code>KUMA_TRANSPARENT_PROXY_STORE_FIREWALLD</code> |

- <strong><code>cniMode</code></strong>

  | Property                 | Value                                        |
  |--------------------------|----------------------------------------------|
  | **Type**                 | <code>bool</code>                            |
  | **Default Value**        | <code>false</code>                           |
  | **Environment Variable** | <code>KUMA_TRANSPARENT_PROXY_CNI_MODE</code> |

- <strong><code>dryRun</code></strong>

  Enables dry-run mode

  | Property                 | Value                                       |
  |--------------------------|---------------------------------------------|
  | **Type**                 | <code>bool</code>                           |
  | **Default Value**        | <code>false</code>                          |
  | **CLI Flag**             | <nobr><code>--dry-run</code></nobr>         |
  | **Environment Variable** | <code>KUMA_TRANSPARENT_PROXY_DRY_RUN</code> |

- <strong><code>verbose</code></strong>

  Enables verbose mode with longer argument/flag names and additional comments

  | Property                 | Value                                       |
  |--------------------------|---------------------------------------------|
  | **Type**                 | <code>bool</code>                           |
  | **Default Value**        | <code>false</code>                          |
  | **CLI Flag**             | <nobr><code>--verbose</code></nobr>         |
  | **Environment Variable** | <code>KUMA_TRANSPARENT_PROXY_VERBOSE</code> |

#### Notes

<table>

<tr>
<td><sup><nobr>[format reference]</nobr></sup></td>
<td>
Format refers to the required format for specifying values when using <strong>Environment Variables</strong>, specific <strong>CLI Flags</strong>, or <strong>Kubernetes Annotations</strong>. When providing values via <strong>YAML</strong> or <strong>JSON</strong> using the <code>--config</code> or <code>--config-file</code> flags, the format should match the setting's expected data type.

For example, to configure <code>redirect.outbound.excludePortsForIPs</code>:

{% tabs transparent-proxy-config-notes-value-format useUrlFragment=false %}
{% tab transparent-proxy-config-notes-value-format <code>YAML</code> / <code>JSON</code> %}

```yaml
# config.yaml
redirect:
  outbound:
    excludePortsForIPs:
    - 10.0.0.1
    - 172.1.0.0/24
```

```sh
kumactl install transparent-proxy --config-file config.yaml
```

{% endtab %}
{% tab transparent-proxy-config-notes-value-format Environment Variable %}

```sh
KUMA_TRANSPARENT_PROXY_REDIRECT_OUTBOUND_EXCLUDE_PORTS_FOR_IPS="10.0.0.1,172.1.0.0/24" kumactl install transparent-proxy
```

{% endtab %}
{% tab transparent-proxy-config-notes-value-format CLI Flag %}

```sh
kumactl install transparent-proxy --exclude-outbound-ips "10.0.0.1,172.1.0.0/24"
```

{% endtab %}
{% tab transparent-proxy-config-notes-value-format Kubernetes Annotation %}

```yaml
metadata:
  annotations:
    traffic.kuma.io/exclude-outbound-ips: "10.0.0.1,172.1.0.0/24"
...
```

{% endtab %}
{% endtabs %}
</td>
</tr>
<tr>
<td><sup>[repeated]</sup></td>
<td>
If a <strong>CLI flag</strong> is marked as <strong>repeated</strong>, it means the flag can be used multiple times. For example:

<pre class="highlight line-numbers language-sh">
<code class="language-sh">kumactl install transparent-proxy \
  --exclude-outbound-ips "10.0.0.1,172.1.0.0/24" \
  --exclude-outbound-ips "fe80::/10"</code>
</pre>

</td>
</tr>

<tr>
<td><sup>[type information]</sup></td>
<td>
<ul>
<li><code>Port</code> - uint16 value greater than <code>0</code></li>
<li><code>Duration</code> - string representation of time duration, i.e. <code>"10s"</code>, <code>"20m"</code>, <code>"1h"</code> etc.</li>
</ul>
</td>
</tr>

<tr>
<td><nobr><sup>[automatically set]</sup></nobr></td>
<td>
This annotation is always applied, and the value will reflect the actual configuration being used to install the transparent proxy
</td>
</tr>

</table>


## How to install Transparent Proxy?

### Universal

### Kubernetes

## How does the Transparent Proxy works

## Modifications

- Underlying Configuration structure
  * Mention that provided structure is just simplification and not the exact representation of the structure we are using
- Kubernetes
  - How to modify tproxy configuration?
    - Ways of configuring tproxy
      - runtime configuration
        - env vars
      - annotations
    - What not to do?
      * use env vars to configure tproxy
      * provide different values of some properties than the ones set in runtime configuration
- How does the tproxy work?
