[TOC]

---

# [Secure settings | 安全设置](https://www.elastic.co/guide/en/elasticsearch/reference/current/secure-settings.html)

Some settings are sensitive, and relying on filesystem permissions to protect their values is not sufficient. For this use case, Elasticsearch provides a keystore and the [elasticsearch-keystore tool](https://www.elastic.co/guide/en/elasticsearch/reference/current/elasticsearch-keystore.html) to manage the settings in the keystore.
有些设置非常敏感，仅依靠文件系统权限来保护其值是不够的。对于这种用例，Elasticsearch 提供了一个密钥库和 [](https://www.elastic.co/guide/en/elasticsearch/reference/current/elasticsearch-keystore.html) 来管理密钥库中的设置。

> **IMPORTANT**
> Only some settings are designed to be read from the keystore. Adding unsupported settings to the keystore causes the validation in the `_nodes/reload_secure_settings` API to fail and if not addressed, will cause Elasticsearch to fail to start. To see whether a setting is supported in the keystore, look for a "Secure" qualifier in the setting reference.
> 只有一些设置被设计为从密钥库中读取。将不受支持的设置添加到密钥库会导致 `_nodes/reload_secure_settings` API 中的验证失败，如果不解决，将导致 Elasticsearch 无法启动。要查看密钥库中是否支持某个设置，请在设置参考中查找“安全”限定符。

All the modifications to the keystore take effect only after restarting Elasticsearch.
所有对 keystore 的修改只有重新启动 Elasticsearch 后才会生效。

These settings, just like the regular ones in the `elasticsearch.yml` config file, need to be specified on each node in the cluster. Currently, all secure settings are node-specific settings that must have the same value on every node.
这些设置与 elasticsearch.yml 配置文件中的常规设置一样，需要在集群中的每个节点上指定。目前，所有安全设置都是特定于节点的设置，每个节点上的值必须相同。


---

## [Reloadable secure settings | 可重新加载的安全设置](https://www.elastic.co/guide/en/elasticsearch/reference/current/secure-settings.html#reloadable-secure-settings)

Just like the settings values in `elasticsearch.yml`, changes to the keystore contents are not automatically applied to the running Elasticsearch node. Re-reading settings requires a node restart. However, certain secure settings are marked as **reloadable**. Such settings can be re-read and applied on a running node.
就像`elasticsearch.yml`中的设置值一样，对密钥库内容的更改不会自动应用于正在运行的 Elasticsearch 节点。重新读取设置需要重新启动节点。但是，某些安全设置被标记为**reloadable**。此类设置可以重新读取并应用于正在运行的节点。

You can define these settings before the node is started, or call the [Nodes reload secure settings API](https://www.elastic.co/guide/en/elasticsearch/reference/current/cluster-nodes-reload-secure-settings.html) after the settings are defined to apply them to a running node.
您可以在节点启动之前定义这些设置，或者在定义设置后调用 [Nodes reload secure settings API](https://www.elastic.co/guide/en/elasticsearch/reference/current/cluster-nodes-reload-secure-settings.html) 将其应用于正在运行的节点。

The values of all secure settings, **reloadable** or not, must be identical across all cluster nodes. After making the desired secure settings changes, using the `bin/elasticsearch-keystore add` command, call:
所有安全设置的值（无论是否**reloadable**）必须在所有集群节点上相同。使用`bin/elasticsearch-keystore add`命令进行所需的安全设置更改后，调用：
``` shell
POST _nodes/reload_secure_settings
{
  "secure_settings_password": "keystore-password" ①
}
```
① 	The password that the Elasticsearch keystore is encrypted with.
① Elasticsearch keystore 加密的密码。

This API decrypts, re-reads the entire keystore and validates all settings on every cluster node, but only the **reloadable** secure settings are applied. Changes to other settings do not go into effect until the next restart. Once the call returns, the reload has been completed, meaning that all internal data structures dependent on these settings have been changed. Everything should look as if the settings had the new value from the start.
此 API 解密、重新读取整个密钥库并验证每个集群节点上的所有设置，但仅应用**reloadable**安全设置。对其他设置的更改直到下次重新启动才会生效。调用返回后，重新加载已完成，这意味着所有依赖于这些设置的内部数据结构都已更改。一切看起来都应该像设置从一开始就具有新值一样。

When changing multiple **reloadable** secure settings, modify all of them on each cluster node, then issue a [reload_secure_settings](https://www.elastic.co/guide/en/elasticsearch/reference/current/cluster-nodes-reload-secure-settings.html) call instead of reloading after each modification.
更改多个 **reloadable** 安全设置时，请在每个集群节点上修改所有设置，然后发出 [reload_secure_settings](https://www.elastic.co/guide/en/elasticsearch/reference/current/cluster-nodes-reload-secure-settings.html) 调用，而不是在每次修改后重新加载。

There are reloadable secure settings for:
有可重新加载的安全设置：
- [The Azure repository plugin | Azure 存储库插件](https://www.elastic.co/guide/en/elasticsearch/reference/current/repository-azure.html)
- [The EC2 discovery plugin | EC2 发现插件](https://www.elastic.co/guide/en/elasticsearch/plugins/8.15/discovery-ec2-usage.html#_configuring_ec2_discovery)
- [The GCS repository plugin | GCS 存储库插件](https://www.elastic.co/guide/en/elasticsearch/reference/current/repository-gcs.html)
- [The S3 repository plugin | S3 存储库插件](https://www.elastic.co/guide/en/elasticsearch/reference/current/repository-s3.html)
- [Monitoring settings | 监控设置](https://www.elastic.co/guide/en/elasticsearch/reference/current/monitoring-settings.html)
- [Watcher settings | 观察者设置](https://www.elastic.co/guide/en/elasticsearch/reference/current/notification-settings.html)
- [JWT realm | JWT 领域](https://www.elastic.co/guide/en/elasticsearch/reference/current/security-settings.html#ref-jwt-settings)
- [Active Directory realm | Active Directory 领域](https://www.elastic.co/guide/en/elasticsearch/reference/current/security-settings.html#ref-ad-settings)
- [LDAP realm | LDAP 领域](https://www.elastic.co/guide/en/elasticsearch/reference/current/security-settings.html#ref-ldap-settings)
- [Remote cluster credentials for the API key based security model | 基于 API 密钥的安全模型的远程集群凭据](https://www.elastic.co/guide/en/elasticsearch/reference/current/remote-clusters-settings.html#remote-cluster-credentials-setting)



