+++
title = "prometheus_config"
weight = 200
+++

# prometheus_config

The `prometheus_config` block is used to define a collection of Prometheus
Instances, each of which is its own mini-Agent. Most users will only need to
define one instance.

```yaml
# Configures the optional scraping service to cluster agents.
[scraping_service: <scraping_service_config>]

# Configures the gRPC client used for agents to connect to other
# clustered agents.
[scraping_service_client: <scraping_service_client_config>]

# Configure values for all Prometheus instances.
[global: <global_config>]

# Configure the directory used by instances to store their WAL.
[wal_directory: <string> | default = ""]

# Configures how long ago an abandoned (not associated with an instance) WAL
# may be written to before being eligible to be deleted
[wal_cleanup_age: <duration> | default = "12h"]

# Configures how often checks for abandoned WALs to be deleted are performed.
# A value of 0 disables periodic cleanup of abandoned WALs
[wal_cleanup_period: <duration> | default = "30m"]

# The list of Prometheus instances to launch with the agent.
configs:
  [- <prometheus_instance_config>]

# If an instance crashes abnormally, how long should we wait before trying
# to restart it. 0s disables the backoff period and restarts the agent
# immediately.
[instance_restart_backoff: <duration> | default = "5s"]

# How to spawn instances based on instance configs. Supported values: shared,
# distinct.
[instance_mode: <string> | default = "shared"]

```

## server_tls_config

