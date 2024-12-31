[TOC]


---

# [Miscellaneous cluster settings | 其它集群设置](https://www.elastic.co/guide/en/elasticsearch/reference/current/misc-cluster-settings.html)

---


## [Metadata | 元数据](https://www.elastic.co/guide/en/elasticsearch/reference/current/misc-cluster-settings.html#cluster-read-only)

An entire cluster may be set to read-only with the following setting:
可以使用以下设置将整个集群设置为只读：

- **`cluster.blocks.read_only`**
  ([Dynamic](https://www.elastic.co/guide/en/elasticsearch/reference/current/settings.html#dynamic-cluster-setting)) Make the whole cluster read only (indices do not accept write operations), metadata is not allowed to be modified (create or delete indices). Defaults to `false`.
  （[Dynamic](https://www.elastic.co/guide/en/elasticsearch/reference/current/settings.html#dynamic-cluster-setting)）使整个集群只读（索引不接受写操作），不允许修改元数据（创建或删除索引）。默认为 `false`。
- **`cluster.blocks.read_only_allow_delete`**
  ([Dynamic](https://www.elastic.co/guide/en/elasticsearch/reference/current/settings.html#dynamic-cluster-setting)) Identical to `cluster.blocks.read_only` but allows to delete indices to free up resources. Defaults to `false`.
  （[Dynamic](https://www.elastic.co/guide/en/elasticsearch/reference/current/settings.html#dynamic-cluster-setting)）与 `cluster.blocks.read_only` 相同，但允许删除索引以释放资源。默认为 `false。`

> **WARNING**
> Don’t rely on this setting to prevent changes to your cluster. Any user with access to the [cluster-update-settings](https://www.elastic.co/guide/en/elasticsearch/reference/current/cluster-update-settings.html) API can make the cluster read-write again.
> 不要依赖此设置来阻止对集群的更改。任何有权访问 [cluster-update-settings](https://www.elastic.co/guide/en/elasticsearch/reference/current/cluster-update-settings.html) API 的用户都可以让集群再次变为读写状态。


---

## [Cluster shard limits | 集群分片限制](https://www.elastic.co/guide/en/elasticsearch/reference/current/misc-cluster-settings.html#cluster-shard-limit)

There is a limit on the number of shards in a cluster, based on the number of nodes in the cluster. This is intended to prevent a runaway process from creating too many shards which can harm performance and in extreme cases may destabilize your cluster.
根据集群中的节点数，集群中的分片数量存在限制。这是为了防止失控进程创建过多分片，因为过多分片可能会损害性能，在极端情况下甚至可能破坏集群的稳定性。

> **IMPORTANT**
> These limits are intended as a safety net to protect against runaway shard creation and are not a sizing recommendation. The exact number of shards your cluster can safely support depends on your hardware configuration and workload, and may be smaller than the default limits.
> 这些限制旨在作为防止分片创建失控的安全网，而不是大小建议。集群可以安全支持的分片的确切数量取决于您的硬件配置和工作负载，并且可能小于默认限制。
>
> We do not recommend increasing these limits beyond the defaults. Clusters with more shards may appear to run well in normal operation, but may take a very long time to recover from temporary disruptions such as a network partition or an unexpected node restart, and may encounter problems when performing maintenance activities such as a rolling restart or upgrade.
> 我们不建议将这些限制提高到超出默认值。具有更多分片的集群在正常运行时可能运行良好，但可能需要很长时间才能从网络分区或节点意外重启等临时中断中恢复，并且在执行滚动重启或升级等维护活动时可能会遇到问题。

If an operation, such as creating a new index, restoring a snapshot of an index, or opening a closed index would lead to the number of shards in the cluster going over this limit, the operation will fail with an error indicating the shard limit. To resolve this, either scale out your cluster by adding nodes, or [delete some indices](https://www.elastic.co/guide/en/elasticsearch/reference/current/indices-delete-index.html) to bring the number of shards below the limit.
如果某个操作（例如创建新索引、恢复索引快照或打开已关闭的索引）导致集群中的分片数量超出此限制，则该操作将失败并显示错误，指出分片数量已达到限制。要解决此问题，请通过添加节点来扩展集群，或[删除一些索引](https://www.elastic.co/guide/en/elasticsearch/reference/current/indices-delete-index.html)以使分片数量低于限制。

If a cluster is already over the limit, perhaps due to changes in node membership or setting changes, all operations that create or open indices will fail.
如果集群已超出限制（可能是由于节点成员资格的变化或设置变化），则创建或打开索引的所有操作都将失败。

The cluster shard limit defaults to 1000 shards per non-frozen data node for normal (non-frozen) indices and 3000 shards per frozen data node for frozen indices. Both primary and replica shards of all open indices count toward the limit, including unassigned shards. For example, an open index with 5 primary shards and 2 replicas counts as 15 shards. Closed indices do not contribute to the shard count.
对于普通（非冻结）索引，集群分片限制默认为每个非冻结数据节点 1000 个分片；对于冻结索引，集群分片限制默认为每个冻结数据节点 3000 个分片。所有开放索引的主分片和副本分片均计入此限制，包括未分配的分片。例如，具有 5 个主分片和 2 个副本的开放索引计为 15 个分片。已关闭的索引不计入分片数。

You can dynamically adjust the cluster shard limit with the following setting:
您可以使用以下设置动态调整集群分片限制：

- **`cluster.max_shards_per_node`**
  ([Dynamic](https://www.elastic.co/guide/en/elasticsearch/reference/current/settings.html#dynamic-cluster-setting)) Limits the total number of primary and replica shards for the cluster. Elasticsearch calculates the limit as follows:
  （[Dynamic](https://www.elastic.co/guide/en/elasticsearch/reference/current/settings.html#dynamic-cluster-setting)）限制集群的主分片和副本分片的总数。Elasticsearch 按如下方式计算限制：

  ``` txt
  cluster.max_shards_per_node * number of non-frozen data nodes
  ```

  Shards for closed indices do not count toward this limit. Defaults to `1000`. A cluster with no data nodes is unlimited.
  已关闭索引的分片不计入此限制。默认为 `1000`。没有数据节点的集群不受限制。

  Elasticsearch rejects any request that creates more shards than this limit allows. For example, a cluster with a `cluster.max_shards_per_node` setting of `100` and three data nodes has a shard limit of 300. If the cluster already contains 296 shards, Elasticsearch rejects any request that adds five or more shards to the cluster.
  Elasticsearch 会拒绝任何创建分片数量超过此限制的请求。例如，如果集群的`cluster.max_shards_per_node`设置为`100`，且有三个数据节点，则分片数量限制为 300。如果集群已包含 296 个分片，Elasticsearch 会拒绝任何向集群添加五个或更多分片的请求。

  Note that if `cluster.max_shards_per_node` is set to a higher value than the default, the limits for [mmap count](https://www.elastic.co/guide/en/elasticsearch/reference/current/vm-max-map-count.html) and [open file descriptors](https://www.elastic.co/guide/en/elasticsearch/reference/current/file-descriptors.html) might also require adjustment.
  请注意，如果 `cluster.max_shards_per_node` 设置为高于默认值，则 [mmap 计数](https://www.elastic.co/guide/en/elasticsearch/reference/current/vm-max-map-count.html) 和 [打开文件描述符](https://www.elastic.co/guide/en/elasticsearch/reference/current/file-descriptors.html) 的限制可能也需要调整。

  Notice that frozen shards have their own independent limit.
  请注意，冻结的碎片有其自己独立的限制。
- **`cluster.max_shards_per_node.frozen`**
  ([Dynamic](https://www.elastic.co/guide/en/elasticsearch/reference/current/settings.html#dynamic-cluster-setting)) Limits the total number of primary and replica frozen shards for the cluster. Elasticsearch calculates the limit as follows:
  （[动态]（https://www.elastic.co/guide/en/elasticsearch/reference/current/settings.html#dynamic-cluster-setting））限制集群的主分片和副本分片的冻结总数。Elasticsearch 按如下方式计算限制：

  ```
  cluster.max_shards_per_node.frozen * number of frozen data nodes
  ```

  Shards for closed indices do not count toward this limit. Defaults to `3000`. A cluster with no frozen data nodes is unlimited.
  已关闭索引的分片不计入此限制。默认为`3000`。没有冻结数据节点的集群不受限制。
  
  Elasticsearch rejects any request that creates more frozen shards than this limit allows. For example, a cluster with a `cluster.max_shards_per_node.frozen` setting of `100` and three frozen data nodes has a frozen shard limit of 300. If the cluster already contains 296 shards, Elasticsearch rejects any request that adds five or more frozen shards to the cluster.
  Elasticsearch 会拒绝任何创建的冻结分片数量超过此限制的请求。例如，如果集群的`cluster.max_shards_per_node.frozen`设置为`100`，且冻结数据节点为 3 个，则冻结分片数量限制为 300。如果集群已包含 296 个分片，Elasticsearch 会拒绝任何向集群添加 5 个或更多冻结分片的请求。

  > **NOTE**
  > These limits only apply to actions which create shards and do not limit the number of shards assigned to each node. To limit the number of shards assigned to each node, use the [cluster.routing.allocation.total_shards_per_node](https://www.elastic.co/guide/en/elasticsearch/reference/current/allocation-total-shards.html#cluster-total-shards-per-node) setting.


---

## [User-defined cluster metadata | 用户定义的集群元数据](https://www.elastic.co/guide/en/elasticsearch/reference/current/misc-cluster-settings.html#user-defined-data)

User-defined metadata can be stored and retrieved using the Cluster Settings API. This can be used to store arbitrary, infrequently-changing data about the cluster without the need to create an index to store it. This data may be stored using any key prefixed with `cluster.metadata..` For example, to store the email address of the administrator of a cluster under the key `cluster.metadata.administrator`, issue this request:
可以使用 Cluster Settings API 存储和检索用户定义的元数据。这可用于存储有关集群的任意、不经常更改的数据，而无需创建索引来存储它。可以使用任何以“cluster.metadata.”为前缀的键来存储此数据。例如，要将集群管理员的电子邮件地址存储在键“cluster.metadata.administrator”下，请发出此请求：

``` shell
PUT /_cluster/settings
{
  "persistent": {
    "cluster.metadata.administrator": "sysadmin@example.com"
  }
}
```

> **IMPORTANT**
> User-defined cluster metadata is not intended to store sensitive or confidential information. Any information stored in user-defined cluster metadata will be viewable by anyone with access to the [Cluster Get Settings](https://www.elastic.co/guide/en/elasticsearch/reference/current/cluster-get-settings.html) API, and is recorded in the Elasticsearch logs.
> 用户定义的集群元数据不用于存储敏感或机密信息。任何有权访问 [Cluster Get Settings](https://www.elastic.co/guide/en/elasticsearch/reference/current/cluster-get-settings.html) API 的人都可以查看用户定义的集群元数据中存储的任何信息，这些信息会记录在 Elasticsearch 日志中。


---

## [Index tombstones | 索引墓碑](https://www.elastic.co/guide/en/elasticsearch/reference/current/misc-cluster-settings.html#cluster-max-tombstones)

The cluster state maintains index tombstones to explicitly denote indices that have been deleted. The number of tombstones maintained in the cluster state is controlled by the following setting:
集群状态维护索引墓碑，以明确指示已删除的索引。集群状态中维护的墓碑数量由以下设置控制：

- **`cluster.indices.tombstones.size`**
  ([Static](https://www.elastic.co/guide/en/elasticsearch/reference/current/settings.html#static-cluster-setting)) Index tombstones prevent nodes that are not part of the cluster when a delete occurs from joining the cluster and reimporting the index as though the delete was never issued. To keep the cluster state from growing huge we only keep the last `cluster.indices.tombstones.size` deletes, which defaults to 500. You can increase it if you expect nodes to be absent from the cluster and miss more than 500 deletes. We think that is rare, thus the default. Tombstones don’t take up much space, but we also think that a number like 50,000 is probably too big.
  （[Static](https://www.elastic.co/guide/en/elasticsearch/reference/current/settings.html#static-cluster-setting)）索引墓碑可防止在删除时不属于集群的节点加入集群并重新导入索引，就像从未发出删除一样。为了防止集群状态变得过大，我们只保留最后的 `cluster.indices.tombstones.size` 删除，默认值为 500。如果您预计节点不在集群中并且错过超过 500 次删除，则可以增加它。我们认为这种情况很少见，因此是默认值。墓碑不会占用太多空间，但我们也认为 50,000 这样的数字可能太大了。

If Elasticsearch encounters index data that is absent from the current cluster state, those indices are considered to be dangling. For example, this can happen if you delete more than `cluster.indices.tombstones.size` indices while an Elasticsearch node is offline.
如果 Elasticsearch 遇到当前集群状态中不存在的索引数据，则这些索引将被视为悬空索引。例如，如果您在 Elasticsearch 节点处于离线状态时删除超过“cluster.indices.tombstones.size”个索引，就会发生这种情况。

You can use the [Dangling indices API](https://www.elastic.co/guide/en/elasticsearch/reference/current/indices.html#dangling-indices-api) to manage this situation.
您可以使用 [悬垂索引 API](https://www.elastic.co/guide/en/elasticsearch/reference/current/indices.html#dangling-indices-api) 来管理这种情况。


---

## [Logger | 日志](https://www.elastic.co/guide/en/elasticsearch/reference/current/misc-cluster-settings.html#cluster-logger)

The settings which control logging can be updated [dynamically](https://www.elastic.co/guide/en/elasticsearch/reference/current/settings.html#dynamic-cluster-setting) with the `logger`. prefix. For instance, to increase the logging level of the `indices.recovery` module to `DEBUG`, issue this request:
控制日志记录的设置可以使用 `logger`. 前缀 [动态](https://www.elastic.co/guide/en/elasticsearch/reference/current/settings.html#dynamic-cluster-setting) 进行更新。例如，要将 `indices.recovery` 模块的日志记录级别提高到 `DEBUG`，请发出此请求：

``` shell
PUT /_cluster/settings
{
  "persistent": {
    "logger.org.elasticsearch.indices.recovery": "DEBUG"
  }
}
```

---

## [Persistent tasks allocation | 持久任务分配](https://www.elastic.co/guide/en/elasticsearch/reference/current/misc-cluster-settings.html#persistent-tasks-allocation)

Plugins can create a kind of tasks called persistent tasks. Those tasks are usually long-lived tasks and are stored in the cluster state, allowing the tasks to be revived after a full cluster restart.
插件可以创建一种称为持久任务的任务。这些任务通常是长期任务，并存储在集群状态中，允许在集群完全重启后恢复任务。

Every time a persistent task is created, the master node takes care of assigning the task to a node of the cluster, and the assigned node will then pick up the task and execute it locally. The process of assigning persistent tasks to nodes is controlled by the following settings:
每次创建持久任务时，主节点都会负责将任务分配给集群中的某个节点，然后分配的节点将拾取该任务并在本地执行。将持久任务分配给节点的过程由以下设置控制：

- **`cluster.persistent_tasks.allocation.enable`**
  ([Dynamic](https://www.elastic.co/guide/en/elasticsearch/reference/current/settings.html#dynamic-cluster-setting)) Enable or disable allocation for persistent tasks:
  （[Dynamic](https://www.elastic.co/guide/en/elasticsearch/reference/current/settings.html#dynamic-cluster-setting)）启用或禁用持久任务的分配：

  - `all` - (default) Allows persistent tasks to be assigned to nodes
    `all`-（默认）允许将持久任务分配给节点
  - `none` - No allocations are allowed for any type of persistent task
    `none` – 不允许为任何类型的持久任务分配

  This setting does not affect the persistent tasks that are already being executed. Only newly created persistent tasks, or tasks that must be reassigned (after a node left the cluster, for example), are impacted by this setting.
  此设置不会影响已在执行的持久性任务。只有新创建的持久性任务或必须重新分配的任务（例如，在节点离开集群后）才会受到此设置的影响。
- **`cluster.persistent_tasks.allocation.recheck_interval`**
  ([Dynamic](https://www.elastic.co/guide/en/elasticsearch/reference/current/settings.html#dynamic-cluster-setting)) The master node will automatically check whether persistent tasks need to be assigned when the cluster state changes significantly. However, there may be other factors, such as memory usage, that affect whether persistent tasks can be assigned to nodes but do not cause the cluster state to change. This setting controls how often assignment checks are performed to react to these factors. The default is 30 seconds. The minimum permitted value is 10 seconds.
  （[Dynamic](https://www.elastic.co/guide/en/elasticsearch/reference/current/settings.html#dynamic-cluster-setting)）主节点将在集群状态发生重大变化时自动检查是否需要分配持久任务。但是，可能还有其他因素（例如内存使用情况）会影响是否可以将持久任务分配给节点但不会导致集群状态发生变化。此设置控制执行分配检查的频率以对这些因素做出反应。默认值为 30 秒。允许的最小值为 10 秒。
