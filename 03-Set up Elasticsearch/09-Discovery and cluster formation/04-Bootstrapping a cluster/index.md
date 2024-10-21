# [Bootstrapping a cluster | 引导集群](https://www.elastic.co/guide/en/elasticsearch/reference/current/modules-discovery-bootstrap-cluster.html)

Starting an Elasticsearch cluster for the very first time requires the initial set of [master-eligible nodes](https://www.elastic.co/guide/en/elasticsearch/reference/current/modules-node.html#master-node) to be explicitly defined on one or more of the master-eligible nodes in the cluster. This is known as *cluster bootstrapping*. This is only required the first time a cluster starts up. Freshly-started nodes that are joining a running cluster obtain this information from the cluster’s elected master.
首次启动 Elasticsearch 集群时，需要在集群中的一个或多个符合主节点条件的节点上明确定义初始的 [master-eligible nodes](https://www.elastic.co/guide/en/elasticsearch/reference/current/modules-node.html#master-node)。这称为 *cluster bootstrapping*。这仅在集群首次启动时才需要。加入正在运行的集群的新启动节点会从集群的选定主节点获取此信息。

The initial set of master-eligible nodes is defined in the [`cluster.initial_master_nodes` setting](https://www.elastic.co/guide/en/elasticsearch/reference/current/important-settings.html#initial_master_nodes). This should be set to a list containing one of the following items for each master-eligible node:
初始的符合主节点资格的节点集在 [`cluster.initial_master_nodes` 设置](https://www.elastic.co/guide/en/elasticsearch/reference/current/important-settings.html#initial_master_nodes) 中定义。对于每个符合主节点资格的节点，应将其设置为包含以下项目之一的列表：

- The [node name](https://www.elastic.co/guide/en/elasticsearch/reference/current/important-settings.html#node-name) of the node.
  节点的 [node name](https://www.elastic.co/guide/en/elasticsearch/reference/current/important-settings.html#node-name)。
- The node’s hostname if `node.name` is not set, because `node.name` defaults to the node’s hostname. You must use either the fully-qualified hostname or the bare hostname [depending on your system configuration](https://www.elastic.co/guide/en/elasticsearch/reference/current/modules-discovery-bootstrap-cluster.html#modules-discovery-bootstrap-cluster-fqdns).
  如果未设置 `node.name`，则为节点的主机名，因为 `node.name` 默认为节点的主机名。您必须使用完全限定主机名或裸主机名[depending on your system configuration](https://www.elastic.co/guide/en/elasticsearch/reference/current/modules-discovery-bootstrap-cluster.html#modules-discovery-bootstrap-cluster-fqdns)。
- The IP address of the node’s [transport publish address](https://www.elastic.co/guide/en/elasticsearch/reference/current/modules-network.html#modules-network-binding-publishing), if it is not possible to use the `node.name` of the node. This is normally the IP address to which [`network.host`](https://www.elastic.co/guide/en/elasticsearch/reference/current/modules-network.html#common-network-settings) resolves but [this can be overridden](https://www.elastic.co/guide/en/elasticsearch/reference/current/modules-network.html#advanced-network-settings).
  如果无法使用节点的 `node.name`，则为节点的 [transport publish address](https://www.elastic.co/guide/en/elasticsearch/reference/current/modules-network.html#modules-network-binding-publishing) 的 IP 地址。这通常是 [`network.host`](https://www.elastic.co/guide/en/elasticsearch/reference/current/modules-network.html#common-network-settings) 解析到的 IP 地址，但 [this can be overridden](https://www.elastic.co/guide/en/elasticsearch/reference/current/modules-network.html#advanced-network-settings)。
- The IP address and port of the node’s publish address, in the form `IP:PORT`, if it is not possible to use the `node.name` of the node and there are multiple nodes sharing a single IP address.
  节点发布地址的 IP 地址和端口，格式为 `IP:PORT`，如果无法使用节点的 `node.name`，且有多个节点共享一个 IP 地址。

Do not set `cluster.initial_master_nodes` on master-ineligible nodes.
不要在不符合主资格的节点上设置“cluster.initial_master_nodes”。

> **IMPORTANT**
> After the cluster has formed, remove the `cluster.initial_master_nodes` setting from each node’s configuration and never set it again for this cluster. Do not configure this setting on nodes joining an existing cluster. Do not configure this setting on nodes which are restarting. Do not configure this setting when performing a full-cluster restart.
> 集群形成后，从每个节点的配置中删除 `cluster.initial_master_nodes` 设置，并且永远不要再为该集群设置它。不要在加入现有集群的节点上配置此设置。不要在正在重新启动的节点上配置此设置。不要在执行全集群重新启动时配置此设置。
> 
> If you leave `cluster.initial_master_nodes` in place once the cluster has formed then there is a risk that a future misconfiguration may result in bootstrapping a new cluster alongside your existing cluster. It may not be possible to recover from this situation without losing data.
> 如果集群形成后仍保留`cluster.initial_master_nodes`，则存在未来配置错误导致在现有集群旁边启动新集群的风险。可能无法在不丢失数据的情况下从这种情况中恢复。

The simplest way to create a new cluster is for you to select one of your master-eligible nodes that will bootstrap itself into a single-node cluster, which all the other nodes will then join. This simple approach is not resilient to failures until the other master-eligible nodes have joined the cluster. For example, if you have a master-eligible node with [node name](https://www.elastic.co/guide/en/elasticsearch/reference/current/important-settings.html#node-name) master-a then configure it as follows (omitting `cluster.initial_master_nodes` from the configuration of all other nodes):
创建新集群的最简单方法是选择一个主节点合格节点，该节点将自行引导到单节点集群中，然后所有其他节点将加入该集群。在其他主节点合格节点加入集群之前，这种简单的方法无法抵御故障。例如，如果您有一个 [node name](https://www.elastic.co/guide/en/elasticsearch/reference/current/important-settings.html#node-name) master-a 的主节点合格节点，则请按如下方式配置它（从所有其他节点的配置中省略`cluster.initial_master_nodes`）：

``` yml
cluster.initial_master_nodes: master-a
```

For fault-tolerant cluster bootstrapping, use all the master-eligible nodes. For instance, if your cluster has 3 master-eligible nodes with [node names](https://www.elastic.co/guide/en/elasticsearch/reference/current/important-settings.html#node-name) `master-a`, `master-b` and `master-c` then configure them all as follows:
对于容错集群引导，请使用所有符合主节点条件的节点。例如，如果您的集群有 3 个符合主节点条件的节点，[node names](https://www.elastic.co/guide/en/elasticsearch/reference/current/important-settings.html#node-name) 分别为 `master-a`、`master-b` 和 `master-c`，则请按如下方式配置它们：

``` yml
cluster.initial_master_nodes:
  - master-a
  - master-b
  - master-c
```

> **IMPORTANT**
> You must set `cluster.initial_master_nodes` to the same list of nodes on each node on which it is set in order to be sure that only a single cluster forms during bootstrapping. If `cluster.initial_master_nodes` varies across the nodes on which it is set then you may bootstrap multiple clusters. It is usually not possible to recover from this situation without losing data.
> 您必须在每个设置了 `cluster.initial_master_nodes` 的节点上将其设置为相同的节点列表，以确保在引导期间仅形成单个集群。如果 `cluster.initial_master_nodes` 在设置了它的节点上有所不同，那么您可能会引导多个集群。通常不可能在不丢失数据的情况下从这种情况中恢复。

<div style="border-width:thin; border-style: dotted; border-color: blue; border-radius: 0.4em; padding: 1em;">

> **Node name formats must match**
> **节点名称格式必须匹配**
> 
> ---
> 
> The node names used in the `cluster.initial_master_nodes` list must exactly match the `node.name` properties of the nodes. By default the node name is set to the machine’s hostname which may or may not be fully-qualified depending on your system configuration. If each node name is a fully-qualified domain name such as `master-a.example.com` then you must use fully-qualified domain names in the `cluster.initial_master_nodes` list too; conversely if your node names are bare hostnames (without the `.example.com` suffix) then you must use bare hostnames in the `cluster.initial_master_nodes` list. If you use a mix of fully-qualified and bare hostnames, or there is some other mismatch between `node.name` and `cluster.initial_master_nodes`, then the cluster will not form successfully and you will see log messages like the following.
> `cluster.initial_master_nodes` 列表中使用的节点名称必须与节点的 `node.name` 属性完全匹配。默认情况下，节点名称设置为计算机的主机名，该主机名可能是也可能不是完全限定的，具体取决于您的系统配置。如果每个节点名称都是完全限定的域名（例如 `master-a.example.com`），那么您也必须在 `cluster.initial_master_nodes` 列表中使用完全限定的域名；相反，如果您的节点名称是裸主机名（没有 `.example.com` 后缀），那么您必须在 `cluster.initial_master_nodes` 列表中使用裸主机名。如果您混合使用完全限定和裸主机名，或者 `node.name` 和 `cluster.initial_master_nodes` 之间存在其他不匹配，则集群将无法成功形成，您将看到类似以下的日志消息。
>
> ``` log
> [master-a.example.com] master not discovered yet, this node has
> not previously joined a bootstrapped (v7+) cluster, and this
> node must discover master-eligible nodes [master-a, master-b] to
> bootstrap a cluster: have discovered [{master-b.example.com}{...
> ```

This message shows the node names `master-a.example.com` and `master-b.example.com` as well as the `cluster.initial_master_nodes` entries `master-a` and `master-b`, and it is clear from this message that they do not match exactly.
该消息显示节点名称`master-a.example.com`和`master-b.example.com`以及`cluster.initial_master_nodes`条目`master-a”和“master-b`，从该消息中可以清楚地看出它们并不完全匹配。

</div>

---

## [Choosing a cluster name | 选择集群名称](https://www.elastic.co/guide/en/elasticsearch/reference/current/modules-discovery-bootstrap-cluster.html#bootstrap-cluster-name)


The [`cluster.name`](https://www.elastic.co/guide/en/elasticsearch/reference/current/important-settings.html#cluster-name) setting enables you to create multiple clusters which are separated from each other. Nodes verify that they agree on their cluster name when they first connect to each other, and Elasticsearch will only form a cluster from nodes that all have the same cluster name. The default value for the cluster name is elasticsearch, but it is recommended to change this to reflect the logical name of the cluster.
[`cluster.name`](https://www.elastic.co/guide/en/elasticsearch/reference/current/important-settings.html#cluster-name) 设置可让您创建多个彼此独立的集群。节点在首次连接时会验证它们是否同意其集群名称，并且 Elasticsearch 只会从所有集群名称相同的节点组成集群。集群名称的默认值为 elasticsearch，但建议将其更改为反映集群的逻辑名称。

---

## [Auto-bootstrapping in development mode | 开发模式下的自动引导](开发模式下的自动引导)

By default each node will automatically bootstrap itself into a single-node cluster the first time it starts. If any of the following settings are configured then auto-bootstrapping will not take place:
默认情况下，每个节点在首次启动时都会自动引导到单节点集群。如果配置了以下任何设置，则不会进行自动引导：
- `discovery.seed_providers`
- `discovery.seed_hosts`
- `cluster.initial_master_nodes`

To add a new node into an existing cluster, configure `discovery.seed_hosts` or other relevant discovery settings so that the new node can discover the existing master-eligible nodes in the cluster. To bootstrap a new multi-node cluster, configure `cluster.initial_master_nodes` as described in the section on [cluster bootstrapping](https://www.elastic.co/guide/en/elasticsearch/reference/current/modules-discovery-bootstrap-cluster.html) as well as `discovery.seed_hosts` or other relevant discovery settings.
要将新节点添加到现有集群，请配置`discovery.seed_hosts`或其他相关发现设置，以便新节点可以发现集群中现有的符合主节点条件的节点。要引导新的多节点集群，请按照[cluster bootstrapping](https://www.elastic.co/guide/en/elasticsearch/reference/current/modules-discovery-bootstrap-cluster.html)部分所述配置`cluster.initial_master_nodes`以及`discovery.seed_hosts`或其他相关发现设置。

<div style="border-width:thin; border-style: dotted; border-color: blue; border-radius: 0.4em; padding: 1em;">

> **Forming a single cluster**
> **形成单个集群**
> 
> ---
>
> Once an Elasticsearch node has joined an existing cluster, or bootstrapped a new cluster, it will not join a different cluster. Elasticsearch will not merge separate clusters together after they have formed, even if you subsequently try and configure all the nodes into a single cluster. This is because there is no way to merge these separate clusters together without a risk of data loss. You can tell that you have formed separate clusters by checking the cluster UUID reported by `GET /` on each node.
> 一旦 Elasticsearch 节点加入现有集群或引导新集群，它就不会再加入其他集群。Elasticsearch 不会在集群形成后将单独的集群合并在一起，即使您随后尝试将所有节点配置为单个集群。这是因为没有办法将这些单独的集群合并在一起而不丢失数据。您可以通过检查每个节点上"GET /"报告的集群 UUID 来判断您已经形成了单独的集群。
> 
> If you intended to add a node into an existing cluster but instead bootstrapped a separate single-node cluster then you must start again:
> 如果您打算将节点添加到现有集群中，但却引导了单独的单节点集群，那么您必须重新开始：
> 
> - Shut down the node.
>   关闭节点。
> - Completely wipe the node by deleting the contents of its [data folder](https://www.elastic.co/guide/en/elasticsearch/reference/current/modules-node.html#data-path).
>   通过删除节点 [data folder](https://www.elastic.co/guide/en/elasticsearch/reference/current/modules-node.html#data-path) 的内容来彻底擦除节点。
> - Configure `discovery.seed_hosts` or `discovery.seed_providers` and other relevant discovery settings. Ensure `cluster.initial_master_nodes` is not set on any node.
    配置`discovery.seed_hosts`或`discovery.seed_providers`和其他相关发现设置。确保任何节点上均未设置`cluster.initial_master_nodes`。
> - Restart the node and verify that it joins the existing cluster rather than forming its own one-node cluster.
>   重新启动节点并验证它是否加入现有集群而不是形成自己的单节点集群。
> 
> If you intended to form a new multi-node cluster but instead bootstrapped a collection of single-node clusters then you must start again:
> 如果您打算组建一个新的多节点集群，但却引导了一组单节点集群，那么您必须重新开始：
> 
> - Shut down all the nodes.
>   关闭所有节点。
> - Completely wipe each node by deleting the contents of their [data folders](https://www.elastic.co/guide/en/elasticsearch/reference/current/modules-node.html#data-path).
>   通过删除每个节点的 [data folders](https://www.elastic.co/guide/en/elasticsearch/reference/current/modules-node.html#data-path) 的内容来彻底擦除每个节点。
> - Configure `cluster.initial_master_nodes` as described above.
>   按照上面所述配置“cluster.initial_master_nodes”。
> - Configure `discovery.seed_hosts` or `discovery.seed_providers` and other relevant discovery settings.
>   配置discovery.seed_hosts或discovery.seed_providers以及其他相关的发现设置。
> - Restart all the nodes and verify that they have formed a single cluster.
>   重新启动所有节点并验证它们已经形成单个集群。
> - Remove `cluster.initial_master_nodes` from every node’s configuration.
> 从每个节点的配置中删除 cluster.initial_master_nodes。

</div>

