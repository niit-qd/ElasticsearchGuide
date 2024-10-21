# Important Elasticsearch configuration

Elasticsearch requires very little configuration to get started, but there are a number of items which **must** be considered before using your cluster in production:
- [Path settings](https://www.elastic.co/guide/en/elasticsearch/reference/current/important-settings.html#path-settings "Path settings")
- [Cluster name setting](https://www.elastic.co/guide/en/elasticsearch/reference/current/important-settings.html#cluster-name "Cluster name setting")
- [Node name setting](https://www.elastic.co/guide/en/elasticsearch/reference/current/important-settings.html#node-name "Node name setting")
- [Network host settings](https://www.elastic.co/guide/en/elasticsearch/reference/current/important-settings.html#network.host "Network host setting")
- [Discovery settings](https://www.elastic.co/guide/en/elasticsearch/reference/current/important-settings.html#discovery-settings "Discovery and cluster formation settings")
- [Heap size settings](https://www.elastic.co/guide/en/elasticsearch/reference/current/important-settings.html#heap-size-settings "Heap size settings")
- [JVM heap dump path setting](https://www.elastic.co/guide/en/elasticsearch/reference/current/important-settings.html#heap-dump-path "JVM heap dump path setting")
- [GC logging settings](https://www.elastic.co/guide/en/elasticsearch/reference/current/important-settings.html#gc-logging "GC logging settings")
- [Temporary directory settings](https://www.elastic.co/guide/en/elasticsearch/reference/current/important-settings.html#es-tmpdir "Temporary directory settings")
- [JVM fatal error log setting](https://www.elastic.co/guide/en/elasticsearch/reference/current/important-settings.html#error-file-path "JVM fatal error log setting")
- [Cluster backups](https://www.elastic.co/guide/en/elasticsearch/reference/current/important-settings.html#important-settings-backups "Cluster backups")

Our [Elastic Cloud](https://www.elastic.co/cloud/elasticsearch-service/signup?page=docs&placement=docs-body) service configures these items automatically, making your cluster production-ready by default.
我们的 [Elastic Cloud](https://www.elastic.co/cloud/elasticsearch-service/signup?page=docs&placement=docs-body) 服务会自动配置这些项目，使您的集群默认可以投入生产。

---

## [Path settings](https://www.elastic.co/guide/en/elasticsearch/reference/current/important-settings.html#path-settings)

Elasticsearch writes the data you index to indices and data streams to a `data` directory. Elasticsearch writes its own application logs, which contain information about cluster health and operations, to a `logs` directory.
Elasticsearch 将您索引到索引和数据流的数据写入`data`目录。Elasticsearch 将其自己的应用程序日志（包含有关集群健康和操作的信息）写入`logs`目录。

For [macOS .tar.gz](https://www.elastic.co/guide/en/elasticsearch/reference/current/targz.html), [Linux .tar.gz](https://www.elastic.co/guide/en/elasticsearch/reference/current/targz.html), and [Windows .zip](https://www.elastic.co/guide/en/elasticsearch/reference/current/zip-windows.html) installations, `data` and `logs` are subdirectories of `$ES_HOME` by default. However, files in `$ES_HOME` risk deletion during an upgrade.
对于 [macOS .tar.gz](https://www.elastic.co/guide/en/elasticsearch/reference/current/targz.html)、[Linux .tar.gz](https://www.elastic.co/guide/en/elasticsearch/reference/current/targz.html) 和 [Windows .zip](https://www.elastic.co/guide/en/elasticsearch/reference/current/zip-windows.html) 安装，`data` 和 `logs` 默认为 `$ES_HOME` 的子目录。但是，`$ES_HOME` 中的文件在升级过程中存在被删除的风险。

Supported `path.data` and `path.logs` values vary by platform:
支持的`path.data`和`path.logs`值因平台而异：
- Unix-like systems
  Linux and macOS installations support Unix-style paths:
  ``` yml
  path:
    data: /var/data/elasticsearch
    logs: /var/log/elasticsearch
  ```
- Windows
  Windows installations support DOS paths with escaped backslashes:
  ``` yml
  path:
    data: "C:\\Elastic\\Elasticsearch\\data"
    logs: "C:\\Elastic\\Elasticsearch\\logs"
  ```

> **WARNING**
> Don’t modify anything within the data directory or run processes that might interfere with its contents. If something other than Elasticsearch modifies the contents of the data directory, then Elasticsearch may fail, reporting corruption or other data inconsistencies, or may appear to work correctly having silently lost some of your data. Don’t attempt to take filesystem backups of the data directory; there is no supported way to restore such a backup. Instead, [use Snapshot and restore](https://www.elastic.co/guide/en/elasticsearch/reference/current/snapshot-restore.html) to take backups safely. Don’t run virus scanners on the data directory. A virus scanner can prevent Elasticsearch from working correctly and may modify the contents of the data directory. The data directory contains no executables so a virus scan will only find false positives.
> **警告**
> 不要修改数据目录中的任何内容，也不要运行可能干扰其内容的进程。如果 Elasticsearch 以外的程序修改了数据目录的内容，Elasticsearch 可能会失败，报告损坏或其他数据不一致，或者可能看似正常工作但实际上丢失了部分数据。不要尝试对数据目录进行文件系统备份；目前没有支持恢复此类备份的方法。相反，请[use Snapshot and restore](https://www.elastic.co/guide/en/elasticsearch/reference/current/snapshot-restore.html)来安全地进行备份。不要在数据目录上运行病毒扫描程序。病毒扫描程序可能会阻止 Elasticsearch 正常工作，并可能修改数据目录的内容。数据目录不包含可执行文件，因此病毒扫描只会发现误报。

---

## [Multiple data paths | 多条数据路径](https://www.elastic.co/guide/en/elasticsearch/reference/current/important-settings.html#_multiple_data_paths)

> **WARNING** Deprecated in 7.13.0.

If needed, you can specify multiple paths in `path.data`. Elasticsearch stores the node’s data across all provided paths but keeps each shard’s data on the same path.
如果需要，您可以在`path.data`中指定多个路径。Elasticsearch 将节点的数据存储在所有提供的路径中，但将每个分片的数据保留在同一路径上。

Elasticsearch does not balance shards across a node’s data paths. High disk usage in a single path can trigger a [high disk usage watermark](https://www.elastic.co/guide/en/elasticsearch/reference/current/modules-cluster.html#disk-based-shard-allocation) for the entire node. If triggered, Elasticsearch will not add shards to the node, even if the node’s other paths have available disk space. If you need additional disk space, we recommend you add a new node rather than additional data paths.
Elasticsearch 不会在节点的数据路径之间平衡分片。单个路径中的高磁盘使用率可能会触发整个节点的[high disk usage watermark](https://www.elastic.co/guide/en/elasticsearch/reference/current/modules-cluster.html#disk-based-shard-allocation)。如果触发，Elasticsearch 将不会向节点添加分片，即使节点的其他路径有可用磁盘空间。如果您需要额外的磁盘空间，我们建议您添加新节点而不是额外的数据路径。

- Unix-like systems
  Linux and macOS installations support multiple Unix-style paths in `path.data`:
  ``` yml
  path:
    data:
      - /mnt/elasticsearch_1
      - /mnt/elasticsearch_2
      - /mnt/elasticsearch_3
  ```
- Windows
  Windows installations support multiple DOS paths in `path.data`:
  ``` yml
  path:
    data:
      - "C:\\Elastic\\Elasticsearch_1"
      - "E:\\Elastic\\Elasticsearch_1"
      - "F:\\Elastic\\Elasticsearch_3"
  ```

---

## [Migrate from multiple data paths | 从多个数据路径迁移](https://www.elastic.co/guide/en/elasticsearch/reference/current/important-settings.html#mdp-migrate)

Support for multiple data paths was deprecated in 7.13 and will be removed in a future release.
对多数据路径的支持在 7.13 中已被弃用，并且将在未来的版本中删除。

As an alternative to multiple data paths, you can create a filesystem which spans multiple disks with a hardware virtualisation layer such as RAID, or a software virtualisation layer such as Logical Volume Manager (LVM) on Linux or Storage Spaces on Windows. If you wish to use multiple data paths on a single machine then you must run one node for each data path.
作为多条数据路径的替代方案，您可以创建一个跨多个磁盘的文件系统，其中包含硬件虚拟化层（例如 RAID）或软件虚拟化层（例如 Linux 上的逻辑卷管理器 (LVM) 或 Windows 上的存储空间）。如果您希望在一台机器上使用多条数据路径，则必须为每个数据路径运行一个节点。

If you currently use multiple data paths in a [highly available cluster](https://www.elastic.co/guide/en/elasticsearch/reference/8.15/high-availability-cluster-design.html) then you can migrate to a setup that uses a single path for each node without downtime using a process similar to a [rolling restart](https://www.elastic.co/guide/en/elasticsearch/reference/8.15/restart-cluster.html#restart-cluster-rolling): shut each node down in turn and replace it with one or more nodes each configured to use a single data path. In more detail, for each node that currently has multiple data paths you should follow the following process. In principle you can perform this migration during a rolling upgrade to 8.0, but we recommend migrating to a single-data-path setup before starting to upgrade.
如果您当前在 [高可用性集群](https://www.elastic.co/guide/en/elasticsearch/reference/8.15/high-availability-cluster-design.html) 中使用多个数据路径，则可以使用类似于 [滚动重启](https://www.elastic.co/guide/en/elasticsearch/reference/8.15/restart-cluster.html#restart-cluster-rolling) 的过程迁移到每个节点使用单个路径的设置而无需停机：依次关闭每个节点并将其替换为一个或多个配置为使用单个数据路径的节点。更详细地说，对于当前具有多个数据路径的每个节点，您应遵循以下过程。原则上，您可以在滚动升级到 8.0 期间执行此迁移，但我们建议在开始升级之前迁移到单数据路径设置。

1. Take a snapshot to protect your data in case of disaster.
   拍摄快照以便在发生灾难时保护您的数据。
2. Optionally, migrate the data away from the target node by using an [allocation filter](https://www.elastic.co/guide/en/elasticsearch/reference/8.15/modules-cluster.html#cluster-shard-allocation-filtering):
   或者，使用分配过滤器将数据从目标节点迁移出去：
   ``` console
   PUT _cluster/settings
   {
     "persistent": {
       "cluster.routing.allocation.exclude._name": "target-node-name"
     }
   }
   ```
   You can use the [cat allocation API](https://www.elastic.co/guide/en/elasticsearch/reference/8.15/cat-allocation.html) to track progress of this data migration. If some shards do not migrate then the [cluster allocation explain API](https://www.elastic.co/guide/en/elasticsearch/reference/8.15/cluster-allocation-explain.html) will help you to determine why.
   您可以使用 [cat 分配 API](https://www.elastic.co/guide/en/elasticsearch/reference/8.15/cat-allocation.html) 跟踪此数据迁移的进度。如果某些分片未迁移，则 [集群分配说明 API](https://www.elastic.co/guide/en/elasticsearch/reference/8.15/cluster-allocation-explain.html) 将帮助您确定原因。
3. Follow the steps in the [rolling restart process](https://www.elastic.co/guide/en/elasticsearch/reference/8.15/restart-cluster.html#restart-cluster-rolling) up to and including shutting the target node down.
   按照[rolling restart process](https://www.elastic.co/guide/en/elasticsearch/reference/8.15/restart-cluster.html#restart-cluster-rolling)中的步骤进行操作，直至关闭目标节点。
4. Ensure your cluster health is `yellow` or `green`, so that there is a copy of every shard assigned to at least one of the other nodes in your cluster.
   确保您的集群健康状态为`yellow`或`green`，以便将每个分片的副本分配给集群中至少一个其他节点。
5. If applicable, remove the allocation filter applied in the earlier step.
   如果适用，请删除先前步骤中应用的分配过滤器。
   ``` console
   PUT _cluster/settings
   {
     "persistent": {
       "cluster.routing.allocation.exclude._name": null
     }
   }
   ```
6. Discard the data held by the stopped node by deleting the contents of its data paths.
   通过删除数据路径的内容来丢弃已停止节点所持有的数据。
7. Reconfigure your storage. For instance, combine your disks into a single filesystem using LVM or Storage Spaces. Ensure that your reconfigured storage has sufficient space for the data that it will hold.
   重新配置您的存储。例如，使用 LVM 或存储空间将您的磁盘组合成单个文件系统。确保重新配置的存储有足够的空间来容纳数据。
8. Reconfigure your node by adjusting the `path.data` setting in its `elasticsearch.yml` file. If needed, install more nodes each with their own `path.data` setting pointing at a separate data path.
   通过调整`elasticsearch.yml`文件中的`path.data`设置来重新配置节点。如果需要，安装更多节点，每个节点都有自己的`path.data`设置，指向单独的数据路径。
9.  Start the new nodes and follow the rest of the [rolling restart process](https://www.elastic.co/guide/en/elasticsearch/reference/8.15/restart-cluster.html#restart-cluster-rolling) for them.
    启动新节点并按照其余的 [rolling restart process](https://www.elastic.co/guide/en/elasticsearch/reference/8.15/restart-cluster.html#restart-cluster-rolling) 进行操作。
10. Ensure your cluster health is `green`, so that every shard has been assigned.
    确保您的集群健康状态为`green`，以便每个分片都已分配。

You can alternatively add some number of single-data-path nodes to your cluster, migrate all your data over to these new nodes using [allocation filters](https://www.elastic.co/guide/en/elasticsearch/reference/8.15/modules-cluster.html#cluster-shard-allocation-filtering), and then remove the old nodes from the cluster. This approach will temporarily double the size of your cluster so it will only work if you have the capacity to expand your cluster like this.
您也可以向集群添加一些单数据路径节点，使用 [allocation filters](https://www.elastic.co/guide/en/elasticsearch/reference/8.15/modules-cluster.html#cluster-shard-allocation-filtering) 将所有数据迁移到这些新节点，然后从集群中删除旧节点。此方法将暂时使集群规模翻倍，因此只有当您有能力像这样扩展集群时，此方法才会有效。

If you currently use multiple data paths but your cluster is not highly available then you can migrate to a non-deprecated configuration by taking a snapshot, creating a new cluster with the desired configuration and restoring the snapshot into it.
如果您当前使用多个数据路径，但您的集群可用性不高，那么您可以通过拍摄快照、创建具有所需配置的新集群并将快照恢复到其中来迁移到非弃用配置。

---

## [Cluster name setting | 集群名称设置](https://www.elastic.co/guide/en/elasticsearch/reference/current/important-settings.html#cluster-name)

A node can only join a cluster when it shares its `cluster.name` with all the other nodes in the cluster. The default name is `elasticsearch`, but you should change it to an appropriate name that describes the purpose of the cluster.
只有当节点与集群中的所有其他节点共享其 `cluster.name` 时，它​​才能加入集群。默认名称为`elasticsearch`，但您应该将其更改为适当的名称以描述集群的用途。
``` yml
cluster.name: logging-prod
```

> **IMPORTANT** 
> Do not reuse the same cluster names in different environments. Otherwise, nodes might join the wrong cluster.
> 不要在不同的环境中重复使用相同的集群名称。否则，节点可能会加入错误的集群。

> **NOTE**
> Changing the name of a cluster requires a [full cluster restart](https://www.elastic.co/guide/en/elasticsearch/reference/current/restart-cluster.html#restart-cluster-full).
> 更改集群名称需要[full cluster restart](https://www.elastic.co/guide/en/elasticsearch/reference/current/restart-cluster.html#restart-cluster-full)。

---

## [Node name setting | 节点名称设置](https://www.elastic.co/guide/en/elasticsearch/reference/current/important-settings.html#node-name)

Elasticsearch uses `node.name` as a human-readable identifier for a particular instance of Elasticsearch. This name is included in the response of many APIs. The node name defaults to the hostname of the machine when Elasticsearch starts, but can be configured explicitly in `elasticsearch.yml`:
Elasticsearch 使用 `node.name` 作为特定 Elasticsearch 实例的人性化标识符。此名称包含在许多 API 的响应中。Elasticsearch 启动时，节点名称默认为计算机的主机名，但可以在 `elasticsearch.yml` 中明确配置：
``` yml
node.name: prod-data-2
```

---

## [Network host setting | 网络主机设置](https://www.elastic.co/guide/en/elasticsearch/reference/current/important-settings.html#network.host)

By default, Elasticsearch only binds to loopback addresses such as `127.0.0.1` and `[::1]`. This is sufficient to run a cluster of one or more nodes on a single server for development and testing, but a [resilient production cluster](https://www.elastic.co/guide/en/elasticsearch/reference/current/high-availability-cluster-design.html) must involve nodes on other servers. There are many [network settings](https://www.elastic.co/guide/en/elasticsearch/reference/current/modules-network.html) but usually all you need to configure is `network.host`:
默认情况下，Elasticsearch 仅绑定到环回地址，例如 `127.0.0.1` 和 `[::1]`。这足以在单个服务器上运行一个或多个节点的集群以进行开发和测试，但 [resilient production cluster](https://www.elastic.co/guide/en/elasticsearch/reference/current/high-availability-cluster-design.html) 必须涉及其他服务器上的节点。有许多 [network settings](https://www.elastic.co/guide/en/elasticsearch/reference/current/modules-network.html)，但通常只需配置 `network.host`：
``` yml
network.host: 192.168.1.10
```

详细配置，可以参考：[Commonly used network settings](https://www.elastic.co/guide/en/elasticsearch/reference/current/modules-network.html#common-network-settings)、[Special values for network addresses](https://www.elastic.co/guide/en/elasticsearch/reference/current/modules-network.html#network-interface-values)。

> **IMPORTANT**
> When you provide a value for network.host, Elasticsearch assumes that you are moving from development mode to production mode, and upgrades a number of system startup checks from warnings to exceptions. See the differences between [development and production modes](https://www.elastic.co/guide/en/elasticsearch/reference/current/system-config.html#dev-vs-prod).
> 当您为 network.host 提供值时，Elasticsearch 会假定您正在从开发模式转为生产模式，并将许多系统启动检查从警告升级为异常。请参阅[development and production modes](https://www.elastic.co/guide/en/elasticsearch/reference/current/system-config.html#dev-vs-prod) 之间的差异。

---

## [Discovery and cluster formation settings | 发现和集群形成设置](https://www.elastic.co/guide/en/elasticsearch/reference/current/important-settings.html#discovery-settings)

Configure two important discovery and cluster formation settings before going to production so that nodes in the cluster can discover each other and elect a master node.
在投入生产之前配置两个重要的发现和集群形成设置，以便集群中的节点可以相互发现并选举主节点。

**[discovery.seed_hosts](https://www.elastic.co/guide/en/elasticsearch/reference/current/important-settings.html#unicast.hosts)**

Out of the box, without any network configuration, Elasticsearch will bind to the available loopback addresses and scan local ports `9300` to `9305` to connect with other nodes running on the same server. This behavior provides an auto-clustering experience without having to do any configuration.
开箱即用，无需任何网络配置，Elasticsearch 将绑定到可用的环回地址并扫描本地端口`9300`至`9305`，以连接在同一服务器上运行的其他节点。此行为无需进行任何配置即可提供自动集群体验。

When you want to form a cluster with nodes on other hosts, use the [static](https://www.elastic.co/guide/en/elasticsearch/reference/current/settings.html#static-cluster-setting) `discovery.seed_hosts` setting. This setting provides a list of other nodes in the cluster that are master-eligible and likely to be live and contactable to seed the [discovery process](https://www.elastic.co/guide/en/elasticsearch/reference/current/discovery-hosts-providers.html). This setting accepts a YAML sequence or array of the addresses of all the master-eligible nodes in the cluster. Each address can be either an IP address or a hostname that resolves to one or more IP addresses via DNS.
当您想要与其他主机上的节点组成集群时，请使用 [static](https://www.elastic.co/guide/en/elasticsearch/reference/current/settings.html#static-cluster-setting) `discovery.seed_hosts` 设置。此设置提供集群中符合主节点条件且可能处于活动状态且可联系的其他节点列表，以播种 [discovery process](https://www.elastic.co/guide/en/elasticsearch/reference/current/discovery-hosts-providers.html)。此设置接受集群中所有符合主节点条件的节点地址的 YAML 序列或数组。每个地址可以是 IP 地址，也可以是通过 DNS 解析为一个或多个 IP 地址的主机名。

``` yml
discovery.seed_hosts:
   - 192.168.1.10:9300
   - 192.168.1.11 # ①
   - seeds.mydomain.com # ②
   - [0:0:0:0:0:ffff:c0a8:10c]:9301 # ③
```
- ① The port is optional and defaults to 9300, but can be [overridden](https://www.elastic.co/guide/en/elasticsearch/reference/current/discovery-hosts-providers.html#built-in-hosts-providers).
  ① 端口是可选的，默认为 9300，但可以[overridden](https://www.elastic.co/guide/en/elasticsearch/reference/current/discovery-hosts-providers.html#built-in-hosts-providers)。
- ② If a hostname resolves to multiple IP addresses, the node will attempt to discover other nodes at all resolved addresses.
  ② 如果主机名解析为多个 IP 地址，则节点将尝试在所有解析的地址上发现其他节点。
- ③ IPv6 addresses must be enclosed in square brackets.
  ③ IPv6 地址必须用方括号括起来。

If your master-eligible nodes do not have fixed names or addresses, use an [alternative hosts provider](https://www.elastic.co/guide/en/elasticsearch/reference/current/discovery-hosts-providers.html#built-in-hosts-providers) to find their addresses dynamically.
如果您的主合格节点没有固定的名称或地址，请使用[alternative hosts provider](https://www.elastic.co/guide/en/elasticsearch/reference/current/discovery-hosts-providers.html#built-in-hosts-providers)动态查找其地址。

**[cluster.initial_master_nodes](https://www.elastic.co/guide/en/elasticsearch/reference/current/important-settings.html#initial_master_nodes)**

When you start an Elasticsearch cluster for the first time, a [cluster bootstrapping](https://www.elastic.co/guide/en/elasticsearch/reference/current/modules-discovery-bootstrap-cluster.html) step determines the set of master-eligible nodes whose votes are counted in the first election. In [development mode](https://www.elastic.co/guide/en/elasticsearch/reference/current/bootstrap-checks.html#dev-vs-prod-mode), with no discovery settings configured, this step is performed automatically by the nodes themselves.
首次启动 Elasticsearch 集群时，[cluster bootstrapping](https://www.elastic.co/guide/en/elasticsearch/reference/current/modules-discovery-bootstrap-cluster.html) 步骤会确定一组符合主节点条件的节点，这些节点的投票将在首次选举中计算在内。在 [development mode](https://www.elastic.co/guide/en/elasticsearch/reference/current/bootstrap-checks.html#dev-vs-prod-mode) 下，如果未配置发现设置，此步骤将由节点本身自动执行。

例如，[File-based seed hosts provider](https://www.elastic.co/guide/en/elasticsearch/reference/current/discovery-hosts-providers.html#file-based-hosts-provider)的`$ES_PATH_CONF/unicast_hosts.txt`。

Because auto-bootstrapping is [inherently unsafe](https://www.elastic.co/guide/en/elasticsearch/reference/current/modules-discovery-quorums.html), when starting a new cluster in production mode, you must explicitly list the master-eligible nodes whose votes should be counted in the very first election. You set this list using the `cluster.initial_master_nodes` setting on every master-eligible node. Do not configure this setting on master-ineligible nodes.
由于自动引导[inherently unsafe](https://www.elastic.co/guide/en/elasticsearch/reference/current/modules-discovery-quorums.html)，在生产模式下启动新集群时，您必须明确列出在第一次选举中应计入投票的符合主节点资格的节点。您可以使用每个符合主节点资格的节点上的`cluster.initial_master_nodes`设置来设置此列表。请勿在不符合主节点资格的节点上配置此设置。

> **IMPORTANT**
> After the cluster forms successfully for the first time, remove the `cluster.initial_master_nodes` setting from each node’s configuration and never set it again for this cluster. Do not configure this setting on nodes joining an existing cluster. Do not configure this setting on nodes which are restarting. Do not configure this setting when performing a full-cluster restart. See [Bootstrapping a cluster](https://www.elastic.co/guide/en/elasticsearch/reference/current/modules-discovery-bootstrap-cluster.html).
> 集群首次成功形成后，从每个节点的配置中删除 `cluster.initial_master_nodes` 设置，并且永远不要再为该集群设置它。不要在加入现有集群的节点上配置此设置。不要在重新启动的节点上配置此设置。不要在执行全集群重新启动时配置此设置。请参阅[Bootstrapping a cluster](https://www.elastic.co/guide/en/elasticsearch/reference/current/modules-discovery-bootstrap-cluster.html)。

``` yml
discovery.seed_hosts:
   - 192.168.1.10:9300
   - 192.168.1.11
   - seeds.mydomain.com
   - [0:0:0:0:0:ffff:c0a8:10c]:9301
cluster.initial_master_nodes: ①
   - master-node-a
   - master-node-b
   - master-node-c
```
① Identify the initial master nodes by their [`node.name`](https://www.elastic.co/guide/en/elasticsearch/reference/current/important-settings.html#node-name), which defaults to their hostname. Ensure that the value in `cluster.initial_master_nodes` matches the `node.name` exactly. If you use a fully-qualified domain name (FQDN) such as `master-node-a.example.com` for your node names, then you must use the FQDN in this list. Conversely, if `node.name` is a bare hostname without any trailing qualifiers, you must also omit the trailing qualifiers in `cluster.initial_master_nodes`.
① 通过其 [`node.name`](https://www.elastic.co/guide/en/elasticsearch/reference/current/important-settings.html#node-name) 识别初始主节点，默认为其主机名。确保 `cluster.initial_master_nodes` 中的值与 `node.name` 完全匹配。如果您使用完全限定域名 (FQDN)（例如 `master-node-a.example.com`）作为节点名称，则必须使用此列表中的 FQDN。相反，如果 `node.name` 是没有任何尾随限定符的裸主机名，您还必须省略 `cluster.initial_master_nodes` 中的尾随限定符。

See [bootstrapping a cluster](https://www.elastic.co/guide/en/elasticsearch/reference/current/modules-discovery-bootstrap-cluster.html) and [discovery and cluster formation settings](https://www.elastic.co/guide/en/elasticsearch/reference/current/modules-discovery-settings.html).
参见[bootstrapping a cluster](https://www.elastic.co/guide/en/elasticsearch/reference/current/modules-discovery-bootstrap-cluster.html)和[discovery and cluster formation settings](https://www.elastic.co/guide/en/elasticsearch/reference/current/modules-discovery-settings.html)。

---

## [Heap size settings | 堆大小设置](https://www.elastic.co/guide/en/elasticsearch/reference/current/important-settings.html#heap-size-settings)

By default, Elasticsearch automatically sets the JVM heap size based on a node’s [roles](https://www.elastic.co/guide/en/elasticsearch/reference/current/modules-node.html#node-roles) and total memory. We recommend the default sizing for most production environments.
默认情况下，Elasticsearch 会根据节点的[roles](https://www.elastic.co/guide/en/elasticsearch/reference/current/modules-node.html#node-roles)和总内存自动设置 JVM 堆大小。我们建议在大多数生产环境中使用默认大小。

If needed, you can override the default sizing by manually [setting the JVM heap size](https://www.elastic.co/guide/en/elasticsearch/reference/current/advanced-configuration.html#set-jvm-heap-size).
如果需要，您可以通过手动[setting the JVM heap size](https://www.elastic.co/guide/en/elasticsearch/reference/current/advanced-configuration.html#set-jvm-heap-size)覆盖默认大小。

---

## [JVM heap dump path setting | JVM 堆转储路径设置](https://www.elastic.co/guide/en/elasticsearch/reference/current/important-settings.html#heap-dump-path)

By default, Elasticsearch configures the JVM to dump the heap on out of memory exceptions to the default data directory. On [RPM](https://www.elastic.co/guide/en/elasticsearch/reference/current/rpm.html) and [Debian](https://www.elastic.co/guide/en/elasticsearch/reference/current/deb.html) packages, the data directory is `/var/lib/elasticsearch`. On [Linux and MacOS and Windows](https://www.elastic.co/guide/en/elasticsearch/reference/current/targz.html) distributions, the [data](https://www.elastic.co/guide/en/elasticsearch/reference/current/targz.html) directory is located under the root of the Elasticsearch installation.
默认情况下，Elasticsearch 将 JVM 配置为在发生内存不足异常时将堆转储到默认数据目录。在 [RPM](https://www.elastic.co/guide/en/elasticsearch/reference/current/rpm.html) 和 [Debian](https://www.elastic.co/guide/en/elasticsearch/reference/current/deb.html) 软件包中，数据目录为 `/var/lib/elasticsearch`。在 [Linux and MacOS and Windows](https://www.elastic.co/guide/en/elasticsearch/reference/current/targz.html) 发行版中，[data](https://www.elastic.co/guide/en/elasticsearch/reference/current/targz.html) 目录位于 Elasticsearch 安装的根目录下。

If this path is not suitable for receiving heap dumps, modify the `-XX:HeapDumpPath=...` entry in [jvm.options](https://www.elastic.co/guide/en/elasticsearch/reference/current/advanced-configuration.html#set-jvm-options):
如果此路径不适合接收堆转储，请修改 [jvm.options](https://www.elastic.co/guide/en/elasticsearch/reference/current/advanced-configuration.html#set-jvm-options) 中的 `-XX:HeapDumpPath=...` 条目：

- If you specify a directory, the JVM will generate a filename for the heap dump based on the PID of the running instance. 如果指定目录，JVM 将根据正在运行的实例的 PID 为堆转储生成文件名。
- If you specify a fixed filename instead of a directory, the file must not exist when the JVM needs to perform a heap dump on an out of memory exception. Otherwise, the heap dump will fail. 如果指定固定文件名而不是目录，则当 JVM 需要针对内存不足异常执行堆转储时，该文件必须不存在。否则，堆转储将失败。

---

## [GC logging settings | GC 日志记录设置](https://www.elastic.co/guide/en/elasticsearch/reference/current/important-settings.html#gc-logging)

By default, Elasticsearch enables garbage collection (GC) logs. These are configured in [jvm.options](https://www.elastic.co/guide/en/elasticsearch/reference/current/advanced-configuration.html#set-jvm-options) and output to the same default location as the Elasticsearch logs. The default configuration rotates the logs every 64 MB and can consume up to 2 GB of disk space.
默认情况下，Elasticsearch 会启用垃圾收集 (GC) 日志。这些日志在 [jvm.options](https://www.elastic.co/guide/en/elasticsearch/reference/current/advanced-configuration.html#set-jvm-options) 中配置，并输出到与 Elasticsearch 日志相同的默认位置。默认配置每 64 MB 轮换一次日志，最多可占用 2 GB 的磁盘空间。

You can reconfigure JVM logging using the command line options described in [JEP 158: Unified JVM Logging](https://openjdk.java.net/jeps/158). Unless you change the default `jvm.options` file directly, the Elasticsearch default configuration is applied in addition to your own settings. To disable the default configuration, first disable logging by supplying the `-Xlog:disable` option, then supply your own command line options. This disables *all* JVM logging, so be sure to review the available options and enable everything that you require.
您可以使用 [JEP 158: Unified JVM Logging](https://openjdk.java.net/jeps/158) 中描述的命令行选项重新配置 JVM 日志记录。除非您直接更改默认的 `jvm.options` 文件，否则除了您自己的设置外，还会应用 Elasticsearch 默认配置。要禁用默认配置，请先通过提供 `-Xlog:disable` 选项禁用日志记录，然后提供您自己的命令行选项。这将禁用 *所有* JVM 日志记录，因此请务必查看可用选项并启用您需要的所有内容。

To see further options not contained in the original JEP, see [Enable Logging with the JVM Unified Logging Framework](https://docs.oracle.com/en/java/javase/13/docs/specs/man/java.html#enable-logging-with-the-jvm-unified-logging-framework).
要查看原始 JEP 中未包含的更多选项，请参阅[Enable Logging with the JVM Unified Logging Framework](https://docs.oracle.com/en/java/javase/13/docs/specs/man/java.html#enable-logging-with-the-jvm-unified-logging-framework)

---

## [Examples | 示例](https://www.elastic.co/guide/en/elasticsearch/reference/current/important-settings.html#gc-logging-examples)

Change the default GC log output location to `/opt/my-app/gc.log` by creating `$ES_HOME/config/jvm.options.d/gc.options` with some sample options:
通过创建 `$ES_HOME/config/jvm.options.d/gc.options` 并使用一些示例选项将默认 GC 日志输出位置更改为 `/opt/my-app/gc.log`：

``` options 
# Turn off all previous logging configuratons
-Xlog:disable

# Default settings from JEP 158, but with `utctime` instead of `uptime` to match the next line
-Xlog:all=warning:stderr:utctime,level,tags

# Enable GC logging to a custom location with a variety of options
-Xlog:gc*,gc+age=trace,safepoint:file=/opt/my-app/gc.log:utctime,level,pid,tags:filecount=32,filesize=64m
```

Configure an Elasticsearch [Docker container](https://www.elastic.co/guide/en/elasticsearch/reference/current/docker.html) to send GC debug logs to standard error (`stderr`). This lets the container orchestrator handle the output. If using the `ES_JAVA_OPTS` environment variable, specify:
配置 Elasticsearch [Docker container](https://www.elastic.co/guide/en/elasticsearch/reference/current/docker.html)，将 GC 调试日志发送到标准错误 (`stderr`)。这允许容器编排器处理输出。如果使用 `ES_JAVA_OPTS` 环境变量，请指定：
``` shell
MY_OPTS="-Xlog:disable -Xlog:all=warning:stderr:utctime,level,tags -Xlog:gc=debug:stderr:utctime"
docker run -e ES_JAVA_OPTS="$MY_OPTS" # etc
```

---

## [Temporary directory settings | 临时目录设置](https://www.elastic.co/guide/en/elasticsearch/reference/current/important-settings.html#es-tmpdir)

By default, Elasticsearch uses a private temporary directory that the startup script creates immediately below the system temporary directory.
默认情况下，Elasticsearch 使用启动脚本在系统临时目录下方立即创建的私有临时目录。

On some Linux distributions, a system utility will clean files and directories from `/tmp` if they have not been recently accessed. This behavior can lead to the private temporary directory being removed while Elasticsearch is running if features that require the temporary directory are not used for a long time. Removing the private temporary directory causes problems if a feature that requires this directory is subsequently used.
在某些 Linux 发行版中，如果文件和目录最近没有被访问过，系统实用程序会从 `/tmp` 中清除它们。如果需要临时目录的功能长时间未使用，此行为可能会导致在 Elasticsearch 运行时删除私有临时目录。如果随后使用需要此目录的功能，则删除私有临时目录会导致问题。

If you install Elasticsearch using the `.deb` or `.rpm` packages and run it under `systemd`, the private temporary directory that Elasticsearch uses is excluded from periodic cleanup.
如果您使用 `.deb` 或 `.rpm` 包安装 Elasticsearch 并在 `systemd` 下运行它，则 Elasticsearch 使用的私有临时目录将被排除在定期清理之外。

If you intend to run the `.tar.gz` distribution on Linux or MacOS for an extended period, consider creating a dedicated temporary directory for Elasticsearch that is not under a path that will have old files and directories cleaned from it. This directory should have permissions set so that only the user that Elasticsearch runs as can access it. Then, set the `$ES_TMPDIR` environment variable to point to this directory before starting Elasticsearch.
如果您打算在 Linux 或 MacOS 上长期运行 `.tar.gz` 发行版，请考虑为 Elasticsearch 创建一个专用的临时目录，该目录不在会清除旧文件和目录的路径下。此目录应设置权限，以便只有运行 Elasticsearch 的用户才能访问它。然后，在启动 Elasticsearch 之前，将 `$ES_TMPDIR` 环境变量设置为指向此目录。

---

## [JVM fatal error log setting | JVM致命错误日志设置](https://www.elastic.co/guide/en/elasticsearch/reference/current/important-settings.html#error-file-path)

By default, Elasticsearch configures the JVM to write fatal error logs to the default logging directory. On [RPM](https://www.elastic.co/guide/en/elasticsearch/reference/current/rpm.html) and [Debian](https://www.elastic.co/guide/en/elasticsearch/reference/current/deb.html) packages, this directory is `/var/log/elasticsearch`. On [Linux and MacOS](https://www.elastic.co/guide/en/elasticsearch/reference/current/targz.html) and [Windows](https://www.elastic.co/guide/en/elasticsearch/reference/current/zip-windows.html) distributions, the `logs` directory is located under the root of the Elasticsearch installation.
默认情况下，Elasticsearch 将 JVM 配置为将致命错误日志写入默认日志目录。在 [RPM](https://www.elastic.co/guide/en/elasticsearch/reference/current/rpm.html) 和 [Debian](https://www.elastic.co/guide/en/elasticsearch/reference/current/deb.html) 软件包中，此目录为 `/var/log/elasticsearch`。在 [Linux and MacOS](https://www.elastic.co/guide/en/elasticsearch/reference/current/targz.html) 和 [Windows](https://www.elastic.co/guide/en/elasticsearch/reference/current/zip-windows.html) 发行版中，`logs` 目录位于 Elasticsearch 安装的根目录下。

These are logs produced by the JVM when it encounters a fatal error, such as a segmentation fault. If this path is not suitable for receiving logs, modify the `-XX:ErrorFile=...` entry in [jvm.options](https://www.elastic.co/guide/en/elasticsearch/reference/current/advanced-configuration.html#set-jvm-options).
这些是 JVM 在遇到致命错误（例如分段错误）时生成的日志。如果此路径不适合接收日志，请修改 [jvm.options](https://www.elastic.co/guide/en/elasticsearch/reference/current/advanced-configuration.html#set-jvm-options) 中的 `-XX:ErrorFile=...` 条目。

---

## [Cluster backups | 集群备份](https://www.elastic.co/guide/en/elasticsearch/reference/current/important-settings.html#important-settings-backups)

In a disaster, [snapshots](https://www.elastic.co/guide/en/elasticsearch/reference/current/snapshot-restore.html) can prevent permanent data loss. [Snapshot lifecycle management](https://www.elastic.co/guide/en/elasticsearch/reference/current/snapshots-take-snapshot.html#automate-snapshots-slm) is the easiest way to take regular backups of your cluster. For more information, see [Create a snapshot](https://www.elastic.co/guide/en/elasticsearch/reference/current/snapshots-take-snapshot.html).
在发生灾难时，[快照](https://www.elastic.co/guide/en/elasticsearch/reference/current/snapshot-restore.html) 可以防止永久性数据丢失。[Snapshot lifecycle management](https://www.elastic.co/guide/en/elasticsearch/reference/current/snapshots-take-snapshot.html#automate-snapshots-slm) 是定期备份集群的最简单方法。有关更多信息，请参阅[创建快照](https://www.elastic.co/guide/en/elasticsearch/reference/current/snapshots-take-snapshot.html)。

> **WARNING**
> **Taking a snapshot is the only reliable and supported way to back up a cluster.** You cannot back up an Elasticsearch cluster by making copies of the data directories of its nodes. There are no supported methods to restore any data from a filesystem-level backup. If you try to restore a cluster from such a backup, it may fail with reports of corruption or missing files or other data inconsistencies, or it may appear to have succeeded having silently lost some of your data.
**拍摄快照是备份集群的唯一可靠且受支持的方法。** 您无法通过复制其节点的数据目录来备份 Elasticsearch 集群。目前不支持从文件系统级备份中恢复任何数据的方法。如果您尝试从此类备份中恢复集群，可能会失败并报告损坏或丢失文件或其他数据不一致，或者可能看似成功但实际上丢失了部分数据。