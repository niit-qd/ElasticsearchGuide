# [Discovery](https://www.elastic.co/guide/en/elasticsearch/reference/current/discovery-hosts-providers.html)

Discovery is the process by which the cluster formation module finds other nodes with which to form a cluster. This process runs when you start an Elasticsearch node or when a node believes the master node failed and continues until the master node is found or a new master node is elected.
发现是集群形成模块寻找其他节点以形成集群的过程。当您启动 Elasticsearch 节点或节点认为主节点发生故障时，此过程将运行，并持续到找到主节点或选出新的主节点为止。

This process starts with a list of *seed* addresses from one or more [seed hosts providers](https://www.elastic.co/guide/en/elasticsearch/reference/current/discovery-hosts-providers.html#built-in-hosts-providers), together with the addresses of any master-eligible nodes that were in the last-known cluster. The process operates in two phases: First, each node probes the seed addresses by connecting to each address and attempting to identify the node to which it is connected and to verify that it is master-eligible. Secondly, if successful, it shares with the remote node a list of all of its known master-eligible peers and the remote node responds with *its* peers in turn. The node then probes all the new nodes that it just discovered, requests their peers, and so on.
此过程从一个或多个[seed hosts providers](https://www.elastic.co/guide/en/elasticsearch/reference/current/discovery-hosts-providers.html#built-in-hosts-providers) 的 *seed* 地址列表开始，同时包含最近已知集群中所有符合主节点条件的节点的地址。此过程分为两个阶段：首先，每个节点通过连接到每个地址并尝试识别其所连接的节点以及验证其是否符合主节点条件来探测种子地址。其次，如果成功，它将与远程节点共享其所有已知符合主节点条件的对等节点的列表，然后远程节点依次响应*其*对等节点。然后，节点探测其刚刚发现的所有新节点，请求其对等节点，依此类推。

If the node is not master-eligible then it continues this discovery process until it has discovered an elected master node. If no elected master is discovered then the node will retry after `discovery.find_peers_interval` which defaults to `1s`.
如果节点不符合主节点条件，则它将继续此发现过程，直到发现选举的主节点。如果没有发现选举的主节点，则节点将在`discovery.find_peers_interval`（默认值为`1s`）之后重试。

Once a master is elected, it will normally remain as the elected master until it is deliberately stopped. It may also stop acting as the master if [fault detection](https://www.elastic.co/guide/en/elasticsearch/reference/current/cluster-fault-detection.html) determines the cluster to be faulty. When a node stops being the elected master, it begins the discovery process again.
一旦主节点被选出，它通常会一直作为主节点，直到被故意停止。如果 [fault detection](https://www.elastic.co/guide/en/elasticsearch/reference/current/cluster-fault-detection.html) 确定集群出现故障，它也可能会停止充当主节点。当节点不再是主节点时，它会再次开始发现过程。

Refer to [Troubleshooting discovery](https://www.elastic.co/guide/en/elasticsearch/reference/current/discovery-troubleshooting.html) for troubleshooting issues with discovery.
有关发现问题的故障排除，请参阅 [Troubleshooting discover](https://www.elastic.co/guide/en/elasticsearch/reference/current/discovery-troubleshooting.html)。

---

## [Seed hosts providers | 种子主机提供商](https://www.elastic.co/guide/en/elasticsearch/reference/current/discovery-hosts-providers.html#built-in-hosts-providers)

By default the cluster formation module offers two seed hosts providers to configure the list of seed nodes: a *settings*-based and a *file*-based seed hosts provider. It can be extended to support cloud environments and other forms of seed hosts providers via [discovery plugins](https://www.elastic.co/guide/en/elasticsearch/plugins/8.15/discovery.html). Seed hosts providers are configured using the `discovery.seed_providers setting`, which defaults to the settings-based hosts provider. This setting accepts a list of different providers, allowing you to make use of multiple ways to find the seed hosts for your cluster.
默认情况下，集群形成模块提供两个种子主机提供程序来配置种子节点列表：基于 *settings* 和基于 *file* 的种子主机提供程序。它可以通过 [discovery plugins](https://www.elastic.co/guide/en/elasticsearch/plugins/8.15/discovery.html) 进行扩展，以支持云环境和其他形式的种子主机提供程序。种子主机提供程序使用 `discovery.seed_providers 设置` 进行配置，默认为基于设置的主机提供程序。此设置接受不同提供程序的列表，允许您使用多种方式查找集群的种子主机。

Each seed hosts provider yields the IP addresses or hostnames of the seed nodes. If it returns any hostnames then these are resolved to IP addresses using a DNS lookup. If a hostname resolves to multiple IP addresses then Elasticsearch tries to find a seed node at all of these addresses. If the hosts provider does not explicitly give the TCP port of the node by then, it will implicitly use the first port in the port range given by `transport.profiles.default.port`, or by `transport.port` if `transport.profiles.default.port` is not set. The number of concurrent lookups is controlled by `discovery.seed_resolver.max_concurrent_resolvers` which defaults to 10, and the timeout for each lookup is controlled by `discovery.seed_resolver.timeout` which defaults to 5s. Note that DNS lookups are subject to [JVM DNS caching](https://www.elastic.co/guide/en/elasticsearch/reference/current/networkaddress-cache-ttl.html).
每个种子主机提供商都会生成种子节点的 IP 地址或主机名。如果它返回任何主机名，则使用 DNS 查找将这些主机名解析为 IP 地址。如果主机名解析为多个 IP 地址，则 Elasticsearch 会尝试在所有这些地址上查找种子节点。如果主机提供商到那时没有明确提供节点的 TCP 端口，它将隐式使用 `transport.profiles.default.port` 或 `transport.port` 给出的端口范围中的第一个端口（如果未设置 `transport.profiles.default.port`）。并发查找的数量由 `discovery.seed_resolver.max_concurrent_resolvers` 控制，默认值为 10，每次查找的超时由 `discovery.seed_resolver.timeout` 控制，默认值为 5 秒。请注意，DNS 查找受 [JVM DNS caching](https://www.elastic.co/guide/en/elasticsearch/reference/current/networkaddress-cache-ttl.html) 约束。

---

### [Settings-based seed hosts provider | 基于设置的种子主机提供商](https://www.elastic.co/guide/en/elasticsearch/reference/current/discovery-hosts-providers.html#settings-based-hosts-provider)

The settings-based seed hosts provider uses a node setting to configure a static list of the addresses of the seed nodes. These addresses can be given as hostnames or IP addresses; hosts specified as hostnames are resolved to IP addresses during each round of discovery.
基于设置的种子主机提供程序使用节点设置来配置种子节点地址的静态列表。这些地址可以作为主机名或 IP 地址指定；在每轮发现期间，指定为主机名的主机都会解析为 IP 地址。

The list of hosts is set using the [discovery.seed_hosts](https://www.elastic.co/guide/en/elasticsearch/reference/current/important-settings.html#unicast.hosts) static setting. For example:
主机列表使用 [discovery.seed_hosts](https://www.elastic.co/guide/en/elasticsearch/reference/current/important-settings.html#unicast.hosts) 静态设置进行设置。例如：

``` yml
discovery.seed_hosts:
   - 192.168.1.10:9300
   - 192.168.1.11 ①
   - seeds.mydomain.com ②
```
- ① The port will default to transport.profiles.default.port and fallback to transport.port if not specified.
  ① 端口默认为transport.profiles.default.port，若未指定则回退到transport.port。
- ② If a hostname resolves to multiple IP addresses, Elasticsearch will attempt to connect to every resolved address.
  如果主机名解析为多个 IP 地址，Elasticsearch 将尝试连接到每个解析的地址。

---

### [File-based seed hosts provider | 基于文件的种子主机提供商](https://www.elastic.co/guide/en/elasticsearch/reference/current/discovery-hosts-providers.html#file-based-hosts-provider)

The file-based seed hosts provider configures a list of hosts via an external file. Elasticsearch reloads this file when it changes, so that the list of seed nodes can change dynamically without needing to restart each node. For example, this gives a convenient mechanism for an Elasticsearch instance that is run in a Docker container to be dynamically supplied with a list of IP addresses to connect to when those IP addresses may not be known at node startup.
基于文件的种子主机提供程序通过外部文件配置主机列表。Elasticsearch 会在文件更改时重新加载此文件，这样种子节点列表就可以动态更改，而无需重新启动每个节点。例如，这为在 Docker 容器中运行的 Elasticsearch 实例提供了一种方便的机制，当节点启动时可能不知道这些 IP 地址时，可以动态地为其提供要连接的 IP 地址列表。

To enable file-based discovery, configure the `file` hosts provider as follows in the` elasticsearch.yml` file:
要启用基于文件的发现，请在 `elasticsearch.yml` 文件中按如下方式配置 `file` 主机提供程序：

``` yml
discovery.seed_providers: file
```

Then create a file at `$ES_PATH_CONF/unicast_hosts.txt` in the format described below. Any time a change is made to the `unicast_hosts.txt` file the new changes will be picked up by Elasticsearch and the new hosts list will be used.
然后按照下面描述的格式在 `$ES_PATH_CONF/unicast_hosts.txt` 处创建一个文件。每当对 `unicast_hosts.txt` 文件进行更改时，Elasticsearch 都会获取新的更改并使用新的主机列表。

Note that the file-based discovery plugin augments the unicast hosts list in `elasticsearch.yml`: if there are valid seed addresses in `discovery.seed_hosts` then Elasticsearch uses those addresses in addition to those supplied in `unicast_hosts.txt`.
请注意，基于文件的发现插件会扩充`elasticsearch.yml`中的单播主机列表：如果`discovery.seed_hosts`中有有效的种子地址，那么 Elasticsearch 除了使用`unicast_hosts.txt`中提供的地址外，还会使用这些地址。

For example, this is an example of `unicast_hosts.txt` for a cluster with four nodes that participate in discovery, some of which are not running on the default port:
例如，这是一个参与发现的具有四个节点的集群的`unicast_hosts.txt`示例，其中一些节点未在默认端口上运行：

``` txt
10.10.10.5
10.10.10.6:9305
10.10.10.5:10005
# an IPv6 address
[2001:0db8:85a3:0000:0000:8a2e:0370:7334]:9301
```

Host names are allowed instead of IP addresses and are resolved by DNS as described above. IPv6 addresses must be given in brackets with the port, if needed, coming after the brackets.
允许使用主机名代替 IP 地址，并由 DNS 解析，如上所述。IPv6 地址必须放在括号中，如果需要，端口号也放在括号后面。

You can also add comments to this file. All comments must appear on their lines starting with `#` (i.e. comments cannot start in the middle of a line).
您还可以向此文件添加注释。所有注释都必须出现在以 `#` 开头的行上（即注释不能从行的中间开始）。

---

### [EC2 hosts provider | EC2 主机提供商](https://www.elastic.co/guide/en/elasticsearch/reference/current/discovery-hosts-providers.html#ec2-hosts-provider)

The [EC2 discovery plugin](https://www.elastic.co/guide/en/elasticsearch/plugins/8.15/discovery-ec2.html) adds a hosts provider that uses the [AWS API](https://github.com/aws/aws-sdk-java) to find a list of seed nodes.
[EC2 discovery plugin](https://www.elastic.co/guide/en/elasticsearch/plugins/8.15/discovery-ec2.html) 添加了主机提供程序，该提供程序使用 [AWS API](https://github.com/aws/aws-sdk-java) 来查找种子节点列表。

---

### [Azure Classic hosts provider | Azure 经典主机提供商](https://www.elastic.co/guide/en/elasticsearch/reference/current/discovery-hosts-providers.html#azure-classic-hosts-provider)

The [Azure Classic discovery plugin](https://www.elastic.co/guide/en/elasticsearch/plugins/8.15/discovery-azure-classic.html) adds a hosts provider that uses the Azure Classic API find a list of seed nodes.
[Azure Classic discovery plugin](https://www.elastic.co/guide/en/elasticsearch/plugins/8.15/discovery-azure-classic.html) 添加了主机提供程序，该提供程序使用 Azure Classic API 查找种子节点列表。

---

### [Google Compute Engine hosts provider | Google Compute Engine 托管提供商](https://www.elastic.co/guide/en/elasticsearch/reference/current/discovery-hosts-providers.html#gce-hosts-provider)

The [GCE discovery plugin](https://www.elastic.co/guide/en/elasticsearch/plugins/8.15/discovery-gce.html) adds a hosts provider that uses the GCE API find a list of seed nodes.
[GCE discovery plugin](https://www.elastic.co/guide/en/elasticsearch/plugins/8.15/discovery-gce.html) 添加了使用 GCE API 查找种子节点列表的主机提供程序。