The `http_tls_config` block configures the server to run with TLS. When set, `integrations.http_tls_config` must
also be provided. Acceptable values for  `client_auth_type` are found in
[Go's `tls` package](https://golang.org/pkg/crypto/tls/#ClientAuthType).

```yaml
# File path to the server certificate
[cert_file: <string>]

# File path to the server key
[key_file: <string>]

# Tells the server what is acceptable from the client, this drives the options in client_tls_config
[client_auth_type: <string>]

# File path to the signing CA certificate, needed if CA is not trusted
[client_ca_file: <string>]

```

## scraping_service_config

The `scraping_service` block configures the
[scraping service](../scraping-service.md), an operational
mode where configurations are stored centrally in a KV store and a cluster of
agents distribute discovery and scrape load between nodes.

```yaml
# Whether to enable scraping service mode. When enabled, local configs
# cannot be used.
[enabled: boolean | default = false]

# How often should the agent manually reshard. Useful for if KV change
# events are not sent by an agent.
[reshard_interval: <duration> | default = "1m"]

# The timeout for a reshard. Applies to a cluster-wide reshard (done when
# joining or leaving the cluster) and local reshards (done every
# reshard_interval). A timeout of 0 indicates no timeout.
[reshard_timeout: <duration> | default = "30s"]

# Configuration for the KV store to store configurations.
kvstore: <kvstore_config>

# When set, allows configs pushed to the KV store to specify configuration
# fields that can read secrets from files.
#
# This is disabled by default. When enabled, a malicious user can craft an
# instance config that reads arbitrary files on the machine the Agent runs
# on and sends its contents to a specically crafted remote_write endpoint.
#
# If enabled, ensure that no untrusted users have access to the Agent API.
[dangerous_allow_reading_files: <boolean>]

# Configuration for how agents will cluster together.
lifecycler: <lifecycler_config>
```

## kvstore_config

The `kvstore_config` block configures the KV store used as storage for
configurations in the scraping service mode.

```yaml
# Which underlying KV store to use. Can be either consul or etcd
[store: <string> | default = ""]

# Key prefix to store all configurations with. Must end in /.
[prefix: <string> | default = "configurations/"]

# Configuration for a Consul client. Only applies if store
# is "consul"
consul:
  # The hostname and port of Consul.
  [host: <string> | duration = "localhost:8500"]

  # The ACL Token used to interact with Consul.
  [acltoken: <string>]

  # The HTTP timeout when communicating with Consul
  [httpclienttimeout: <duration> | default = 20s]

  # Whether or not consistent reads to Consul are enabled.
  [consistentreads: <boolean> | default = true]

# Configuration for an ETCD v3 client. Only applies if
# store is "etcd"
etcd:
  # The ETCD endpoints to connect to.
  endpoints:
    - <string>

  # The Dial timeout for the ETCD connection.
  [dial_tmeout: <duration> | default = 10s]

  # The maximum number of retries to do for failed ops to ETCD.
  [max_retries: <int> | default = 10]
```

## lifecycler_config

The `lifecycler_config` block configures the lifecycler; the component that
Agents use to cluster together.

```yaml
# Configures the distributed hash ring storage.
ring:
  # KV store for getting and sending distributed hash ring updates.
  kvstore: <kvstore_config>

  # Specifies when other agents in the clsuter should be considered
  # unhealthy if they haven't sent a heartbeat within this duration.
  [heartbeat_timeout: <duration> | default = "1m"]

# Number of tokens to generate for the distributed hash ring.
[num_tokens: <int> | default = 128]

# How often agents should send a heartbeat to the distributed hash
# ring.
[heartbeat_period: <duration> | default = "5s"]

# How long to wait for tokens from other agents after generating
# a new set to resolve collisions. Useful only when using a gossip
# KV store.
[observe_period: <duration> | default = "0s"]

# Period to wait before joining the ring. 0s means to join immediately.
[join_after: <duration> | default = "0s"]

# Minimum duration to wait before marking the agent as ready to receive
# traffic. Used to work around race conditions for multiple agents exiting
# the distributed hash ring at the same time.
[min_ready_duration: <duration> | default = "1m"]

# Network interfaces to resolve addresses defined by other agents
# registered in distributed hash ring.
[interface_names: <string array> | default = ["eth0", "en0"]]

# Duration to sleep before exiting. Ensures that metrics get scraped
# before the process quits.
[final_sleep: <duration> | default = "30s"]

# File path to store tokens. If empty, tokens will not be stored during
# shutdown and will not be restored at startup.
[tokens_file_path: <string> | default = ""]

# Availability zone of the host the agent is running on. Default is an
# empty string which disables zone awareness for writes.
[availability_zone: <string> | default = ""]
```

## scraping_service_client_config

The `scraping_service_client_config` block configures how clustered Agents will
generate gRPC clients to connect to each other.

```yaml
grpc_client_config:
  # Maximum size in bytes the gRPC client will accept from the connected server.
  [max_recv_msg_size: <int> | default = 104857600]

  # Maximum size in bytes the gRPC client will sent to the connected server.
  [max_send_msg_size: <int> | default = 16777216]

  # Whether messages should be gzipped.
  [use_gzip_compression: <boolean> | default = false]

  # The rate limit for gRPC clients; 0 means no rate limit.
  [rate_limit: <float64> | default = 0]

  # gRPC burst allowed for rate limits.
  [rate_limit_burst: <int> | default = 0]

  # Controls if when a rate limit is hit whether the client should
  # retry the request.
  [backoff_on_ratelimits: <boolean> | default = false]

  # Configures the retry backoff when backoff_on_ratelimits is
  # true.
  backoff_config:
    # The minimum delay when backing off.
    [min_period: <duration> | default = "100ms"]

    # The maximum delay when backing off.
    [max_period: <duration> | default = "10s"]

    # The number of times to backoff and retry before failing.
    [max_retries: <int> | default = 10]
```

## global_config

The `global_config` block configures global values for all launched Prometheus
instances.

```yaml
# How frequently should Prometheus instances scrape.
[scrape_interval: duration | default = "1m"]

# How long to wait before timing out a scrape from a target.
[scrape_timeout: duration | default = "10s"]

# A list of static labels to add for all metrics.
external_labels:
  { <string>: <string> }

# Default set of remote_write endpoints. If an instance doesn't define any
# remote_writes, it will use this list.
remote_write:
  - [<remote_write>]
```

## prometheus_instance_config

The `prometheus_instance_config` block configures an individual Prometheus
instance, which acts as its own mini Prometheus agent.

```yaml
# Name of the instance. Must be present. Will be added as a label to agent
# metrics.
name: string

# Whether this agent instance should only scrape from targets running on the
# same machine as the agent process.
[host_filter: <boolean> | default = false]

# Relabel configs to apply against discovered targets. The relabeling is
# temporary and just used for filtering targets.
host_filter_relabel_configs:
  [ - <relabel_config> ... ]

# How frequently the WAL truncation process should run. Every iteration of
# the truncation will checkpoint old series and remove old samples. If data
# has not been sent within this window, some of it may be lost.
#
# The size of the WAL will increase with less frequent truncations. Making
# truncations more frequent reduces the size of the WAL but increases the
# chances of data loss when remote_write is failing for longer than the
# specified frequency.
[wal_truncate_frequency: <duration> | default = "60m"]

# The minimum amount of time that series and samples should exist in the WAL
# before being considered for deletion. The consumed disk space of the WAL will
# increase by making this value larger.
#
# Setting this value to 0s is valid, but may delete series before all
# remote_write shards have been able to write all data, and may cause errors on
# slower machines.
[min_wal_time: <duration> | default = "5m"]

# The maximum amount of time that series and samples may exist within the WAL
# before being considered for deletion. Series that have not received writes
# since this period will be removed, and all samples older than this period will
# be removed.
#
# This value is useful in long-running network outages, preventing the WAL from
# growing forever.
#
# Must be larger than min_wal_time.
[max_wal_time: <duration> | default = "4h"]

# Deadline for flushing data when a Prometheus instance shuts down
# before giving up and letting the shutdown proceed.
[remote_flush_deadline: <duration> | default = "1m"]

# When true, writes staleness markers to all active series to
# remote_write.
[write_stale_on_shutdown: <boolean> | default = false]

# A list of scrape configuration rules.
scrape_configs:
  - [<scrape_config>]

# A list of remote_write targets.
remote_write:
  - [<remote_write>]
```

## scrape_config

A `scrape_config` section specifies a set of targets and parameters describing
how to scrape them. In the general case, one scrape configuration specifies a
single job. In advanced configurations, this may change.

Targets may be statically configured via the `static_configs` parameter or
dynamically discovered using one of the supported service-discovery mechanisms.

Additionally, `relabel_configs` allow advanced modifications to any target and
its labels before scraping.

```yaml
# The job name assigned to scraped metrics by default.
job_name: <job_name>

# How frequently to scrape targets from this job.
[ scrape_interval: <duration> | default = <global_config.scrape_interval> ]

# Per-scrape timeout when scraping this job.
[ scrape_timeout: <duration> | default = <global_config.scrape_timeout> ]

# The HTTP resource path on which to fetch metrics from targets.
[ metrics_path: <path> | default = /metrics ]

# honor_labels controls how Prometheus handles conflicts between labels that are
# already present in scraped data and labels that Prometheus would attach
# server-side ("job" and "instance" labels, manually configured target
# labels, and labels generated by service discovery implementations).
#
# If honor_labels is set to "true", label conflicts are resolved by keeping label
# values from the scraped data and ignoring the conflicting server-side labels.
#
# If honor_labels is set to "false", label conflicts are resolved by renaming
# conflicting labels in the scraped data to "exported_<original-label>" (for
# example "exported_instance", "exported_job") and then attaching server-side
# labels.
#
# Setting honor_labels to "true" is useful for use cases such as federation and
# scraping the Pushgateway, where all labels specified in the target should be
# preserved.
#
# Note that any globally configured "external_labels" are unaffected by this
# setting. In communication with external systems, they are always applied only
# when a time series does not have a given label yet and are ignored otherwise.
[ honor_labels: <boolean> | default = false ]

# honor_timestamps controls whether Prometheus respects the timestamps present
# in scraped data.
#
# If honor_timestamps is set to "true", the timestamps of the metrics exposed
# by the target will be used.
#
# If honor_timestamps is set to "false", the timestamps of the metrics exposed
# by the target will be ignored.
[ honor_timestamps: <boolean> | default = true ]

# Configures the protocol scheme used for requests.
[ scheme: <scheme> | default = http ]

# Optional HTTP URL parameters.
params:
  [ <string>: [<string>, ...] ]

# Sets the `Authorization` header on every scrape request with the
# configured username and password.
# password and password_file are mutually exclusive.
basic_auth:
  [ username: <string> ]
  [ password: <secret> ]
  [ password_file: <string> ]

# Sets the `Authorization` header on every scrape request with
# the configured bearer token. It is mutually exclusive with `bearer_token_file`.
[ bearer_token: <secret> ]

# Sets the `Authorization` header on every scrape request with the bearer token
# read from the configured file. It is mutually exclusive with `bearer_token`.
[ bearer_token_file: /path/to/bearer/token/file ]

# Configures the scrape request's TLS settings.
tls_config:
  [ <tls_config> ]

# Optional proxy URL.
[ proxy_url: <string> ]

# List of Azure service discovery configurations.
azure_sd_configs:
  [ - <azure_sd_config> ... ]

# List of Consul service discovery configurations.
consul_sd_configs:
  [ - <consul_sd_config> ... ]

# List of Digitalocean service discovery configurations.
digitalocean_sd_config:
  [ - <digitalocean_sd_config> ... ]

# List of Docker Swarm service discovery configurations.
dockerswarm_sd_configs:
  [ - <dockerswarm_sd_config> ... ]

# List of DNS service discovery configurations.
dns_sd_configs:
  [ - <dns_sd_config> ... ]

# List of EC2 service discovery configurations.
ec2_sd_configs:
  [ - <ec2_sd_config> ... ]

# List of OpenStack service discovery configurations.
openstack_sd_configs:
  [ - <openstack_sd_config> ... ]

# List of file service discovery configurations.
file_sd_configs:
  [ - <file_sd_config> ... ]

# List of GCE service discovery configurations.
gce_sd_configs:
  [ - <gce_sd_config> ... ]

# List of Hetzner service discovery configurations.
hetzner_sd_configs:
  [ - <hetzner_sd_config> ... ]

# List of Kubernetes service discovery configurations.
kubernetes_sd_configs:
  [ - <kubernetes_sd_config> ... ]

# List of Marathon service discovery configurations.
marathon_sd_configs:
  [ - <marathon_sd_config> ... ]

# List of AirBnB's Nerve service discovery configurations.
nerve_sd_configs:
  [ - <nerve_sd_config> ... ]

# List of Zookeeper Serverset service discovery configurations.
serverset_sd_configs:
  [ - <serverset_sd_config> ... ]

# List of Triton service discovery configurations.
triton_sd_configs:
  [ - <triton_sd_config> ... ]

# List of Eureka service discovery configurations.
eureka_sd_configs:
  [ - <eureka_sd_config> ... ]

# List of labeled statically configured targets for this job.
static_configs:
  [ - <static_config> ... ]

# List of target relabel configurations.
relabel_configs:
  [ - <relabel_config> ... ]

# List of metric relabel configurations.
metric_relabel_configs:
  [ - <relabel_config> ... ]

# Per-scrape limit on number of scraped samples that will be accepted.
# If more than this number of samples are present after metric relabelling
# the entire scrape will be treated as failed. 0 means no limit.
[ sample_limit: <int> | default = 0 ]

# Per-scrape config limit on number of unique targets that will be accepted.
# If more than this number of targets are present after target relabeling,
# Prometheus will mark the targets as failed without scraping them. 0 means
# no limit. This is an experimental feature of Prometheus and the behavior
# may change in the future.
[ target_limit: <int> | default = 0]
```

## azure_sd_config

Azure SD configurations allow retrieving scrape targets from Azure VMs.

The following meta labels are available on targets during relabeling:

* `__meta_azure_machine_id`: the machine ID
* `__meta_azure_machine_location`: the location the machine runs in
* `__meta_azure_machine_name`: the machine name
* `__meta_azure_machine_os_type`: the machine operating system
* `__meta_azure_machine_private_ip`: the machine's private IP
* `__meta_azure_machine_public_ip`: the machine's public IP if it exists
* `__meta_azure_machine_resource_group`: the machine's resource group
* `__meta_azure_machine_tag_<tagname>`: each tag value of the machine
* `__meta_azure_machine_scale_set`: the name of the scale set which the vm is part of (this value is only set if you are using a scale set)
* `__meta_azure_subscription_id`: the subscription ID
* `__meta_azure_tenant_id`: the tenant ID

See below for the configuration options for Azure discovery:

```yaml
# The information to access the Azure API.
# The Azure environment.
[ environment: <string> | default = AzurePublicCloud ]

# The authentication method, either OAuth or ManagedIdentity.
# See https://docs.microsoft.com/en-us/azure/active-directory/managed-identities-azure-resources/overview
[ authentication_method: <string> | default = OAuth]
# The subscription ID. Always required.
subscription_id: <string>
# Optional tenant ID. Only required with authentication_method OAuth.
[ tenant_id: <string> ]
# Optional client ID. Only required with authentication_method OAuth.
[ client_id: <string> ]
# Optional client secret. Only required with authentication_method OAuth.
[ client_secret: <secret> ]

# Refresh interval to re-read the instance list.
[ refresh_interval: <duration> | default = 300s ]

# The port to scrape metrics from. If using the public IP address, this must
# instead be specified in the relabeling rule.
[ port: <int> | default = 80 ]
```

## consul_sd_config

Consul SD configurations allow retrieving scrape targets from Consul's Catalog API.

The following meta labels are available on targets during relabeling:

* `__meta_consul_address`: the address of the target
* `__meta_consul_dc`: the datacenter name for the target
* `__meta_consul_tagged_address_<key>`: each node tagged address key value of the target
* `__meta_consul_metadata_<key>`: each node metadata key value of the target
* `__meta_consul_node`: the node name defined for the target
* `__meta_consul_service_address`: the service address of the target
* `__meta_consul_service_id`: the service ID of the target
* `__meta_consul_service_metadata_<key>`: each service metadata key value of the target
* `__meta_consul_service_port`: the service port of the target
* `__meta_consul_service`: the name of the service the target belongs to
* `__meta_consul_tags`: the list of tags of the target joined by the tag separator

```yaml
# The information to access the Consul API. It is to be defined
# as the Consul documentation requires.
[ server: <host> | default = "localhost:8500" ]
[ token: <secret> ]
[ datacenter: <string> ]
[ scheme: <string> | default = "http" ]
[ username: <string> ]
[ password: <secret> ]

tls_config:
  [ <tls_config> ]

# A list of services for which targets are retrieved. If omitted, all services
# are scraped.
services:
  [ - <string> ]

# See https://www.consul.io/api/catalog.html#list-nodes-for-service to know more
# about the possible filters that can be used.

# An optional list of tags used to filter nodes for a given service. Services must contain all tags in the list.
tags:
  [ - <string> ]

# Node metadata used to filter nodes for a given service.
[ node_meta:
  [ <name>: <value> ... ] ]

# The string by which Consul tags are joined into the tag label.
[ tag_separator: <string> | default = , ]

# Allow stale Consul results (see https://www.consul.io/api/features/consistency.html). Will reduce load on Consul.
[ allow_stale: <bool> ]

# The time after which the provided names are refreshed.
# On large setup it might be a good idea to increase this value because the catalog will change all the time.
[ refresh_interval: <duration> | default = 30s ]
```

## digitalocean_sd_config

DigitalOcean SD configurations allow retrieving scrape targets from
[DigitalOcean's](https://www.digitalocean.com/) Droplets API. This service
discovery uses the public IPv4 address by default, by that can be changed with
relabelling, as demonstrated in the [Prometheus digitalocean-sd configuration
file](https://github.com/prometheus/prometheus/blob/release-2.22/documentation/examples/prometheus-digitalocean.yml).

The following meta labels are available on targets during relabeling:

- `__meta_digitalocean_droplet_id`: the id of the droplet
- `__meta_digitalocean_droplet_name`: the name of the droplet
- `__meta_digitalocean_image`: the image name of the droplet
- `__meta_digitalocean_private_ipv4`: the private IPv4 of the droplet
- `__meta_digitalocean_public_ipv4`: the public IPv4 of the droplet
- `__meta_digitalocean_public_ipv6`: the public IPv6 of the droplet
- `__meta_digitalocean_region`: the region of the droplet
- `__meta_digitalocean_size`: the size of the droplet
- `__meta_digitalocean_status`: the status of the droplet
- `__meta_digitalocean_features`: the comma-separated list of features of the droplet
- `__meta_digitalocean_tags`: the comma-separated list of tags of the droplet

```yaml
# Authentication information used to authenticate to the API server.
# Note that `basic_auth`, `bearer_token` and `bearer_token_file` options are
# mutually exclusive.
# password and password_file are mutually exclusive.

# Optional HTTP basic authentication information, not currently supported by DigitalOcean.
basic_auth:
  [ username: <string> ]
  [ password: <secret> ]
  [ password_file: <string> ]

# Optional bearer token authentication information.
[ bearer_token: <secret> ]

# Optional bearer token file authentication information.
[ bearer_token_file: <filename> ]

# Optional proxy URL.
[ proxy_url: <string> ]

# TLS configuration.
tls_config:
  [ <tls_config> ]

# The port to scrape metrics from.
[ port: <int> | default = 80 ]

# The time after which the droplets are refreshed.
[ refresh_interval: <duration> | default = 60s ]
```

## dockerswarm_sd_config

Docker Swarm SD configurations allow retrieving scrape targets from [Docker
Swarm](https://docs.docker.com/engine/swarm/) engine.

One of the following roles can be configured to discover targets:

### services

The `services` role discovers all [Swarm
services](https://docs.docker.com/engine/swarm/key-concepts/#services-and-tasks)
and exposes their ports as targets. For each published port of a service, a
single target is generated. If a service has no published ports, a target per
service is created using the port parameter defined in the SD configuration.

Available meta labels:

- `__meta_dockerswarm_service_id`: the id of the service
- `__meta_dockerswarm_service_name`: the name of the service
- `__meta_dockerswarm_service_mode`: the mode of the service
- `__meta_dockerswarm_service_endpoint_port_name`: the name of the endpoint port, if available
- `__meta_dockerswarm_service_endpoint_port_publish_mode`: the publish mode of the endpoint port
- `__meta_dockerswarm_service_label_<labelname>`: each label of the service
- `__meta_dockerswarm_service_task_container_hostname`: the container hostname of the target, if available
- `__meta_dockerswarm_service_task_container_image`: the container image of the target
- `__meta_dockerswarm_service_updating_status`: the status of the service, if available
- `__meta_dockerswarm_network_id`: the ID of the network
- `__meta_dockerswarm_network_name`: the name of the network
- `__meta_dockerswarm_network_ingress`: whether the network is ingress
- `__meta_dockerswarm_network_internal`: whether the network is internal
- `__meta_dockerswarm_network_label_<labelname>`: each label of the network
- `__meta_dockerswarm_network_scope`: the scope of the network

### tasks

The `tasks` role discovers all [Swarm
tasks](https://docs.docker.com/engine/swarm/key-concepts/#services-and-tasks)
and exposes their ports as targets. For each published port of a task, a single
target is generated. If a task has no published ports, a target per task is
created using the `port` parameter defined in the SD configuration.

Available meta labels:

- `__meta_dockerswarm_task_id`: the id of the task
- `__meta_dockerswarm_task_container_id`: the container id of the task
- `__meta_dockerswarm_task_desired_state`: the desired state of the task
- `__meta_dockerswarm_task_label_<labelname>`: each label of the task
- `__meta_dockerswarm_task_slot`: the slot of the task
- `__meta_dockerswarm_task_state`: the state of the task
- `__meta_dockerswarm_task_port_publish_mode`: the publish mode of the task port
- `__meta_dockerswarm_service_id`: the id of the service
- `__meta_dockerswarm_service_name`: the name of the service
- `__meta_dockerswarm_service_mode`: the mode of the service
- `__meta_dockerswarm_service_label_<labelname>`: each label of the service
- `__meta_dockerswarm_network_id`: the ID of the network
- `__meta_dockerswarm_network_name`: the name of the network
- `__meta_dockerswarm_network_ingress`: whether the network is ingress
- `__meta_dockerswarm_network_internal`: whether the network is internal
- `__meta_dockerswarm_network_label_<labelname>`: each label of the network
- `__meta_dockerswarm_network_label`: each label of the network
- `__meta_dockerswarm_network_scope`: the scope of the network
- `__meta_dockerswarm_node_id`: the ID of the node
- `__meta_dockerswarm_node_hostname`: the hostname of the node
- `__meta_dockerswarm_node_address`: the address of the node
- `__meta_dockerswarm_node_availability`: the availability of the node
- `__meta_dockerswarm_node_label_<labelname>`: each label of the node
- `__meta_dockerswarm_node_platform_architecture`: the architecture of the node
- `__meta_dockerswarm_node_platform_os`: the operating system of the node
- `__meta_dockerswarm_node_role`: the role of the node
- `__meta_dockerswarm_node_status`: the status of the node

The `__meta_dockerswarm_network_*` meta labels are not populated for ports which are published with mode=host.

### nodes

The `nodes` role is used to discover [Swarm
nodes](https://docs.docker.com/engine/swarm/key-concepts/#nodes).

Available meta labels:

- `__meta_dockerswarm_node_address`: the address of the node
- `__meta_dockerswarm_node_availability`: the availability of the node
- `__meta_dockerswarm_node_engine_version`: the version of the node engine
- `__meta_dockerswarm_node_hostname`: the hostname of the node
- `__meta_dockerswarm_node_id`: the ID of the node
- `__meta_dockerswarm_node_label_<labelname>`: each label of the node
- `__meta_dockerswarm_node_manager_address`: the address of the manager component of the node
- `__meta_dockerswarm_node_manager_leader`: the leadership status of the manager component of the node (true or false)
- `__meta_dockerswarm_node_manager_reachability`: the reachability of the manager component of the node
- `__meta_dockerswarm_node_platform_architecture`: the architecture of the node
- `__meta_dockerswarm_node_platform_os`: the operating system of the node
- `__meta_dockerswarm_node_role`: the role of the node
- `__meta_dockerswarm_node_status`: the status of the node

See below for the configuration options for Docker Swarm discovery:

```yaml
# Address of the Docker daemon.
host: <string>

# Optional proxy URL.
[ proxy_url: <string> ]

# TLS configuration.
tls_config:
  [ <tls_config> ]

# Role of the targets to retrieve. Must be `services`, `tasks`, or `nodes`.
role: <string>

# The port to scrape metrics from, when `role` is nodes, and for discovered
# tasks and services that don't have published ports.
[ port: <int> | default = 80 ]

# The time after which the droplets are refreshed.
[ refresh_interval: <duration> | default = 60s ]

# Authentication information used to authenticate to the Docker daemon.
# Note that `basic_auth`, `bearer_token` and `bearer_token_file` options are
# mutually exclusive.
# password and password_file are mutually exclusive.

# Optional HTTP basic authentication information.
basic_auth:
  [ username: <string> ]
  [ password: <secret> ]
  [ password_file: <string> ]

# Optional bearer token authentication information.
[ bearer_token: <secret> ]

# Optional bearer token file authentication information.
[ bearer_token_file: <filename> ]
```

See [this example Prometheus configuration
file](https://github.com/prometheus/prometheus/blob/release-2.22/documentation/examples/prometheus-dockerswarm.yml)
for a detailed example of configuring Prometheus for Docker Swarm.

## dns_sd_config

A DNS-based service discovery configuration allows specifying a set of DNS
domain names which are periodically queried to discover a list of targets. The
DNS servers to be contacted are read from `/etc/resolv.conf`.

This service discovery method only supports basic DNS A, AAAA and SRV record
queries, but not the advanced DNS-SD approach specified in RFC6763.

During the relabeling phase, the meta label `__meta_dns_name` is available on
each target and is set to the record name that produced the discovered target.

```yaml
# A list of DNS domain names to be queried.
names:
  [ - <domain_name> ]

# The type of DNS query to perform.
[ type: <query_type> | default = 'SRV' ]

# The port number used if the query type is not SRV.
[ port: <number>]

# The time after which the provided names are refreshed.
[ refresh_interval: <duration> | default = 30s ]
```

## ec2_sd_config

EC2 SD configurations allow retrieving scrape targets from AWS EC2 instances.
The private IP address is used by default, but may be changed to the public IP
address with relabeling.

The following meta labels are available on targets during relabeling:

* `__meta_ec2_availability_zone`: the availability zone in which the instance is running
* `__meta_ec2_instance_id`: the EC2 instance ID
* `__meta_ec2_instance_state`: the state of the EC2 instance
* `__meta_ec2_instance_type`: the type of the EC2 instance
* `__meta_ec2_owner_id`: the ID of the AWS account that owns the EC2 instance
* `__meta_ec2_platform`: the Operating System platform, set to 'windows' on Windows servers, absent otherwise
* `__meta_ec2_primary_subnet_id`: the subnet ID of the primary network interface, if available
* `__meta_ec2_private_dns_name`: the private DNS name of the instance, if available
* `__meta_ec2_private_ip`: the private IP address of the instance, if present
* `__meta_ec2_public_dns_name`: the public DNS name of the instance, if available
* `__meta_ec2_public_ip`: the public IP address of the instance, if available
* `__meta_ec2_subnet_id`: comma separated list of subnets IDs in which the instance is running, if available
* `__meta_ec2_tag_<tagkey>`: each tag value of the instance
* `__meta_ec2_vpc_id`: the ID of the VPC in which the instance is running, if available

See below for the configuration options for EC2 discovery:

```yaml
# The information to access the EC2 API.

# The AWS region. If blank, the region from the instance metadata is used.
[ region: <string> ]

# Custom endpoint to be used.
[ endpoint: <string> ]

# The AWS API keys. If blank, the environment variables `AWS_ACCESS_KEY_ID`
# and `AWS_SECRET_ACCESS_KEY` are used.
[ access_key: <string> ]
[ secret_key: <secret> ]
# Named AWS profile used to connect to the API.
[ profile: <string> ]

# AWS Role ARN, an alternative to using AWS API keys.
[ role_arn: <string> ]

# Refresh interval to re-read the instance list.
[ refresh_interval: <duration> | default = 60s ]

# The port to scrape metrics from. If using the public IP address, this must
# instead be specified in the relabeling rule.
[ port: <int> | default = 80 ]

# Filters can be used optionally to filter the instance list by other criteria.
# Available filter criteria can be found here:
# https://docs.aws.amazon.com/AWSEC2/latest/APIReference/API_DescribeInstances.html
# Filter API documentation: https://docs.aws.amazon.com/AWSEC2/latest/APIReference/API_Filter.html
filters:
  [ - name: <string>
      values: <string>, [...] ]
```

## openstack_sd_config

OpenStack SD configurations allow retrieving scrape targets from OpenStack Nova
instances.

One of the following `<openstack_role>` types can be configured to discover
targets:

### hypervisor

The hypervisor role discovers one target per Nova hypervisor node. The target
address defaults to the host_ip attribute of the hypervisor.

The following meta labels are available on targets during relabeling:

* `__meta_openstack_hypervisor_host_ip`: the hypervisor node's IP address.
* `__meta_openstack_hypervisor_name`: the hypervisor node's name.
* `__meta_openstack_hypervisor_state`: the hypervisor node's state.
* `__meta_openstack_hypervisor_status`: the hypervisor node's status.
* `__meta_openstack_hypervisor_type`: the hypervisor node's type.

### instance

The instance role discovers one target per network interface of Nova instance.
The target address defaults to the private IP address of the network interface.

The following meta labels are available on targets during relabeling:

* `__meta_openstack_address_pool`: the pool of the private IP.
* `__meta_openstack_instance_flavor`: the flavor of the OpenStack instance.
* `__meta_openstack_instance_id`: the OpenStack instance ID.
* `__meta_openstack_instance_name`: the OpenStack instance name.
* `__meta_openstack_instance_status`: the status of the OpenStack instance.
* `__meta_openstack_private_ip`: the private IP of the OpenStack instance.
* `__meta_openstack_project_id`: the project (tenant) owning this instance.
* `__meta_openstack_public_ip`: the public IP of the OpenStack instance.
* `__meta_openstack_tag_<tagkey>`: each tag value of the instance.
* `__meta_openstack_user_id`: the user account owning the tenant.

See below for the configuration options for OpenStack discovery:

```yaml
# The information to access the OpenStack API.

# The OpenStack role of entities that should be discovered.
role: <openstack_role>

# The OpenStack Region.
region: <string>

# identity_endpoint specifies the HTTP endpoint that is required to work with
# the Identity API of the appropriate version. While it's ultimately needed by
# all of the identity services, it will often be populated by a provider-level
# function.
[ identity_endpoint: <string> ]

# username is required if using Identity V2 API. Consult with your provider's
# control panel to discover your account's username. In Identity V3, either
# userid or a combination of username and domain_id or domain_name are needed.
[ username: <string> ]
[ userid: <string> ]

# password for the Identity V2 and V3 APIs. Consult with your provider's
# control panel to discover your account's preferred method of authentication.
[ password: <secret> ]

# At most one of domain_id and domain_name must be provided if using username
# with Identity V3. Otherwise, either are optional.
[ domain_name: <string> ]
[ domain_id: <string> ]

# The project_id and project_name fields are optional for the Identity V2 API.
# Some providers allow you to specify a project_name instead of the project_id.
# Some require both. Your provider's authentication policies will determine
# how these fields influence authentication.
[ project_name: <string> ]
[ project_id: <string> ]

# The application_credential_id or application_credential_name fields are
# required if using an application credential to authenticate. Some providers
# allow you to create an application credential to authenticate rather than a
# password.
[ application_credential_name: <string> ]
[ application_credential_id: <string> ]

# The application_credential_secret field is required if using an application
# credential to authenticate.
[ application_credential_secret: <secret> ]

# Whether the service discovery should list all instances for all projects.
# It is only relevant for the 'instance' role and usually requires admin permissions.
[ all_tenants: <boolean> | default: false ]

# Refresh interval to re-read the instance list.
[ refresh_interval: <duration> | default = 60s ]

# The port to scrape metrics from. If using the public IP address, this must
# instead be specified in the relabeling rule.
[ port: <int> | default = 80 ]

# TLS configuration.
tls_config:
  [ <tls_config> ]
```

## tls_config

A `tls_config` allows configuring TLS connections.

```yaml
# CA certificate to validate API server certificate with.
[ ca_file: <filename> ]

# Certificate and key files for client cert authentication to the server.
[ cert_file: <filename> ]
[ key_file: <filename> ]

# ServerName extension to indicate the name of the server.
# https://tools.ietf.org/html/rfc4366#section-3.1
[ server_name: <string> ]

# Disable validation of the server certificate.
[ insecure_skip_verify: <boolean> ]
```

## file_sd_config

File-based service discovery provides a more generic way to configure static
targets and serves as an interface to plug in custom service discovery
mechanisms.

It reads a set of files containing a list of zero or more `<static_config>`s.
Changes to all defined files are detected via disk watches and applied
immediately. Files may be provided in YAML or JSON format. Only changes
resulting in well-formed target groups are applied.

The JSON file must contain a list of static configs, using this format:

```json
[
  {
    "targets": [ "<host>", ... ],
    "labels": {
      "<labelname>": "<labelvalue>", ...
    }
  },
  ...
]
```

As a fallback, the file contents are also re-read periodically at the specified
refresh interval.

Each target has a meta label `__meta_filepath` during the relabeling phase. Its
value is set to the filepath from which the target was extracted.

There is a list of
[integrations](https://prometheus.io/docs/operating/integrations/#file-service-discovery)
with this discovery mechanism.

```yaml
# Patterns for files from which target groups are extracted.
files:
  [ - <filename_pattern> ... ]

# Refresh interval to re-read the files.
[ refresh_interval: <duration> | default = 5m ]
```

Where `<filename_pattern>` may be a path ending in `.json`, `.yml` or `.yaml`. The
last path segment may contain a single `*` that matches any character sequence,
e.g. `my/path/tg_*.json`.

## gce_sd_config

GCE SD configurations allow retrieving scrape targets from GCP GCE instances.
The private IP address is used by default, but may be changed to the public IP
address with relabeling.

The following meta labels are available on targets during relabeling:

* `__meta_gce_instance_id`: the numeric id of the instance
* `__meta_gce_instance_name`: the name of the instance
* `__meta_gce_label_<name>`: each GCE label of the instance
* `__meta_gce_machine_type`: full or partial URL of the machine type of the instance
* `__meta_gce_metadata_<name>`: each metadata item of the instance
* `__meta_gce_network`: the network URL of the instance
* `__meta_gce_private_ip`: the private IP address of the instance
* `__meta_gce_project`: the GCP project in which the instance is running
* `__meta_gce_public_ip`: the public IP address of the instance, if present
* `__meta_gce_subnetwork`: the subnetwork URL of the instance
* `__meta_gce_tags`: comma separated list of instance tags
* `__meta_gce_zone`: the GCE zone URL in which the instance is running

See below for the configuration options for GCE discovery:

```yaml
# The information to access the GCE API.

# The GCP Project
project: <string>

# The zone of the scrape targets. If you need multiple zones use multiple
# gce_sd_configs.
zone: <string>

# Filter can be used optionally to filter the instance list by other criteria
# Syntax of this filter string is described here in the filter query parameter section:
# https://cloud.google.com/compute/docs/reference/latest/instances/list
[ filter: <string> ]

# Refresh interval to re-read the instance list
[ refresh_interval: <duration> | default = 60s ]

# The port to scrape metrics from. If using the public IP address, this must
# instead be specified in the relabeling rule.
[ port: <int> | default = 80 ]

# The tag separator is used to separate the tags on concatenation
[ tag_separator: <string> | default = , ]
```

Credentials are discovered by the Google Cloud SDK default client by looking in the following places, preferring the first location found:

1. a JSON file specified by the `GOOGLE_APPLICATION_CREDENTIALS` environment
   variable
2. a JSON file in the well-known path
   `$HOME/.config/gcloud/application_default_credentials.json`
3. fetched from the GCE metadata server

If Prometheus is running within GCE, the service account associated with the
instance it is running on should have at least read-only permissions to the
compute resources. If running outside of GCE make sure to create an appropriate
service account and place the credential file in one of the expected locations.

## hetzner_sd_config

Hetzner SD configurations allow retrieving scrape targets from [Hetzner
Cloud](https://www.hetzner.com/) API and
[Robot](https://docs.hetzner.com/robot/) API. This service discovery uses the
public IPv4 address by default, but that can be changed with relabeling, as
demonstrated in the [Prometheus hetzner-sd configuration
file](https://github.com/prometheus/prometheus/blob/release-2.22/documentation/examples/prometheus-hetzner.yml).

The following meta labels are available on all targets during relabeling:

- `__meta_hetzner_server_id`: the ID of the server
- `__meta_hetzner_server_name`: the name of the server
- `__meta_hetzner_server_status`: the status of the server
- `__meta_hetzner_public_ipv4`: the public ipv4 address of the server
- `__meta_hetzner_public_ipv6_network`: the public ipv6 network (/64) of the server
- `__meta_hetzner_datacenter`: the datacenter of the server

The labels below are only available for targets with `role` set to `hcloud`:

- `__meta_hetzner_hcloud_image_name`: the image name of the server
- `__meta_hetzner_hcloud_image_description`: the description of the server image
- `__meta_hetzner_hcloud_image_os_flavor`: the OS flavor of the server image
- `__meta_hetzner_hcloud_image_os_version`: the OS version of the server image
- `__meta_hetzner_hcloud_image_description`: the description of the server image
- `__meta_hetzner_hcloud_datacenter_location`: the location of the server
- `__meta_hetzner_hcloud_datacenter_location_network_zone`: the network zone of the server
- `__meta_hetzner_hcloud_server_type`: the type of the server
- `__meta_hetzner_hcloud_cpu_cores`: the CPU cores count of the server
- `__meta_hetzner_hcloud_cpu_type`: the CPU type of the server (shared or dedicated)
- `__meta_hetzner_hcloud_memory_size_gb`: the amount of memory of the server (in GB)
- `__meta_hetzner_hcloud_disk_size_gb`: the disk size of the server (in GB)
- `__meta_hetzner_hcloud_private_ipv4_<networkname>`: the private ipv4 address of the server within a given network
- `__meta_hetzner_hcloud_label_<labelname>`: each label of the server

The labels below are only available for targets with `role` set to `robot`:

- `__meta_hetzner_robot_product`: the product of the server
- `__meta_hetzner_robot_cancelled`: the server cancellation status

```yaml
# The Hetzner role of entities that should be discovered.
# One of robot or hcloud.
role: <string>

# Authentication information used to authenticate to the API server.
# Note that `basic_auth`, `bearer_token` and `bearer_token_file` options are
# mutually exclusive.
# password and password_file are mutually exclusive.

# Optional HTTP basic authentication information, required when role is robot
# Role hcloud does not support basic auth.
basic_auth:
  [ username: <string> ]
  [ password: <secret> ]
  [ password_file: <string> ]

# Optional bearer token authentication information, required when role is hcloud
# Role robot does not support bearer token authentication.
[ bearer_token: <secret> ]

# Optional bearer token file authentication information.
[ bearer_token_file: <filename> ]

# Optional proxy URL.
[ proxy_url: <string> ]

# TLS configuration.
tls_config:
  [ <tls_config> ]

# The port to scrape metrics from.
[ port: <int> | default = 80 ]

# The time after which the servers are refreshed.
[ refresh_interval: <duration> | default = 60s ]
```

## kubernetes_sd_config

Kubernetes SD configurations allow retrieving scrape targets from Kubernetes'
REST API and always staying synchronized with the cluster state.

One of the following role types can be configured to discover targets:

### node

The `node` role discovers one target per cluster node with the address defaulting
to the Kubelet's HTTP port. The target address defaults to the first existing
address of the Kubernetes node object in the address type order of
`NodeInternalIP`, `NodeExternalIP`, `NodeLegacyHostIP`, and `NodeHostName`.

Available meta labels:

* `__meta_kubernetes_node_name`: The name of the node object.
* `__meta_kubernetes_node_label_<labelname>`: Each label from the node object.
* `__meta_kubernetes_node_labelpresent_<labelname>`: true for each label from the node object.
* `__meta_kubernetes_node_annotation_<annotationname>`: Each annotation from the node object.
* `__meta_kubernetes_node_annotationpresent_<annotationname>`: true for each annotation from the node object.
* `__meta_kubernetes_node_address_<address_type>`: The first address for each node address type, if it exists.

In addition, the `instance` label for the node will be set to the node name as
retrieved from the API server.

### service

The `service` role discovers a target for each service port for each service. This
is generally useful for blackbox monitoring of a service. The address will be
set to the Kubernetes DNS name of the service and respective service port.

Available meta labels:

* `__meta_kubernetes_namespace`: The namespace of the service object.
* `__meta_kubernetes_service_annotation_<annotationname>`: Each annotation from the service object.
* `__meta_kubernetes_service_annotationpresent_<annotationname>`: "true" for each annotation of the service object.
* `__meta_kubernetes_service_cluster_ip`: The cluster IP address of the service. (Does not apply to services of type ExternalName)
* `__meta_kubernetes_service_external_name`: The DNS name of the service. (Applies to services of type ExternalName)
* `__meta_kubernetes_service_label_<labelname>`: Each label from the service object.
* `__meta_kubernetes_service_labelpresent_<labelname>`: true for each label of the service object.
* `__meta_kubernetes_service_name`: The name of the service object.
* `__meta_kubernetes_service_port_name`: Name of the service port for the target.
* `__meta_kubernetes_service_port_protocol`: Protocol of the service port for the target.

### pod

The `pod` role discovers all pods and exposes their containers as targets. For
each declared port of a container, a single target is generated. If a container
has no specified ports, a port-free target per container is created for manually
adding a port via relabeling.

Available meta labels:

* `__meta_kubernetes_namespace`: The namespace of the pod object.
* `__meta_kubernetes_pod_name`: The name of the pod object.
* `__meta_kubernetes_pod_ip`: The pod IP of the pod object.
* `__meta_kubernetes_pod_label_<labelname>`: Each label from the pod object.
* `__meta_kubernetes_pod_labelpresent_<labelname>`: truefor each label from the pod object.
* `__meta_kubernetes_pod_annotation_<annotationname>`: Each annotation from the pod object.
* `__meta_kubernetes_pod_annotationpresent_<annotationname>`: true for each annotation from the pod object.
* `__meta_kubernetes_pod_container_init`: true if the container is an InitContainer
* `__meta_kubernetes_pod_container_name`: Name of the container the target address points to.
* `__meta_kubernetes_pod_container_port_name`: Name of the container port.
* `__meta_kubernetes_pod_container_port_number`: Number of the container port.
* `__meta_kubernetes_pod_container_port_protocol`: Protocol of the container port.
* `__meta_kubernetes_pod_ready`: Set to true or false for the pod's ready state.
* `__meta_kubernetes_pod_phase`: Set to Pending, Running, Succeeded, Failed or Unknown in the lifecycle.
* `__meta_kubernetes_pod_node_name`: The name of the node the pod is scheduled onto.
* `__meta_kubernetes_pod_host_ip`: The current host IP of the pod object.
* `__meta_kubernetes_pod_uid`: The UID of the pod object.
* `__meta_kubernetes_pod_controller_kind`: Object kind of the pod controller.
* `__meta_kubernetes_pod_controller_name`: Name of the pod controller.

### endpoints

The `endpoints` role discovers targets from listed endpoints of a service. For
each endpoint address one target is discovered per port. If the endpoint is
backed by a pod, all additional container ports of the pod, not bound to an
endpoint port, are discovered as targets as well.

Available meta labels:

* `__meta_kubernetes_namespace`: The namespace of the endpoints object.
* `__meta_kubernetes_endpoints_name`: The names of the endpoints object.
* For all targets discovered directly from the endpoints list (those not
  additionally inferred from underlying pods), the following labels are
  attached:
  * `__meta_kubernetes_endpoint_hostname`: Hostname of the endpoint.
  * `__meta_kubernetes_endpoint_node_name`: Name of the node hosting the endpoint.
  * `__meta_kubernetes_endpoint_ready`: Set to true or false for the endpoint's
    ready state.
  * `__meta_kubernetes_endpoint_port_name`: Name of the endpoint port.
  * `__meta_kubernetes_endpoint_port_protocol`: Protocol of the endpoint port.
  * `__meta_kubernetes_endpoint_address_target_kind`: Kind of the endpoint address target.
  * `__meta_kubernetes_endpoint_address_target_name`: Name of the endpoint address target.
* If the endpoints belong to a service, all labels of the `role: service`
  discovery are attached.
* For all targets backed by a pod, all labels of the role: `pod discovery` are
  attached.

### ingress

The `ingress` role discovers a target for each path of each ingress. This is
generally useful for blackbox monitoring of an ingress. The address will be set
to the host specified in the ingress spec.

Available meta labels:

* `__meta_kubernetes_namespace`: The namespace of the ingress object.
* `__meta_kubernetes_ingress_name`: The name of the ingress object.
* `__meta_kubernetes_ingress_label_<labelname>`: Each label from the ingress object.
* `__meta_kubernetes_ingress_labelpresent_<labelname>`: true for each label from the ingress object.
* `__meta_kubernetes_ingress_annotation_<annotationname>`: Each annotation from the ingress object.
* `__meta_kubernetes_ingress_annotationpresent_<annotationname>`: true for each annotation from the ingress object.
* `__meta_kubernetes_ingress_scheme`: Protocol scheme of ingress, `https` if TLS config is set. Defaults to `http`.
* `__meta_kubernetes_ingress_path`: Path from ingress spec. Defaults to /.

See below for the configuration options for Kubernetes discovery:

```yaml
# The information to access the Kubernetes API.

# The API server addresses. If left empty, Prometheus is assumed to run inside
# of the cluster and will discover API servers automatically and use the pod's
# CA certificate and bearer token file at /var/run/secrets/kubernetes.io/serviceaccount/.
[ api_server: <host> ]

# The Kubernetes role of entities that should be discovered.
role: <role>

# Optional authentication information used to authenticate to the API server.
# Note that `basic_auth`, `bearer_token` and `bearer_token_file` options are
# mutually exclusive.
# password and password_file are mutually exclusive.

# Optional HTTP basic authentication information.
basic_auth:
  [ username: <string> ]
  [ password: <secret> ]
  [ password_file: <string> ]

# Optional bearer token authentication information.
[ bearer_token: <secret> ]

# Optional bearer token file authentication information.
[ bearer_token_file: <filename> ]

# Optional proxy URL.
[ proxy_url: <string> ]

# TLS configuration.
tls_config:
  [ <tls_config> ]

# Optional namespace discovery. If omitted, all namespaces are used.
namespaces:
  names:
    [ - <string> ]
```

Where `<role>` must be `endpoints`, `service`, `pod`, `node`, or `ingress`.

## marathon_sd_config

Marathon SD configurations allow retrieving scrape targets using the Marathon
REST API. Prometheus will periodically check the REST endpoint for currently
running tasks and create a target group for every app that has at least one
healthy task.

The following meta labels are available on targets during relabeling:

* `__meta_marathon_app`: the name of the app (with slashes replaced by dashes)
* `__meta_marathon_image`: the name of the Docker image used (if available)
* `__meta_marathon_task`: the ID of the Mesos task
* `__meta_marathon_app_label_<labelname>`: any Marathon labels attached to the app
* `__meta_marathon_port_definition_label_<labelname>`: the port definition labels
* `__meta_marathon_port_mapping_label_<labelname>`: the port mapping labels
* `__meta_marathon_port_index`: the port index number (e.g. 1 for PORT1)

See below for the configuration options for Marathon discovery:

```yaml
# List of URLs to be used to contact Marathon servers.
# You need to provide at least one server URL.
servers:
  - <string>

# Polling interval
[ refresh_interval: <duration> | default = 30s ]

# Optional authentication information for token-based authentication
# https://docs.mesosphere.com/1.11/security/ent/iam-api/#passing-an-authentication-token
# It is mutually exclusive with `auth_token_file` and other authentication mechanisms.
[ auth_token: <secret> ]

# Optional authentication information for token-based authentication
# https://docs.mesosphere.com/1.11/security/ent/iam-api/#passing-an-authentication-token
# It is mutually exclusive with `auth_token` and other authentication mechanisms.
[ auth_token_file: <filename> ]

# Sets the `Authorization` header on every request with the
# configured username and password.
# This is mutually exclusive with other authentication mechanisms.
# password and password_file are mutually exclusive.
basic_auth:
  [ username: <string> ]
  [ password: <string> ]
  [ password_file: <string> ]

# Sets the `Authorization` header on every request with
# the configured bearer token. It is mutually exclusive with `bearer_token_file` and other authentication mechanisms.
# NOTE: The current version of DC/OS marathon (v1.11.0) does not support standard Bearer token authentication. Use `auth_token` instead.
[ bearer_token: <string> ]

# Sets the `Authorization` header on every request with the bearer token
# read from the configured file. It is mutually exclusive with `bearer_token` and other authentication mechanisms.
# NOTE: The current version of DC/OS marathon (v1.11.0) does not support standard Bearer token authentication. Use `auth_token_file` instead.
[ bearer_token_file: /path/to/bearer/token/file ]

# TLS configuration for connecting to marathon servers
tls_config:
  [ <tls_config> ]

# Optional proxy URL.
[ proxy_url: <string> ]
```

By default every app listed in Marathon will be scraped by Prometheus. If not
all of your services provide Prometheus metrics, you can use a Marathon label
and Prometheus relabeling to control which instances will actually be scraped.

By default, all apps will show up as a single job in Prometheus (the one
specified in the configuration file), which can also be changed using
relabeling.

## nerve_sd_config

Nerve SD configurations allow retrieving scrape targets from AirBnB's Nerve
which are stored in Zookeeper.

The following meta labels are available on targets during relabeling:

* `__meta_nerve_path`: the full path to the endpoint node in Zookeeper
* `__meta_nerve_endpoint_host`: the host of the endpoint
* `__meta_nerve_endpoint_port`: the port of the endpoint
* `__meta_nerve_endpoint_name`: the name of the endpoint

```yaml
# The Zookeeper servers.
servers:
  - <host>
# Paths can point to a single service, or the root of a tree of services.
paths:
  - <string>
[ timeout: <duration> | default = 10s ]
```

## serverset_sd_config

Serverset SD configurations allow retrieving scrape targets from Serversets
which are stored in Zookeeper. Serversets are commonly used by Finagle and
Aurora.

The following meta labels are available on targets during relabeling:

* `__meta_serverset_path`: the full path to the serverset member node in Zookeeper
* `__meta_serverset_endpoint_host`: the host of the default endpoint
* `__meta_serverset_endpoint_port`: the port of the default endpoint
* `__meta_serverset_endpoint_host_<endpoint>`: the host of the given endpoint
* `__meta_serverset_endpoint_port_<endpoint>`: the port of the given endpoint
* `__meta_serverset_shard`: the shard number of the member
* `__meta_serverset_status`: the status of the member

```yaml
# The Zookeeper servers.
servers:
  - <host>
# Paths can point to a single serverset, or the root of a tree of serversets.
paths:
  - <string>
[ timeout: <duration> | default = 10s ]
```

## triton_sd_config

Triton SD configurations allow retrieving scrape targets from Container Monitor
discovery endpoints.

The following meta labels are available on targets during relabeling:

* `__meta_triton_groups`: the list of groups belonging to the target joined by a comma separator
* `__meta_triton_machine_alias`: the alias of the target container
* `__meta_triton_machine_brand`: the brand of the target container
* `__meta_triton_machine_id`: the UUID of the target container
* `__meta_triton_machine_image`: the target containers image type
* `__meta_triton_server_id`: the server UUID for the target container

```yaml
# The information to access the Triton discovery API.

# The account to use for discovering new target containers.
account: <string>

# The DNS suffix which should be applied to target containers.
dns_suffix: <string>

# The Triton discovery endpoint (e.g. 'cmon.us-east-3b.triton.zone'). This is
# often the same value as dns_suffix.
endpoint: <string>

# A list of groups for which targets are retrieved. If omitted, all containers
# available to the requesting account are scraped.
groups:
  [ - <string> ... ]

# The port to use for discovery and metric scraping.
[ port: <int> | default = 9163 ]

# The interval which should be used for refreshing target containers.
[ refresh_interval: <duration> | default = 60s ]

# The Triton discovery API version.
[ version: <int> | default = 1 ]

# TLS configuration.
tls_config:
  [ <tls_config> ]
```

## eureka_sd_config

Eureka SD configurations allow retrieving scrape targets using the
[Eureka](https://github.com/Netflix/eureka) REST API. Prometheus will
periodically check the REST endpoint and create a target for every app instance.

The following meta labels are available on targets during relabeling:

- `__meta_eureka_app_name`: the name of the app
- `__meta_eureka_app_instance_id`: the ID of the app instance
- `__meta_eureka_app_instance_hostname`: the hostname of the instance
- `__meta_eureka_app_instance_homepage_url`: the homepage url of the app instance
- `__meta_eureka_app_instance_statuspage_url`: the status page url of the app instance
- `__meta_eureka_app_instance_healthcheck_url`: the health check url of the app instance
- `__meta_eureka_app_instance_ip_addr`: the IP address of the app instance
- `__meta_eureka_app_instance_vip_address`: the VIP address of the app instance
- `__meta_eureka_app_instance_secure_vip_address`: the secure VIP address of the app instance
- `__meta_eureka_app_instance_status`: the status of the app instance
- `__meta_eureka_app_instance_port`: the port of the app instance
- `__meta_eureka_app_instance_port_enabled`: the port enabled of the app instance
- `__meta_eureka_app_instance_secure_port`: the secure port address of the app instance
- `__meta_eureka_app_instance_secure_port_enabled`: the secure port of the app instance
- `__meta_eureka_app_instance_country_id`: the country ID of the app instance
- `__meta_eureka_app_instance_metadata_<metadataname>`: app instance metadata
- `__meta_eureka_app_instance_datacenterinfo_name`: the datacenter name of the app instance
- `__meta_eureka_app_instance_datacenterinfo_<metadataname>`: the datacenter metadata

See below for the configuration options for Eureka discovery:

```yaml
# The URL to connect to the Eureka server.
server: <string>

# Sets the `Authorization` header on every request with the
# configured username and password.
# password and password_file are mutually exclusive.
basic_auth:
  [ username: <string> ]
  [ password: <secret> ]
  [ password_file: <string> ]

# Sets the `Authorization` header on every request with
# the configured bearer token. It is mutually exclusive with `bearer_token_file`.
[ bearer_token: <string> ]

# Sets the `Authorization` header on every request with the bearer token
# read from the configured file. It is mutually exclusive with `bearer_token`.
[ bearer_token_file: <filename> ]

# Configures the scrape request's TLS settings.
tls_config:
  [ <tls_config> ]

# Optional proxy URL.
[ proxy_url: <string> ]

# Refresh interval to re-read the app instance list.
[ refresh_interval: <duration> | default = 30s ]
```

See [the Prometheus eureka-sd configuration
file](https://github.com/prometheus/prometheus/blob/release-2.22/documentation/examples/prometheus-eureka.yml)
for a practical example on how to set up your Eureka app and your Prometheus
configuration.

## static_config

A `static_config` allows specifying a list of targets and a common label set for
them. It is the canonical way to specify static targets in a scrape
configuration.

```yaml
# The targets specified by the static config.
targets:
  [ - '<host>' ]

# Labels assigned to all metrics scraped from the targets.
labels:
  [ <labelname>: <labelvalue> ... ]
```

## relabel_config

Relabeling is a powerful tool to dynamically rewrite the label set of a target
before it gets scraped. Multiple relabeling steps can be configured per scrape
configuration. They are applied to the label set of each target in order of
their appearance in the configuration file.

Initially, aside from the configured per-target labels, a target's job label is
set to the `job_name` value of the respective scrape configuration. The
__address__ label is set to the `<host>:<port>` address of the target. After
relabeling, the `instance` label is set to the value of `__address__` by default if
it was not set during relabeling. The `__scheme__` and `__metrics_path__` labels are
set to the scheme and metrics path of the target respectively. The
`__param_<name>` label is set to the value of the first passed URL parameter
called `<name>`.

Additional labels prefixed with `__meta_` may be available during the relabeling
phase. They are set by the service discovery mechanism that provided the target
and vary between mechanisms.

Labels starting with `__` will be removed from the label set after target
relabeling is completed.

If a relabeling step needs to store a label value only temporarily (as the input
to a subsequent relabeling step), use the `__tmp` label name prefix. This prefix
is guaranteed to never be used by Prometheus itself.

```yaml
# The source labels select values from existing labels. Their content is concatenated
# using the configured separator and matched against the configured regular expression
# for the replace, keep, and drop actions.
[ source_labels: '[' <labelname> [, ...] ']' ]

# Separator placed between concatenated source label values.
[ separator: <string> | default = ; ]

# Label to which the resulting value is written in a replace action.
# It is mandatory for replace actions. Regex capture groups are available.
[ target_label: <labelname> ]

# Regular expression against which the extracted value is matched.
[ regex: <regex> | default = (.*) ]

# Modulus to take of the hash of the source label values.
[ modulus: <uint64> ]

# Replacement value against which a regex replace is performed if the
# regular expression matches. Regex capture groups are available.
[ replacement: <string> | default = $1 ]

# Action to perform based on regex matching.
[ action: <relabel_action> | default = replace ]
```

`<regex>` is any valid RE2 regular expression. It is required for the `replace`,
`keep`, `drop`, `labelmap`, `labeldrop` and `labelkeep` actions. The regex is
anchored on both ends. To un-anchor the regex, use `.*<regex>.*`.

`<relabel_action>` determines the relabeling action to take:

* `replace`: Match regex against the concatenated source_labels. Then, set
  target_label to replacement, with match group references (${1}, ${2}, ...) in
  replacement substituted by their value. If regex does not match, no
  replacement takes place.
* `keep`: Drop targets for which regex does not match the concatenated
  source_labels.
* `drop`: Drop targets for which regex matches the concatenated source_labels.
* `hashmod`: Set target_label to the modulus of a hash of the concatenated
  source_labels.
* `labelmap`: Match regex against all label names. Then copy the values of the
  matching labels to label names given by replacement with match group
  references (${1}, ${2}, ...) in replacement substituted by their value.
* `labeldrop`: Match regex against all label names. Any label that matches will
  be removed from the set of labels.
* `labelkeep`: Match regex against all label names. Any label that does not
  match will be removed from the set of labels.

Care must be taken with `labeldrop` and `labelkeep` to ensure that metrics are still uniquely labeled once the labels are removed.

### remote_write

`write_relabel_configs` is relabeling applied to samples before sending them to
the remote endpoint. Write relabeling is applied after external labels. This
could be used to limit which samples are sent.

```yaml
# The URL of the endpoint to send samples to.
url: <string>

# Timeout for requests to the remote write endpoint.
[ remote_timeout: <duration> | default = 30s ]

# Custom HTTP headers to be sent along with each remote write request.
# Be aware that headers that are set by Prometheus itself can't be overwritten.
headers:
  [ <string>: <string> ... ]

# List of remote write relabel configurations.
write_relabel_configs:
  [ - <relabel_config> ... ]

# Enables sending of exemplars over remote write.
[ send_exemplars: <boolean> | default = false ]

# Sets the `Authorization` header on every remote write request with the
# configured username and password.
# password and password_file are mutually exclusive.
basic_auth:
  [ username: <string> ]
  [ password: <string> ]
  [ password_file: <string> ]

# Sets the `Authorization` header on every remote write request with
# the configured bearer token. It is mutually exclusive with `bearer_token_file`.
[ bearer_token: <string> ]

# Sets the `Authorization` header on every remote write request with the bearer token
# read from the configured file. It is mutually exclusive with `bearer_token`.
[ bearer_token_file: /path/to/bearer/token/file ]

# Optionally configures AWS's Signature Verification 4 signing process to
# sign requests. Cannot be set at the same time as basic_auth or authorization.
# To use the default credentials from the AWS SDK, use `sigv4: {}`.
sigv4:
  # The AWS region. If blank, the region from the default credentials chain
  # is used.
  [ region: <string> ]

  # The AWS API keys. If blank, the environment variables `AWS_ACCESS_KEY_ID`
  # and `AWS_SECRET_ACCESS_KEY` are used.
  [ access_key: <string> ]
  [ secret_key: <secret> ]

  # Named AWS profile used to authenticate.
  [ profile: <string> ]

  # AWS Role ARN, an alternative to using AWS API keys.
  [ role_arn: <string> ]

# Configures the remote write request's TLS settings.
tls_config:
  [ <tls_config> ]

# Optional proxy URL.
[ proxy_url: <string> ]

# Configures the queue used to write to remote storage.
queue_config:
  # Number of samples to buffer per shard before we block reading of more
  # samples from the WAL. It is recommended to have enough capacity in each
  # shard to buffer several requests to keep throughput up while processing
  # occasional slow remote requests.
  [ capacity: <int> | default = 2500 ]
  # Maximum number of shards, i.e. amount of concurrency.
  [ max_shards: <int> | default = 200 ]
  # Minimum number of shards, i.e. amount of concurrency.
  [ min_shards: <int> | default = 1 ]
  # Maximum number of samples per send.
  [ max_samples_per_send: <int> | default = 500 ]
  # Maximum time a sample will wait in buffer.
  [ batch_send_deadline: <duration> | default = 5s ]
  # Initial retry delay. Gets doubled for every retry.
  [ min_backoff: <duration> | default = 30ms ]
  # Maximum retry delay.
  [ max_backoff: <duration> | default = 100ms ]

# Configures the sending of series metadata to remote storage.
# It is experimental and subject to change at any point.
metadata_config:
  # Whether metric metadata is sent to remote storage or not.
  [ send: <boolean> | default = true ]
  # How frequently metric metadata is sent to remote storage.
  [ send_interval: <duration> | default = 1m ]
```