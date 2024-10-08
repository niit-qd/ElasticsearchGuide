**[Configuring Elasticsearch](https://www.elastic.co/guide/en/elasticsearch/reference/current/settings.html)**

---

Elasticsearch ships with good defaults and requires very little configuration. Most settings can be changed on a running cluster using the [Cluster update settings](https://www.elastic.co/guide/en/elasticsearch/reference/current/cluster-update-settings.html) API.
Elasticsearch 具有良好的默认设置，几乎不需要配置。大多数设置都可以在正在运行的集群上使用 [Cluster update settings](https://www.elastic.co/guide/en/elasticsearch/reference/current/cluster-update-settings.html) 进行更改。

The configuration files should contain settings which are node-specific (such as `node.name` and paths), or settings which a node requires in order to be able to join a cluster, such as `cluster.name` and `network.host`.
配置文件应该包含特定于节点的设置（例如“node.name”和路径），或者节点为了能够加入集群所需的设置，例如“cluster.name”和“network.host”。

---

### [Config files location | 配置文件位置](https://www.elastic.co/guide/en/elasticsearch/reference/current/settings.html#config-files-location)

Elasticsearch has three configuration files:
- `elasticsearch.yml` for configuring Elasticsearch
- `jvm.options` for configuring Elasticsearch JVM settings
- `log4j2.properties` for configuring Elasticsearch logging

Elasticsearch 有三个配置文件：
- `elasticsearch.yml` 用于配置 Elasticsearch
- `jvm.options` 用于配置 Elasticsearch JVM 设置
- `log4j2.properties` 用于配置 Elasticsearch 日志记录

These files are located in the config directory, whose default location depends on whether or not the installation is from an archive distribution (`tar.gz` or `zip`) or a package distribution (Debian or RPM packages).
这些文件位于配置目录中，其默认位置取决于安装是否来自存档分发（“tar.gz”或“zip”）或软件包分发（Debian 或 RPM 软件包）。

For the archive distributions, the config directory location defaults to `$ES_HOME/config`. The location of the config directory can be changed via the `ES_PATH_CONF` environment variable as follows:
对于存档发行版，配置目录位置默认为 `$ES_HOME/config`。可以通过 `ES_PATH_CONF` 环境变量更改配置目录的位置，如下所示：
``` service
ES_PATH_CONF=/path/to/my/config ./bin/elasticsearch
```

> 此部分的配置，可以查看elasticsearch的启动路径来分析。
> 这里以[Linux x86_64](https://www.elastic.co/cn/downloads/elasticsearch)版本为例。
> - `bin\elasticsearch`
>   调用`bin\elasticsearch-cli`
> - `bin\elasticsearch-cli`
>   调用`elasticsearch-env`配置环境变量
>   使用这些环境变量，执行`org.elasticsearch.launcher.CliToolLauncher`。
>   从这里也可以看到，实际执行的类在`lib\cli-launcher\cli-launcher-8.15.2.jar`中。
> - `bin\elasticsearch-env`
>   会配置各个环境变量。如果某个环境变量不存在，则会使用默认值。
>   例如`ES_PATH_CONF`。
>   ``` shell
>   if [ -z "$ES_PATH_CONF" ]; then ES_PATH_CONF="$ES_HOME"/config; fi
>   ```

Alternatively, you can `export` the `ES_PATH_CONF` environment variable via the command line or via your shell profile.
或者，您可以通过命令行或 shell 配置文件`export` `ES_PATH_CONF` 环境变量。

For the package distributions, the config directory location defaults to `/etc/elasticsearch`. The location of the config directory can also be changed via the `ES_PATH_CONF` environment variable, but note that setting this in your shell is not sufficient. Instead, this variable is sourced from `/etc/default/elasticsearch` (for the Debian package) and `/etc/sysconfig/elasticsearch` (for the RPM package). You will need to edit the `ES_PATH_CONF=/etc/elasticsearch` entry in one of these files accordingly to change the config directory location.
对于软件包发行版，配置目录位置默认为 `/etc/elasticsearch`。也可以通过 `ES_PATH_CONF` 环境变量更改配置目录的位置，但请注意，在 shell 中设置它是不够的。相反，此变量来自 `/etc/default/elasticsearch`（对于 Debian 软件包）和 `/etc/sysconfig/elasticsearch`（对于 RPM 软件包）。您需要相应地编辑其中一个文件中的 `ES_PATH_CONF=/etc/elasticsearch` 条目以更改配置目录位置。
> 这里之所以说只在shell中设置是不够的，原因是对于包安装器方式安装的版本，例如rpm、deb等，Elasticsearch 的启动是通过`systemctl/service`来启动的，启动文件存在可以通过`systemctl cat elasticsearch.service`中启动。而`systemd`无法继承来自于shell中系统环境变量，所以需要在原始service文件或者使用`sudo systemctl edit elasticsearch`方式重写默认值。

---

### [Config file format | 配置文件格式](https://www.elastic.co/guide/en/elasticsearch/reference/current/settings.html#_config_file_format)

The configuration format is **`YAML`**. Here is an example of changing the path of the data and logs directories:
配置格式为 **`YAML`**。下面是更改数据和日志目录路径的示例：
``` yml
path:
    data: /var/lib/elasticsearch
    logs: /var/log/elasticsearch
```
Settings can also be flattened as follows:
设置也可以按如下方式展平：
``` yml
path.data: /var/lib/elasticsearch
path.logs: /var/log/elasticsearch
```
In YAML, you can format non-scalar values as sequences:
在 YAML 中，您可以将非标量值格式化为序列：
``` yml
discovery.seed_hosts:
   - 192.168.1.10:9300
   - 192.168.1.11
   - seeds.mydomain.com
```
Though less common, you can also format non-scalar values as arrays:
虽然不太常见，但您也可以将非标量值格式化为数组：
``` yml
discovery.seed_hosts: ["192.168.1.10:9300", "192.168.1.11", "seeds.mydomain.com"]
```

---

[Environment variable substitution | 环境变量替换](https://www.elastic.co/guide/en/elasticsearch/reference/current/settings.html#_environment_variable_substitution)

Environment variables referenced with the `${...}` notation within the configuration file will be replaced with the value of the environment variable. For example:
配置文件中使用 `${...}` 符号引用的环境变量将被替换为环境变量的值。例如：
``` yml
node.name:    ${HOSTNAME}
network.host: ${ES_NETWORK_HOST}
```
Values for environment variables must be simple strings. Use a comma-separated string to provide values that Elasticsearch will parse as a list. For example, Elasticsearch will split the following string into a list of values for the `${HOSTNAME}` environment variable:
环境变量的值必须是简单字符串。使用逗号分隔的字符串提供 Elasticsearch 将解析为列表的值。例如，Elasticsearch 将把以下字符串拆分为 `${HOSTNAME}` 环境变量的值列表：
``` shell
export HOSTNAME="host1,host2"
```

---

### [Cluster and node setting types | 集群和节点设置类型](https://www.elastic.co/guide/en/elasticsearch/reference/current/settings.html#cluster-setting-types)

Cluster and node settings can be categorized based on how they are configured:
集群和节点设置可以根据其配置方式进行分类：

**Dynamic**

You can configure and update dynamic settings on a running [cluster using the cluster update settings API](https://www.elastic.co/guide/en/elasticsearch/reference/current/cluster-update-settings.html). You can also configure dynamic settings locally on an unstarted or shut down node using `elasticsearch.yml`.
您可以在正在运行的[cluster using the cluster update settings API](https://www.elastic.co/guide/en/elasticsearch/reference/current/cluster-update-settings.html)配置和更新动态设置。您还可以使用“elasticsearch.yml”在未启动或关闭的节点上本地配置动态设置。

Updates made using the cluster update settings API can be persistent, which apply across cluster restarts, or *transient*, which reset after a cluster restart. You can also reset transient or persistent settings by assigning them a `null` value using the API.
使用集群更新设置 API 进行的更新可以是持久性的（在集群重启后应用），也可以是*暂时性的*（在集群重启后重置）。您还可以通过使用 API 为暂时性或持久性设置分配“null”值来重置它们。

If you configure the same setting using multiple methods, Elasticsearch applies the settings in following order of precedence:
如果使用多种方法配置相同的设置，Elasticsearch 将按照以下优先顺序应用设置：
- Transient setting
- Persistent setting
- `elasticsearch.yml` setting
- Default setting value

For example, you can apply a transient setting to override a persistent setting or `elasticsearch.yml` setting. However, a change to an `elasticsearch.yml` setting will not override a defined transient or persistent setting.

> If you use Elasticsearch Service, use the [user settings](https://www.elastic.co/guide/en/cloud/current/ec-add-user-settings.html) feature to configure all cluster settings. This method lets Elasticsearch Service automatically reject unsafe settings that could break your cluster.
> 如果您使用 Elasticsearch Service，请使用 [user setting](https://www.elastic.co/guide/en/cloud/current/ec-add-user-settings.html) 功能配置所有集群设置。此方法可让 Elasticsearch Service 自动拒绝可能破坏集群的不安全设置。
>
> If you run Elasticsearch on your own hardware, use the [cluster update settings API](https://www.elastic.co/guide/en/elasticsearch/reference/current/cluster-update-settings.html) to configure dynamic cluster settings. Only use `elasticsearch.yml` for static cluster settings and node settings. The API doesn’t require a restart and ensures a setting’s value is the same on all nodes.
> 如果您在自己的硬件上运行 Elasticsearch，请使用 [集群更新设置 API](https://www.elastic.co/guide/en/elasticsearch/reference/current/cluster-update-settings.html) 配置动态集群设置。仅使用 `elasticsearch.yml` 进行静态集群设置和节点设置。该 API 不需要重新启动，并确保设置的值在所有节点上都相同。

> We no longer recommend using transient cluster settings. Use persistent cluster settings instead. If a cluster becomes unstable, transient settings can clear unexpectedly, resulting in a potentially undesired cluster configuration. See the [Transient settings migration guide](https://www.elastic.co/guide/en/elasticsearch/reference/current/transient-settings-migration-guide.html).
> 我们不再建议使用临时集群设置。请改用持久集群设置。如果集群变得不稳定，临时设置可能会意外清除，从而导致可能不理想的集群配置。请参阅[临时设置迁移指南](https://www.elastic.co/guide/en/elasticsearch/reference/current/transient-settings-migration-guide.html)。

**Static**

Static settings can only be configured on an unstarted or shut down node using `elasticsearch.yml`.
静态设置只能使用`elasticsearch.yml`在未启动或关闭的节点上配置。

Static settings must be set on every relevant node in the cluster.
必须在集群中的每个相关节点上设置静态设置。
