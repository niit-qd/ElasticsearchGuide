[Install Elasticsearch with RPM](https://www.elastic.co/guide/en/elasticsearch/reference/current/rpm.html)

0. 配置源
   CentOS：[CentOS 镜像](https://developer.aliyun.com/mirror/centos/)
1. Installing from the RPM repository
    `/etc/yum.repos.d/elasticsearch.repo`

    ```repo
    [elasticsearch]
    name=Elasticsearch repository for 8.x packages
    baseurl=https://artifacts.elastic.co/packages/8.x/yum
    gpgcheck=1
    gpgkey=https://artifacts.elastic.co/GPG-KEY-elasticsearch
    enabled=0
    autorefresh=1
    type=rpm-md
    ```
    ```shell
    sudo yum install --enablerepo=elasticsearch elasticsearch
    ```

    安装完毕后会打印：

    ```log
    --------------------------- Security autoconfiguration information ------------------------------

    Authentication and authorization are enabled.
    TLS for the transport and HTTP layers is enabled and configured.

    The generated password for the elastic built-in superuser is : isXCjoe3Ag1RJIx*wsei

    If this node should join an existing cluster, you can reconfigure this with
    '/usr/share/elasticsearch/bin/elasticsearch-reconfigure-node --enrollment-token <token-here>'
    after creating an enrollment token on your existing cluster.

    You can complete the following actions at any time:

    Reset the password of the elastic built-in superuser with
    '/usr/share/elasticsearch/bin/elasticsearch-reset-password -u elastic'.

    Generate an enrollment token for Kibana instances with
    '/usr/share/elasticsearch/bin/elasticsearch-create-enrollment-token -s kibana'.

    Generate an enrollment token for Elasticsearch nodes with
    '/usr/share/elasticsearch/bin/elasticsearch-create-enrollment-token -s node'.

    -------------------------------------------------------------------------------------------------
    ### NOT starting on installation, please execute the following statements to configure elasticsearch service to start automatically using systemd
    sudo systemctl daemon-reload
    sudo systemctl enable elasticsearch.service
    ### You can start elasticsearch service by executing
    sudo systemctl start elasticsearch.service
    Verifying  : elasticsearch-8.15.1-1.x86_64
    ```
    根据提示启用以及开机启动`elasticsearch.service`。
    ``` shell
    ### NOT starting on installation, please execute the following statements to configure elasticsearch service to start automatically using systemd
    sudo systemctl daemon-reload
    sudo systemctl enable elasticsearch.service
    ### You can start elasticsearch service by executing
    sudo systemctl start elasticsearch.service
    ```
2. 密码重置与设定
    [Start Elasticsearch with security enabled](https://www.elastic.co/guide/en/elasticsearch/reference/current/rpm.html#rpm-security-configuration)
    执行上面命令之后，会启用认证授权，并为弹性内置超级用户 `elastic`生成密码。
    根据安装之后的提示，可以找到命令路径。
    操作方法参考：[elasticsearch-reset-password](https://www.elastic.co/guide/en/elasticsearch/reference/master/reset-password.html)
    注：设置密码之前，应该先启动`elasticsearch.service`。
    `elasticsearch-reset-password`的执行路径，根据安装方式有所不同。
    1. 重置密码
        1. 内建用户elastic
            会自动生产一个密码
            ```shell
            bin/elasticsearch-reset-password -u elastic
            ```
        2. 指定用户 并 同时指定密码
            `-i, --interactive`: 提示输入指定用户的密码。使用此选项显式设置密码。
            ```shell
            bin/elasticsearch-reset-password --username user1 -i
            ```
    2. 设置密码
        如上，不再赘述。
        根据提示，输入密码。
        *练习中会将内建用户`elastic`设置为同名，简化后续操作。*
        ```
        bin/elasticsearch-reset-password --username elastic -i
        # 输入y确认
        # 输入elastic作为密码
        ```
3. 配置集群
    [Reconfigure a node to join an existing cluster](https://www.elastic.co/guide/en/elasticsearch/reference/current/rpm.html#_reconfigure_a_node_to_join_an_existing_cluster_2)

    > When you install Elasticsearch, the installation process configures a single-node cluster by default. If you want a node to join an existing cluster instead, generate an enrollment token on an existing node before you start the new node for the first time.
    > 安装 Elasticsearch 时，安装过程默认配置单节点集群。如果您希望节点加入现有集群，请在首次启动新节点之前在现有节点上生成注册令牌。

    1. 在一个节点上先创建token
        On any node in your existing cluster, generate a node enrollment token:
        在现有集群中的任何节点上，生成节点注册令牌：
        ``` shell
        /usr/share/elasticsearch/bin/elasticsearch-create-enrollment-token -s node
        ```
        注：先启动elasticsearch。`systemctl start elasticsearch.service`
        示例：`101 节点`
        ``` shell
        [root@192 ~]# /usr/share/elasticsearch/bin/elasticsearch-create-enrollment-token -s node
        eyJ2ZXIiOiI4LjE0LjAiLCJhZHIiOlsiMTkyLjE2OC42NS4xMDE6OTIwMCJdLCJmZ3IiOiI1Y2IyOTE2ZWU1YzZjNWMzZDIyYjYzYWUwMjdmNmNlOWMyM2E5ZDI5ZjlhZDYzZGRmYmUxNTU5NDA1YmE0NjcxIiwia2V5Ijoib3lFZlRaSUJGZUl6Zk5uVlo5S0Y6VmM3MmR0OE5SOGlaM0VuQkZGdldpdyJ9
        ```
        该token会超期，超期后，重新执行该命令即可，创建一个新的。
    2. 复制上面得到的token
        Copy the enrollment token, which is output to your terminal.
        复制输出到您的终端的注册令牌。
    3. 在其它节点上面应用上面的token，加入集群。
        On your new Elasticsearch node, pass the enrollment token as a parameter to the elasticsearch-reconfigure-node tool:
        在新的 Elasticsearch 节点上，将注册令牌作为参数传递给 elasticsearch-reconfigure-node 工具：
        ``` shell
        /usr/share/elasticsearch/bin/elasticsearch-reconfigure-node --enrollment-token <enrollment-token>
        ```
        示例：`102、103 节点`
        ``` shell
        [root@192 ~]# /usr/share/elasticsearch/bin/elasticsearch-reconfigure-node --enrollment-token eyJ2ZXIiOiI4LjE0LjAiLCJhZHIiOlsiMTkyLjE2OC42NS4xMDE6OTIwMCJdLCJmZ3IiOiI1Y2IyOTE2ZWU1YzZjNWMzZDIyYjYzYWUwMjdmNmNlOWMyM2E5ZDI5ZjlhZDYzZGRmYmUxNTU5NDA1YmE0NjcxIiwia2V5Ijoib3lFZlRaSUJGZUl6Zk5uVlo5S0Y6VmM3MmR0OE5SOGlaM0VuQkZGdldpdyJ9

        This node will be reconfigured to join an existing cluster, using the enrollment token that you provided.
        This operation will overwrite the existing configuration. Specifically:
          - Security auto configuration will be removed from elasticsearch.yml
          - The [certs] config directory will be removed
          - Security auto configuration related secure settings will be removed from the elasticsearch.keystore
        Do you want to continue with the reconfiguration process [y/N]y
        ```
        此操作会修改与key相关的文件以及`/etc/elasticsearch/elasticsearch.yml`配置文件。
        - `etc/elasticsearch/certs/http.p12`
        - `etc/elasticsearch/certs/http_ca.crt`
        - `etc/elasticsearch/certs/transport.p12`
        - `etc/elasticsearch/elasticsearch.keystore`
        - `/etc/elasticsearch/elasticsearch.yml`
            主要修改是：
            SECURITY AUTO CONFIGURATION 相关
            1. 变更修改时间
            2. cluster.initial_master_nodes：补充已知节点。即，在当前节点的后面追加其它节点的信息。例如当前是["192.168.65.101"]，执行时，如果当前集群中只有原始节点（执行`elasticsearch-create-enrollment-token`命令的节点）101，那么执行后，其值被修改为`["192.168.65.101:9300", "192.168.65.102:9300"]`；如果已经有101和102节点，那么执行后，其值被修改为`["192.168.65.101:9300", "192.168.65.102:9300", "192.168.65.103:9300"]`。
            3. `#transport.host: 0.0.0.0`被打开，变成`transport.host: 0.0.0.0`。
            4. 注：此命令只影响当前节点中的`/etc/elasticsearch/elasticsearch.yml`中的内容，之前节点的值不影响。而且，当重新启动后，重启之前已经创建的节点不变。
    4. 如果原先的token过期或者发生变化，重新执行会报错：
        ``` log
        ERROR: Aborting enrolling to cluster. Could not communicate with the node on any of the addresses from the enrollment token. All of [192.168.65.101:9200] were attempted., with exit code 69
        ```
        重新执行``，又会报错：
        ``` log
        ERROR: Aborting enrolling to cluster. This node doesn't appear to be auto-configured for security. Expected configuration is missing from elasticsearch.yml., with exit code 64
        ```
        对比`/etc/elasticsearch`下的相关配置文件，发生了变更。例如，`/elasticsearch.yml`，`SECURITY AUTO CONFIGURATION`相关的配置项丢失。
        如果还保存有之前的配置备份，拷贝覆盖即可。
        ``` shell
        # 这里etc/elasticsearch的初始文档的备份目录
        cp -rf etc/elasticsearch/* /etc/elasticsearch/
        ```
        注：这里制作粗略处理，未细化查看具体文件。*待后续有时间再对比。*

    ---

    **几个需要注意的地方**
    1. 为了简化操作，这里禁用了防火墙。
        `systemctl stop firewalld` `systemctl disable firewalld`
        如果新增节点成功加入
    2. 部分配置
        `/etc/elasticsearch/elasticsearch.yml`
        示例：
        ``` yml
        # 集群名称
        cluster.name: my-cluster
        # 节点名称
        node.name: node-101
        ```
    3. VMware问题
        如果先在某个VMware实例中安装Elasticsearch，然后通过克隆创建其它cluster节点实例，则会报错。目前上不知解决方法。例如，已经创建了100节点，通过克隆方式创建101、102、103节点。*待后续待机解决。*

4. 系统索引的自动创建
    [Enable automatic creation of system indices](https://www.elastic.co/guide/en/elasticsearch/reference/current/rpm.html?by=history&from=kkframenew#rpm-enable-indices:~:text=using%20systemd.-,Enable%20automatic%20creation%20of%20system%20indices,-edit)

    Some commercial features automatically create indices within Elasticsearch. By default, Elasticsearch is configured to allow automatic index creation, and no additional steps are required. However, if you have disabled automatic index creation in Elasticsearch, you must configure [action.auto_create_index](https://www.elastic.co/guide/en/elasticsearch/reference/current/docs-index_.html#index-creation) in `elasticsearch.yml` to allow the commercial features to create the following indices:
    一些商业功能会在Elasticsearch中自动创建索引。默认情况下，Elasticsearch被配置为允许自动索引创建，并且不需要其他步骤。但是，如果您在Elasticsearch中已禁用自动索引创建，则必须在`elasticsearch.yml`中配置[Action.auto_create_index](https://www.elastic.co/guide/en/elasticsearch/reference/current/docs-index_.html#index-creation)允许商业功能创建以下索引：:
    ``` yml
    action.auto_create_index: .monitoring*,.watches,.triggered_watches,.watcher-history*,.ml*
    ```

5. 运行 Elasticsearch 
    Running Elasticsearch with `systemd`
    To configure Elasticsearch to start automatically when the system boots up, run the following commands:
    要配置Elasticsearch以在系统启动时自动启动，请运行以下命令：
    ``` shell
    sudo /bin/systemctl daemon-reload
    sudo /bin/systemctl enable elasticsearch.service
    ```
    Elasticsearch can be started and stopped as follows:
    Elasticsearch可以启动并停止如下：
    ``` shell
    sudo systemctl start elasticsearch.service
    sudo systemctl stop elasticsearch.service
    ```
    These commands provide no feedback as to whether Elasticsearch was started successfully or not. Instead, this information will be written in the log files located in `/var/log/elasticsearch/`.
    这些命令没有关于是否成功启动Elasticsearch的反馈。相反，此信息将写入位于`/var/log/log/elasticsearch/`的日志文件中。

    <br/>

    If you have password-protected your Elasticsearch keystore, you will need to provide `systemd` with the keystore password using a local file and systemd environment variables. This local file should be protected while it exists and may be safely deleted once Elasticsearch is up and running.
    如果您对您的Elasticsearch密钥库进行了密码保护，则需要使用本地文件和 systemd 环境变量为 `systemd` 提供密钥库密码。该本地文件在存在的同时应受到保护，并且一旦Elasticsearch启动并运行，可以安全删除。
    *这里主要是再使用keystore的情况下将keystore的密码临时保存在一个临时文件中，用完即删。*
    ``` shell
    echo "keystore_password" > /path/to/my_pwd_file.tmp
    chmod 600 /path/to/my_pwd_file.tmp
    sudo systemctl set-environment ES_KEYSTORE_PASSPHRASE_FILE=/path/to/my_pwd_file.tmp
    sudo systemctl start elasticsearch.service
    ```
    注：上面的`"keystore_password"`要替换为实际的密码，目的是创建一个存放keystore密码的文件。创建此文件后，再将此文件路径设置到环境变量中，启动`elasticsearch.service`的时候会调用此文件。*具体原理参考下面对`/usr/share/elasticsearch/bin/systemd-entrypoint`的分析。*

    <br/>

    By default the Elasticsearch service doesn’t log information in the `systemd` journal. To enable `journalctl` logging, the `--quiet` option must be removed from the `ExecStart` command line in the `elasticsearch.service` file.
    默认情况下，Elasticsearch服务未在 `systemd` Journal中记录信息。要启用 `journalctl` 日志记录，必须从 `elasticsearch.service` 文件中的`Execstart`命令行中删除  `--quiet` 选项。
    When `systemd` logging is enabled, the logging information are available using the `journalctl` commands:
    启用 `systemd` 记录时，可以使用 `journalctl` 命令提供记录信息：
    - To tail the journal:
        ``` shell
        sudo journalctl -f
        ```
    - To list journal entries for the elasticsearch service:
        ``` shell
        sudo journalctl --unit elasticsearch
        ```
    - To list journal entries for the elasticsearch service starting from a given time:
        ``` shell
        sudo journalctl --unit elasticsearch --since  "2016-10-30 18:17:16"
        ```
    Check `man journalctl` or https://www.freedesktop.org/software/systemd/man/journalctl.html for more command line options.

    > **Startup timeouts with older systemd versions**
    > By default Elasticsearch sets the `TimeoutStartSec` parameter to `systemd` to `900s`. If you are running at least version 238 of `systemd` then Elasticsearch can automatically extend the startup timeout, and will do so repeatedly until startup is complete even if it takes longer than 900s.
    > 默认情况下，Elasticsearch将`TimeOutStartSec`参数设置为`systemd`将其设置为`900秒`。如果您至少运行了`systemd`版本238，则Elasticsearch可以自动扩展启动超时，并且可以重复执行此操作，直到启动完成，即使需要超过900秒的时间。
    > Versions of `systemd` prior to 238 do not support the timeout extension mechanism and will terminate the Elasticsearch process if it has not fully started up within the configured timeout. If this happens, Elasticsearch will report in its logs that it was shut down normally a short time after it started:
    > 238之前的`systemd`版本不支持超时扩展机制，并且如果在配置的超时范围内尚未完全启动，则将终止Elasticsearch进程。如果发生这种情况，Elasticsearch将在其日志中报告说，它通常在启动后不久就被关闭：
    > ``` log
    > [2022-01-31T01:22:31,077][INFO ][o.e.n.Node               ] > [instance-0000000123] starting ...
    > ...
    > [2022-01-31T01:37:15,077][INFO ][o.e.n.Node               ] > [instance-0000000123] stopping ...
    > ```
    > However the `systemd` logs will report that the startup timed out:
    > 但是，`systemd`日志将报告该启动时间计时：
    > ``` log
    > Jan 31 01:22:30 debian systemd[1]: Starting Elasticsearch...
    > Jan 31 01:37:15 debian systemd[1]: elasticsearch.service: Start operation timed out. Terminating.
    > Jan 31 01:37:15 debian systemd[1]: elasticsearch.service: Main process exited, code=killed, status=15/TERM
    > Jan 31 01:37:15 debian systemd[1]: elasticsearch.service: Failed with result 'timeout'.
    > Jan 31 01:37:15 debian systemd[1]: Failed to start Elasticsearch.
    > ```
    > 注：该部分tip主要是说，Elasticsearch的启动超时问题受`systemd`版本的影响。当`systemd`版本大于等于238的时候，可以通过不断重置来规避`900s`限制，只要有1次启动通过即可。并建议`systemd`版本低于238时进行升级。

6. 运行检测
    [Check that Elasticsearch is running](https://www.elastic.co/guide/en/elasticsearch/reference/current/rpm.html?by=history&from=kkframenew#rpm-check-running)
    You can test that your Elasticsearch node is running by sending an HTTPS request to port `9200` on `localhost`:
    您可以通过将https请求发送到`localhost`上的端口`9200`来测试您的Elasticsearch节点正在运行：
    ``` shell
    curl --cacert /etc/elasticsearch/certs/http_ca.crt -u elastic:$ELASTIC_PASSWORD https://localhost:9200 
    ```
    > Ensure that you use `https` in your call, or the request will fail.
    > 确保您在呼叫中使用`https`，否则请求将失败。
    > - `--cacert`
    >   Path to the generated `http_ca.crt` certificate for the HTTP layer.
    >   通往HTTP层生成的`http_ca.crt`证书的路径。

    The call returns a response like this:
    呼叫返回这样的回复：
    ``` json
    {
      "name" : "Cp8oag6",
      "cluster_name" : "elasticsearch",
      "cluster_uuid" : "AT69_T_DTp-1qgIJlatQqA",
      "version" : {
        "number" : "8.15.2",
        "build_type" : "tar",
        "build_hash" : "f27399d",
        "build_flavor" : "default",
        "build_date" : "2016-03-30T09:51:41.449Z",
        "build_snapshot" : false,
        "lucene_version" : "9.11.1",
        "minimum_wire_compatibility_version" : "1.2.3",
        "minimum_index_compatibility_version" : "1.2.3"
      },
      "tagline" : "You Know, for Search"
    }
    ```
7. 配置Elasticsearch 
    The `/etc/elasticsearch` directory contains the default runtime configuration for Elasticsearch. The ownership of this directory and all contained files are set to `root:elasticsearch` on package installations.
    `/etc/elasticsearch` 目录包含 Elasticsearch 的默认运行时配置。在软件包安装中，此目录及其所包含的所有文件的所有权均设置为 `root:elasticsearch`。
    The `setgid` flag applies group permissions on the `/etc/elasticsearch` directory to ensure that Elasticsearch can read any contained files and subdirectories. All files and subdirectories inherit the `root:elasticsearch` ownership. Running commands from this directory or any subdirectories, such as the [elasticsearch-keystore tool](https://www.elastic.co/guide/en/elasticsearch/reference/current/secure-settings.html), requires `root:elasticsearch` permissions.
    `setgid` 标志将组权限应用于 `/etc/elasticsearch` 目录，以确保 Elasticsearch 可以读取任何包含的文件和子目录。所有文件和子目录都继承 `root:elasticsearch` 所有权。从此目录或任何子目录运行命令（例如 [elasticsearch-keystore 工具](https://www.elastic.co/guide/en/elasticsearch/reference/current/secure-settings.html)）需要 `root:elasticsearch` 权限。

    上面的文本主要是解释Elasticsearch的配置文件路径以及权限。
    配置文件目录路径：`/etc/elasticsearch`
    配置文件目录及子文件或目录权限：`root:elasticsearch`
    - 查看 `elasticsearch.service` 及相关配置
        ``` shell
        systemctl cat elasticsearch.service
        ```
        可知，调用的的最终命令是：
        ``` shell
        /usr/share/elasticsearch/bin/systemd-entrypoint -p ${PID_DIR}/elasticsearch.pid --quiet
        ```
        该文件还配置了其它信息，例如ES的安装目录、配置文件目录、pid目录、环境变量、工作目录、User和Group等。
        ``` service
        User=elasticsearch
        Group=elasticsearch
        ```
        指定执行`elasticsearch.service`的user和group都是`elasticsearch`。
        *注：关于`systemd`，参考[USER/GROUP IDENTITY](https://man7.org/linux/man-pages/man5/systemd.exec.5.html#USER/GROUP_IDENTITY)。*
    - 具体执行：`systemd-entrypoint `
        查看`systemd-entrypoint `
        ``` shell
        cat /usr/share/elasticsearch/bin/systemd-entrypoint
        ```
        ``` shell
        if [ -n "$ES_KEYSTORE_PASSPHRASE_FILE" ] ; then
          exec /usr/share/elasticsearch/bin/elasticsearch "$@" < "$ES_KEYSTORE_PASSPHRASE_FILE"
        else
          exec /usr/share/elasticsearch/bin/elasticsearch "$@"
        fi
        ```
        可知，实际上最终调用的还是`elasticsearch`启动程序。
        在`$ES_KEYSTORE_PASSPHRASE_FILE`环境变量存在的时候，会直接从文件路径中读取keystore的密码作为参数。

    <br/>
   
    Elasticsearch loads its configuration from the `/etc/elasticsearch/elasticsearch.yml` file by default. The format of this config file is explained in [*Configuring Elasticsearch*](https://www.elastic.co/guide/en/elasticsearch/reference/current/settings.html).
    Elasticsearch默认情况下，从`/etc/eLasticsearch/elasticsearch.yml`文件加载其配置。此配置文件的格式在[*Configuring Elasticsearch*](https://www.elastic.co/guide/en/elasticsearch/reference/current/settings.html)）中说明。

    <br/>

    The RPM also has a system configuration file (`/etc/sysconfig/elasticsearch`), which allows you to set the following parameters:
    RPM还具有系统配置文件（`/etc/sysconfig/elasticsearch`），该文件允许您设置以下参数：
    |                      |                                                                                                                                                                                                                                                                                                                                                                |
    | -------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
    | `ES_JAVA_HOME`       | Set a custom Java path to be used.                                                                                                                                                                                                                                                                                                                             |
    | `ES_PATH_CONF`       | Configuration file directory (which needs to include `elasticsearch.yml`, `jvm.options`, and `log4j2.properties` files); defaults to `/etc/elasticsearch`.                                                                                                                                                                                                     |
    | `ES_JAVA_OPTS`       | Any additional JVM system properties you may want to apply.                                                                                                                                                                                                                                                                                                    |
    | `RESTART_ON_UPGRADE` | Configure restart on package upgrade, defaults to false. This means you will have to restart your Elasticsearch instance after installing a package manually. The reason for this is to ensure, that upgrades in a cluster do not result in a continuous shard reallocation resulting in high network traffic and reducing the response times of your cluster. |

    |                      |                                                                                                                                                                                                            |
    | -------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
    | `ES_JAVA_HOME`       | 设置自定义的Java路径。                                                                                                                                                                                     |
    | `ES_PATH_CONF`       | 配置文件目录（其中需要包括`eLasticsearch.yml`，`jvm.options`和log4j2.properties files）;默认为“/etc/elasticsearch”。                                                                                       |  |
    | `ES_JAVA_OPTS`       | 您可能需要应用的任何其他JVM系统属性。                                                                                                                                                                      |
    | `RESTART_ON_UPGRADE` | 在软件包升级上配置重新启动，默认为false。这意味着您将在手动安装软件包后必须重新启动Elasticsearch实例。这样做的原因是要确保群集中的升级不会导致连续的碎片重新定位，从而导致高网络流量并减少群集的响应时间。 |
    
8. 
