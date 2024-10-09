[Install Elasticsearch with RPM](https://www.elastic.co/guide/en/elasticsearch/reference/current/rpm.html)


---

### 配置源

CentOS：[CentOS 镜像](https://developer.aliyun.com/mirror/centos/)

---

### Installing from the RPM repository

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

---

### 密码重置与设定

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

---

### 配置集群

[Reconfigure a node to join an existing cluster](https://www.elastic.co/guide/en/elasticsearch/reference/current/rpm.html#_reconfigure_a_node_to_join_an_existing_cluster_2)

> When you install Elasticsearch, the installation process configures a single-node cluster by default. If you want a node to join an existing cluster instead, generate an enrollment token on an existing node before you start the new node for the first time.
> 安装 Elasticsearch 时，安装过程默认配置单节点集群。如果您希望节点加入现有集群，请在首次启动新节点之前在现有节点上生成注册令牌。

1. 在一个节点上先创建token
    On any node in your existing cluster, generate a node enrollment token:
    在现有集群中的任何节点上，生成节点注册令牌：
    ``` shell
    /usr/share/elasticsearch/bin/elasticsearch-create-enrollment-token -s node
    ```
    示例：`101 节点`
    ``` shell
    [root@192 ~]# /usr/share/elasticsearch/bin/elasticsearch-create-enrollment-token -s node
    eyJ2ZXIiOiI4LjE0LjAiLCJhZHIiOlsiMTkyLjE2OC42NS4xMDE6OTIwMCJdLCJmZ3IiOiI1Y2IyOTE2ZWU1YzZjNWMzZDIyYjYzYWUwMjdmNmNlOWMyM2E5ZDI5ZjlhZDYzZGRmYmUxNTU5NDA1YmE0NjcxIiwia2V5Ijoib3lFZlRaSUJGZUl6Zk5uVlo5S0Y6VmM3MmR0OE5SOGlaM0VuQkZGdldpdyJ9
    ```
    该token会超期，超期后，重新执行该命令即可，创建一个新的。

    注：先启动elasticsearch。`systemctl start elasticsearch.service`
    注意，为了执行`elasticsearch-reconfigure-node`的节点可以连接到当前节点所在的集群，在首次执行`elasticsearch-create-enrollment-token`命令的节点启动之前，首先修改`/etc/elasticsearch/elasticsearch.yml`的`network.hos`。
    请参考：[Commonly used network settings](https://www.elastic.co/guide/en/elasticsearch/reference/current/modules-network.html#common-network-settings)、[Special values for network addresses](https://www.elastic.co/guide/en/elasticsearch/reference/current/modules-network.html#network-interface-values)。
    *只要不选择`_local_`、`127.0.0.1`这种只能适配伪集群的IP就行。*
    示例：
    ``` yml
    network.host: _site_
    # 或
    network.host: 具体的IP
    # 或
    network.host: 0.0.0.0
    ```
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
        `SECURITY AUTO CONFIGURATION` 相关
        1. 执行`elasticsearch-create-enrollment-token`命令的节点
            没有变化
        2. 执行`elasticsearch-reconfigure-node`命令的节点
            删除`cluster.initial_master_nodes`、`http.host`、`transport.host`的配置和注释。
            增加执行`elasticsearch-create-enrollment-token`命令的节点的ip和端口到`discovery.seed_hosts`，例如：
            ``` yml
            # Discover existing nodes in the cluster
            discovery.seed_hosts: ["192.168.65.101:9300"]
            ```
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
    注：这里只关注了`/elasticsearch.yml`。*其它文件的变更，待后续验证。*

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
4. **目前无法解决**执行`elasticsearch-create-enrollment-token`的节点**变更IP**的问题
    1. 如果IP变更之后，并且token过期，所以需要重写创建token。
      执行之前的命令会报异常：
      ``` shell
      [root@192 ~]# /usr/share/elasticsearch/bin/elasticsearch-create-enrollment-token -s node
      06:03:52.245 [main] WARN  org.elasticsearch.common.ssl.DiagnosticTrustManager - failed to establish trust with server at [192.168.65.108]; the server provided a certificate with subject name [CN=192.168.65.101], fingerprint [c86e82e047189d58214f6afcb0f937ac46132e2c], no keyUsage and extendedKeyUsage [serverAuth]; the certificate is valid between [2024-10-02T11:38:14Z] and [2026-10-02T11:38:14Z] (current time is [2024-10-09T10:03:52.227453800Z], certificate dates are valid); the session uses cipher suite [TLS_AES_256_GCM_SHA384] and protocol [TLSv1.3]; the certificate has subject alternative names [IP:192.168.65.101,DNS:192.168.65.101,IP:0:0:0:0:0:0:0:1,IP:127.0.0.1,IP:fe80:0:0:0:48c4:f873:2fcd:1dc1,DNS:localhost]; the certificate is issued by [CN=Elasticsearch security auto-configuration HTTP CA]; the certificate is signed by (subject [CN=Elasticsearch security auto-configuration HTTP CA] fingerprint [91b71e79753bcd893a0e61855b02fbf15063a72e] {trusted issuer}) which is self-issued; the [CN=Elasticsearch security auto-configuration HTTP CA] certificate is trusted in this ssl context ([xpack.security.http.ssl (with trust configuration: Composite-Trust{JDK-trusted-certs,StoreTrustConfig{path=certs/http.p12, password=<non-empty>, type=PKCS12, algorithm=PKIX}})])
      java.security.cert.CertificateException: No subject alternative names matching IP address 192.168.65.108 found
      ```
      临时处理方案是，追加`--url参数`。注意不能使用实际的对外IP地址。
      ``` shell
      /usr/share/elasticsearch/bin/elasticsearch-create-enrollment-token -s node --url https://localhost:9200
      ```
      然后在其它节点上面修改`/elasticsearch.yml`，修改`discovery.seed_hosts`即可。
    2. 但是上述方案，只能对已经加入集群的节点有效。
    3. 对于新节点，直接执行会报异常，目前无法解决。
      ``` shell
      [root@192 ~]# /usr/share/elasticsearch/bin/elasticsearch-reconfigure-node --enrollment-token eyJ2ZXIiOiI4LjE0LjAiLCJhZHIiOlsiMTkyLjE2OC42NS4xMDE6OTIwMCJdLCJmZ3IiOiI1Y2IyOTE2ZWU1YzZjNWMzZDIyYjYzYWUwMjdmNmNlOWMyM2E5ZDI5ZjlhZDYzZGRmYmUxNTU5NDA1YmE0NjcxIiwia2V5IjoiZ24wNmNKSUJZQzhpTUlDTXZBTjQ6MEhQUVRla0xUX21wS1owQzl6ODNVQSJ9
      
      This node will be reconfigured to join an existing cluster, using the enrollment token that you provided.
      This operation will overwrite the existing configuration. Specifically:
        - Security auto configuration will be removed from elasticsearch.yml
        - The [certs] config directory will be removed
        - Security auto configuration related secure settings will be removed from the elasticsearch.keystore
      Do you want to continue with the reconfiguration process [y/N]y
      Unable to communicate with the node on https://192.168.65.101:9200/_security/enroll/node. Error was No route to host
      ERROR: Aborting enrolling to cluster. Could not communicate with the node on any of the addresses from the enrollment token. All of [192.168.65.101:9200] were attempted., with exit code 69
      ```
      此时，`/elasticsearch.yml`被删除了`BEGIN SECURITY AUTO CONFIGURATION`中的内容。
      后续重试，报新异常：
      ``` shell
      [root@192 ~]# /usr/share/elasticsearch/bin/elasticsearch-reconfigure-node --enrollment-token eyJ2ZXIiOiI4LjE0LjAiLCJhZHIiOlsiMTkyLjE2OC42NS4xMDg6OTIwMCJdLCJmZ3IiOiI1Y2IyOTE2ZWU1YzZjNWMzZDIyYjYzYWUwMjdmNmNlOWMyM2E5ZDI5ZjlhZDYzZGRmYmUxNTU5NDA1YmE0NjcxIiwia2V5IjoiWnFIWmNKSUJhZWRUdWVySzZIclY6UnJWeklzMlhRNEtSTzBhZjFYQ1ZLQSJ9
      Generates all the necessary security configuration for a node in a secured cluster
      
      Option              Description
      ------              -----------
      -E <KeyValuePair>   Configure a setting
      --enrollment-token  The enrollment token to use
      -h, --help          Show help
      -s, --silent        Show minimal output
      -v, --verbose       Show verbose output
      
      
      ERROR: Aborting enrolling to cluster. This node doesn't appear to be auto-configured for security. Expected configuration is missing from elasticsearch.yml., with exit code 64
      ```

### 系统索引的自动创建

[Enable automatic creation of system indices](https://www.elastic.co/guide/en/elasticsearch/reference/current/rpm.html?by=history&from=kkframenew#rpm-enable-indices:~:text=using%20systemd.-,Enable%20automatic%20creation%20of%20system%20indices,-edit)
 
Some commercial features automatically create indices within Elasticsearch. By default, Elasticsearch is configured to allow automatic index creation, and no additional steps are required. However, if you have disabled automatic index creation in Elasticsearch, you must configure [action.auto_create_index](https://www.elastic.co/guide/en/elasticsearch/reference/current/docs-index_.html#index-creation) in `elasticsearch.yml` to allow the commercial features to create the following indices:
一些商业功能会在Elasticsearch中自动创建索引。默认情况下，Elasticsearch被配置为允许自动索引创建，并且不需要其他步骤。但是，如果您在Elasticsearch中已禁用自动索引创建，则必须在`elasticsearch.yml`中配置[Action.auto_create_index](https://www.elastic.co/guide/en/elasticsearch/reference/current/docs-index_.html#index-creation)允许商业功能创建以下索引：:
``` yml
action.auto_create_index: .monitoring*,.watches,.triggered_watches,.watcher-history*,.ml*
```

---

### 运行 Elasticsearch 
[Running Elasticsearch with `systemd`](https://www.elastic.co/guide/en/elasticsearch/reference/current/rpm.html#rpm-running-systemd)
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

---

### 运行检测
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

---

### 配置Elasticsearch 

[Configuring Elasticsearch](https://www.elastic.co/guide/en/elasticsearch/reference/current/rpm.html#rpm-configuring)

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
    查看`ExecStart`可知，实际上最终调用的还是`elasticsearch`启动程序。
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

> Distributions that use systemd require that system resource limits be configured via systemd rather than via the `/etc/sysconfig/elasticsearch` file. See [Systemd configuration](https://www.elastic.co/guide/en/elasticsearch/reference/current/setting-system-settings.html#systemd) for more information.
> 使用 systemd 的发行版要求通过 systemd 而不是通过 `/etc/sysconfig/elasticsearch` 文件来配置系统资源限制。有关更多信息，请参阅 [Systemd 配置](https://www.elastic.co/guide/en/elasticsearch/reference/current/setting-system-settings.html#systemd)。

注：上面这个tip的意思是，对于RPM、DEB包安装器方式安装并启动的Elasticsearch服务，由于是通过`systemd`方式进行启动，而`systemd`无法继承来自于shell中系统环境变量，所以需要在原始service文件或者使用`sudo systemctl edit elasticsearch`方式重写默认值。
参考：[Environment variables](https://wiki.archlinux.org/title/Systemd/User#Environment_variables)
Units started by user instance of systemd do not inherit any of the [environment variables](https://wiki.archlinux.org/title/Environment_variables) set in places like `.bashrc` etc. 
由 systemd 用户实例启动的单元不会继承在 `.bashrc` 等位置设置的任何 [environment variables](https://wiki.archlinux.org/title/Environment_variables)。

--- 

**配置顺序分析**
根据RPM方式的启动方式分析，各配置方式的生效顺序如下：
其中，启动入口是：
``` shell
systemctl cat elasticsearch.service
```
``` service
/usr/share/elasticsearch/bin/systemd-entrypoint -p ${PID_DIR}/elasticsearch.pid --quiet
```
- `/usr/share/elasticsearch/bin/systemd-entrypoint`
  调用`exec /usr/share/elasticsearch/bin/elasticsearch`
- `exec /usr/share/elasticsearch/bin/elasticsearch`
  ``` shell
  source "`dirname "$0"`"/elasticsearch-cli
  ```
- `/usr/share/elasticsearch/bin/elasticsearch-cli`
  ``` shell
  # 先配置环境变量 ★各环境变量就是在这里配置的★
  source "`dirname "$0"`"/elasticsearch-env
  # exec 各种参数 org.elasticsearch.launcher.CliToolLauncher "$@"
  # 跟踪进去可知，实际调用的文件是：安装目录`/usr/share/elasticsearch`下`lib\cli-launcher\cli-launcher-8.15.2.jar`的`org.elasticsearch.launcher.CliToolLauncher`类。
  source "`dirname "$0"`"/elasticsearch-env
  
  exec \
    "$JAVA" \
    $CLI_JAVA_OPTS \
    -Dcli.name="$CLI_NAME" \
    -Dcli.script="$0" \
    -Dcli.libs="$CLI_LIBS" \
    -Des.path.home="$ES_HOME" \
    -Des.path.conf="$ES_PATH_CONF" \
    -Des.distribution.type="$ES_DISTRIBUTION_TYPE" \
    -cp "$LAUNCHER_CLASSPATH" \
    org.elasticsearch.launcher.CliToolLauncher \
    "$@"
  ```
- `/usr/share/elasticsearch/bin/elasticsearch-env`
  在这里完成基础配置。
  并在配置的之后校验配置是否有效之前，执行RPM的系统配置文件
  ``` shell
  source /etc/sysconfig/elasticsearch
  ```
- `/etc/sysconfig/elasticsearch`
  在这里可以重写上面的4个环境变量。
  推荐在这里修改相关配置。
由上可知，基于RPM方式安装的Elasticsearch，其配置的启用顺序优先级（由高到低）是：
- `/etc/sysconfig/elasticsearch`
- `/usr/share/elasticsearch/bin/elasticsearch-env`
  不推荐在这里修改配置，因为该文件的目的是进行逻辑处理。
- `/usr/lib/systemd/system/elasticsearch.service`
  这是服务的配置文件。
  不推荐在这里修改配置。
  如前面的描述，对于`systemd`配置的环境变量，应该使用下面的命令在`override.conf`文件中进行编辑：
  ``` shell
  sudo systemctl edit elasticsearch
  ```
  此命令创建的文件路径是：`/etc/systemd/system/elasticsearch.service.d/override.conf`。执行此命令之前，此文件不存在。
  并且，目录名`elasticsearch.service.d`中带有当前的服务名称`elasticsearch`。
  编辑类似如下内容保存：
  ``` service
  [Service]
  Environment=ES_PATH_CONF=/etc/elasticsearch_customed
  ```

注意：自定义配置文件路径，默认执行的时候，会报权限拒绝：`Permission denied`。处理方法如下：
示例：假设，自定义的配置文件路径是`/etc/sysconfig/elasticsearch_customed`。
``` shell
# copy一份默认配置
mkdir -p /etc/elasticsearch_customed
cp -r /etc/elasticsearch/* /etc/elasticsearch_customed

# 变更文件（夹）权限
# 重置文件夹访问模式
chmod 700 /etc/elasticsearch_customed
chmod g+s /etc/elasticsearch_customed
# 重置文件（夹） 注意：尽管原始的权限组是root，经验证依然会报权限问题，所以这里将group也改为elasticsearch（冒号前面）。
chown -R elasticsearch:elasticsearch /etc/elasticsearch_customed
```

> 我的选择：由于RPM、DEB安装包方式是通过`systemd`方式进行启动，所以推荐使用`sudo systemctl edit elasticsearch`的方式重写配置。

---

### 将客户端连接到 Elasticsearch

[Connect clients to Elasticsearch](https://www.elastic.co/guide/en/elasticsearch/reference/current/rpm.html?by=history&from=kkframenew#_connect_clients_to_elasticsearch_4)

When you start Elasticsearch for the first time, TLS is configured automatically for the HTTP layer. A CA certificate is generated and stored on disk at:
首次启动 Elasticsearch 时，会自动为 HTTP 层配置 TLS。生成 CA 证书并将其存储在磁盘上的以下位置：
```
/etc/elasticsearch/certs/http_ca.crt
```
The hex-encoded SHA-256 fingerprint of this certificate is also output to the terminal. Any clients that connect to Elasticsearch, such as the [Elasticsearch Clients](https://www.elastic.co/guide/en/elasticsearch/client/index.html), Beats, standalone Elastic Agents, and Logstash must validate that they trust the certificate that Elasticsearch uses for HTTPS. Fleet Server and Fleet-managed Elastic Agents are automatically configured to trust the CA certificate. Other clients can establish trust by using either the fingerprint of the CA certificate or the CA certificate itself.
此证书的十六进制编码 SHA-256 指纹也会输出到终端。任何连接到 Elasticsearch 的客户端（例如 [Elasticsearch Clients](https://www.elastic.co/guide/en/elasticsearch/client/index.html)、Beats、独立 Elastic Agent 和 Logstash）都必须验证它们是否信任 Elasticsearch 用于 HTTPS 的证书。Fleet Server 和 Fleet 管理的 Elastic Agent 会自动配置为信任 CA 证书。其他客户端可以使用 CA 证书的指纹或 CA 证书本身来建立信任。
    
If the auto-configuration process already completed, you can still obtain the fingerprint of the security certificate. You can also copy the CA certificate to your machine and configure your client to use it.
如果自动配置过程已完成，您仍然可以获取安全证书的指纹。您还可以将 CA 证书复制到您的机器并配置您的客户端以使用它。

---

### 使用CA指纹

[Use the CA fingerprint](https://www.elastic.co/guide/en/elasticsearch/reference/current/rpm.html?by=history&from=kkframenew#_use_the_ca_fingerprint_4)

Copy the fingerprint value that’s output to your terminal when Elasticsearch starts, and configure your client to use this fingerprint to establish trust when it connects to Elasticsearch.
将 Elasticsearch 启动时输出到终端的指纹值复制，并配置客户端在连接到 Elasticsearch 时使用此指纹建立信任。
示例：
``` shell
[root@192 ~]# cat /etc/elasticsearch/certs/http_ca.crt
-----BEGIN CERTIFICATE-----
MIIFWjCCA0KgAwIBAgIVALYuN0xzqw7IEF63nOT/qPzKGAiJMA0GCSqGSIb3DQEB
CwUAMDwxOjA4BgNVBAMTMUVsYXN0aWNzZWFyY2ggc2VjdXJpdHkgYXV0by1jb25m
aWd1cmF0aW9uIEhUVFAgQ0EwHhcNMjQxMDAyMTEzODEzWhcNMjcxMDAyMTEzODEz
WjA8MTowOAYDVQQDEzFFbGFzdGljc2VhcmNoIHNlY3VyaXR5IGF1dG8tY29uZmln
dXJhdGlvbiBIVFRQIENBMIICIjANBgkqhkiG9w0BAQEFAAOCAg8AMIICCgKCAgEA
lNkBokbjn/5unBI+7THlYbeFoAb05tjqZOn6oj/2Z5t7MmU96AowKpCoZp/5QOeP
NySu04Ri+PX+W62F4/pQdTCOvxUmN2Mac/8osURRDXJIXPkwc9Bt5MBiqyo/5YGN
C5HiWzzk6P3RWqZCE8yk9Ibd7I09WRkxvQPcAhcck7L7PdFJ9hffpw3FEILyraoQ
3n5cAXaHieFCkEjaKCnx+CGdmybLaPiOaQLqhxXe/MT6DnIHDoJ++RPEjydhd28C
bRyRrUskCWgUsqhAgSUDguSPPMZ5zAd2maeZQbVgKePt+Rgct1ELRAZCzTXYJ7Oc
gYvZkitvYohNYT0gfhcKy7mWd8pA+c3icx26A+fNjs9X7+dEZOwMKguXKAPzj1fZ
9pJRLAoQnpbTDlGP5OT7Y3q9tclLI65vrTmA6+PcpLTLPR0BL8pOGeQ8rWLqul8/
rAhvwge9HfqVcmkUs178QQJEC85VMOSIBf7NYZ++q1Vr7IDgCVcQWGIGsFdBTaP5
9ZcujKMrGF0w5u62HXXdTsPUx5nQhUxno4fLcyTaPvjFIISiDwe1yTZ7Q59YQKEk
9QbSU0ydzEkgXBvihxMdnZnOC7rbPsB/St298zraPu3VUCqZqP8LT2Xvko9HIzXL
/A2G9DSTOugToyZmR78dafDACpOD/ePopy9/k5llUl0CAwEAAaNTMFEwHQYDVR0O
BBYEFN454CkRC8wbHolCh8XKo9zwszkmMB8GA1UdIwQYMBaAFN454CkRC8wbHolC
h8XKo9zwszkmMA8GA1UdEwEB/wQFMAMBAf8wDQYJKoZIhvcNAQELBQADggIBAHsG
xjVJkBo8TsCOUJ8p6JHosLKKtxOMZOmh/mfPjavRNJ9w/YConB57usW4pvMA6lGb
lKXVm/GbWoxQz0EBHNIQ6hF7ZA5neJEGX97Uvov3I6TrXcRpRd5msbjiH1I7QKMe
+rjnEVgYoYUm9vXHx2Wnb/PX047XaYCnBQDO0oCcmck+bmeXkzw4TfhGz8Fn/ysT
Cd6NPnD+o3X72FQjItSVvakweweLDWejNpet+JH8MUYbuoxn8xBsarB/EdgXYZZ5
d3h8zDdsL2QDCGoD5eYkTNbALz9+V2mcHa93tZC1uHGT1FVOZWmt5lExklZV3z9L
+4dvIsLAdR1Zt79lzNU/Gmw0Gymv6OOksj4e2VwGiUgSYVI4zC4flAuD3PWZ9VAE
e/cw9sPGJ4wNqSsJEMFIX1OTpLS/TbSmCFCaIDcgf8dk98Tj6alm8bKSTJz1gxVl
rNkbqmJRCCEf5zbyOtsmfDNHSVysCtvs9c10RY9bBuxJvOle/u5a6L/9yK8gUjQm
58zmw1/itn9MeSWpHsxS7aDBTPo0lCbidIK6E0p+2fgEDZ82zfmTbAT25odXQ8aa
Eh3R2GDXjIj3+3eG9aPsT6B+0JPLAL3dKJUYOLAquhRyXWBuP/XZSXiJ3I3oGz1v
/ZgvX7fuRKBFkzDJiSvpFwvCdpJKKAJr+IMNsVaC
-----END CERTIFICATE-----
```

If the auto-configuration process already completed, you can still obtain the fingerprint of the security certificate by running the following command. The path is to the auto-generated CA certificate for the HTTP layer.
如果自动配置过程已完成，您仍然可以通过运行以下命令获取安全证书的指纹。路径是自动生成的 HTTP 层 CA 证书。
``` shell
openssl x509 -fingerprint -sha256 -in config/certs/http_ca.crt
```

The command returns the security certificate, including the fingerprint. The `issuer` should be `Elasticsearch security auto-configuration HTTP CA`.
该命令返回安全证书，包括指纹。`issuer` 应该是 `Elasticsearch security auto-configuration HTTP CA`。
``` shell
issuer= /CN=Elasticsearch security auto-configuration HTTP CA
SHA256 Fingerprint=<fingerprint>
```

示例：
``` shell
[root@192 ~]# openssl x509 -fingerprint -sha256 -in /etc/elasticsearch/certs/http_ca.crt
SHA256 Fingerprint=5C:B2:91:6E:E5:C6:C5:C3:D2:2B:63:AE:02:7F:6C:E9:C2:3A:9D:29:F9:AD:63:DD:FB:E1:55:94:05:BA:46:71
-----BEGIN CERTIFICATE-----
MIIFWjCCA0KgAwIBAgIVALYuN0xzqw7IEF63nOT/qPzKGAiJMA0GCSqGSIb3DQEB
CwUAMDwxOjA4BgNVBAMTMUVsYXN0aWNzZWFyY2ggc2VjdXJpdHkgYXV0by1jb25m
aWd1cmF0aW9uIEhUVFAgQ0EwHhcNMjQxMDAyMTEzODEzWhcNMjcxMDAyMTEzODEz
WjA8MTowOAYDVQQDEzFFbGFzdGljc2VhcmNoIHNlY3VyaXR5IGF1dG8tY29uZmln
dXJhdGlvbiBIVFRQIENBMIICIjANBgkqhkiG9w0BAQEFAAOCAg8AMIICCgKCAgEA
lNkBokbjn/5unBI+7THlYbeFoAb05tjqZOn6oj/2Z5t7MmU96AowKpCoZp/5QOeP
NySu04Ri+PX+W62F4/pQdTCOvxUmN2Mac/8osURRDXJIXPkwc9Bt5MBiqyo/5YGN
C5HiWzzk6P3RWqZCE8yk9Ibd7I09WRkxvQPcAhcck7L7PdFJ9hffpw3FEILyraoQ
3n5cAXaHieFCkEjaKCnx+CGdmybLaPiOaQLqhxXe/MT6DnIHDoJ++RPEjydhd28C
bRyRrUskCWgUsqhAgSUDguSPPMZ5zAd2maeZQbVgKePt+Rgct1ELRAZCzTXYJ7Oc
gYvZkitvYohNYT0gfhcKy7mWd8pA+c3icx26A+fNjs9X7+dEZOwMKguXKAPzj1fZ
9pJRLAoQnpbTDlGP5OT7Y3q9tclLI65vrTmA6+PcpLTLPR0BL8pOGeQ8rWLqul8/
rAhvwge9HfqVcmkUs178QQJEC85VMOSIBf7NYZ++q1Vr7IDgCVcQWGIGsFdBTaP5
9ZcujKMrGF0w5u62HXXdTsPUx5nQhUxno4fLcyTaPvjFIISiDwe1yTZ7Q59YQKEk
9QbSU0ydzEkgXBvihxMdnZnOC7rbPsB/St298zraPu3VUCqZqP8LT2Xvko9HIzXL
/A2G9DSTOugToyZmR78dafDACpOD/ePopy9/k5llUl0CAwEAAaNTMFEwHQYDVR0O
BBYEFN454CkRC8wbHolCh8XKo9zwszkmMB8GA1UdIwQYMBaAFN454CkRC8wbHolC
h8XKo9zwszkmMA8GA1UdEwEB/wQFMAMBAf8wDQYJKoZIhvcNAQELBQADggIBAHsG
xjVJkBo8TsCOUJ8p6JHosLKKtxOMZOmh/mfPjavRNJ9w/YConB57usW4pvMA6lGb
lKXVm/GbWoxQz0EBHNIQ6hF7ZA5neJEGX97Uvov3I6TrXcRpRd5msbjiH1I7QKMe
+rjnEVgYoYUm9vXHx2Wnb/PX047XaYCnBQDO0oCcmck+bmeXkzw4TfhGz8Fn/ysT
Cd6NPnD+o3X72FQjItSVvakweweLDWejNpet+JH8MUYbuoxn8xBsarB/EdgXYZZ5
d3h8zDdsL2QDCGoD5eYkTNbALz9+V2mcHa93tZC1uHGT1FVOZWmt5lExklZV3z9L
+4dvIsLAdR1Zt79lzNU/Gmw0Gymv6OOksj4e2VwGiUgSYVI4zC4flAuD3PWZ9VAE
e/cw9sPGJ4wNqSsJEMFIX1OTpLS/TbSmCFCaIDcgf8dk98Tj6alm8bKSTJz1gxVl
rNkbqmJRCCEf5zbyOtsmfDNHSVysCtvs9c10RY9bBuxJvOle/u5a6L/9yK8gUjQm
58zmw1/itn9MeSWpHsxS7aDBTPo0lCbidIK6E0p+2fgEDZ82zfmTbAT25odXQ8aa
Eh3R2GDXjIj3+3eG9aPsT6B+0JPLAL3dKJUYOLAquhRyXWBuP/XZSXiJ3I3oGz1v
/ZgvX7fuRKBFkzDJiSvpFwvCdpJKKAJr+IMNsVaC
-----END CERTIFICATE-----
```

---
### 使用 CA 证书

[Use the CA certificate](https://www.elastic.co/guide/en/elasticsearch/reference/current/rpm.html?by=history&from=kkframenew#_use_the_ca_certificate_4)

If your library doesn’t support a method of validating the fingerprint, the auto-generated CA certificate is created in the following directory on each Elasticsearch node:
如果您的库不支持验证指纹的方法，则自动生成的 CA 证书将在每个 Elasticsearch 节点上的以下目录中创建：
```
/etc/elasticsearch/certs/http_ca.crt
```

Copy the `http_ca.crt` file to your machine and configure your client to use this certificate to establish trust when it connects to Elasticsearch.
将 `http_ca.crt` 文件复制到您的机器，并配置您的客户端以在连接到 Elasticsearch 时使用此证书建立信任。

---

### RPM 的目录布局

[Directory layout of RPM](https://www.elastic.co/guide/en/elasticsearch/reference/current/rpm.html?by=history&from=kkframenew#rpm-layout)

The RPM places config files, logs, and the data directory in the appropriate locations for an RPM-based system:
RPM 将配置文件、日志和数据目录放置在基于 RPM 的系统的适当位置：

| Type        | Description                                                                                                                                                          | Default Location                   | Setting                                                                                                             |
| ----------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ---------------------------------- | ------------------------------------------------------------------------------------------------------------------- |
| **home**    | Elasticsearch home directory or `$ES_HOME`                                                                                                                           | `/usr/share/elasticsearch`         |                                                                                                                     |
| **bin**     | Binary scripts including `elasticsearch` to start a node and `elasticsearch-plugin` to install plugins                                                               | `/usr/share/elasticsearch/bin`     |                                                                                                                     |
| **conf**    | Configuration files including `elasticsearch.yml`                                                                                                                    | `/etc/elasticsearch`               | [ES_PATH_CONF](https://www.elastic.co/guide/en/elasticsearch/reference/current/settings.html#config-files-location) |
| **conf**    | Environment variables including heap size, file descriptors.                                                                                                         | `/etc/sysconfig/elasticsearch`     |                                                                                                                     |
| **conf**    | Generated TLS keys and certificates for the transport and http layer.                                                                                                | `/etc/elasticsearch/certs`         |                                                                                                                     |
| **data**    | The location of the data files of each index / shard allocated on the node.                                                                                          | `/var/lib/elasticsearch`           | `path.data`                                                                                                         |
| **jdk**     | The bundled Java Development Kit used to run Elasticsearch. Can be overridden by setting the `ES_JAVA_HOME` environment variable in `/etc/sysconfig/elasticsearch`.  | `/usr/share/elasticsearch/jdk`     |                                                                                                                     |
| **logs**    | Log files location.                                                                                                                                                  | `/var/log/elasticsearch`           | `path.logs`                                                                                                         |
| **plugins** | Plugin files location. Each plugin will be contained in a subdirectory.                                                                                              | `/usr/share/elasticsearch/plugins` |                                                                                                                     |
| **repo**    | Shared file system repository locations. Can hold multiple locations. A file system repository can be placed in to any subdirectory of any directory specified here. | Not configured                     | `path.repo`                                                                                                         |
注：该表用于基于使用rpm方式安装的Elasticsearch。

---

### 安全证书和密钥

[Security certificates and keys](https://www.elastic.co/guide/en/elasticsearch/reference/current/rpm.html?by=history&from=kkframenew#_security_certificates_and_keys_3)

When you install Elasticsearch, the following certificates and keys are generated in the Elasticsearch configuration directory, which are used to connect a Kibana instance to your secured Elasticsearch cluster and to encrypt internode communication. The files are listed here for reference.
安装 Elasticsearch 时，Elasticsearch 配置目录中会生成以下证书和密钥，用于将 Kibana 实例连接到安全的 Elasticsearch 集群并加密节点间通信。此处列出了这些文件以供参考。
`/etc/elasticsearch/certs`
- `http_ca.crt`
    The CA certificate that is used to sign the certificates for the HTTP layer of this Elasticsearch cluster.
    用于签署此 Elasticsearch 集群的 HTTP 层证书的 CA 证书。
- `http.p12`
    Keystore that contains the key and certificate for the HTTP layer for this node.
    包含此节点的 HTTP 层的密钥和证书的密钥库。
- `transport.p12`
    Keystore that contains the key and certificate for the transport layer for all the nodes in your cluster.
    包含集群中所有节点的传输层的密钥和证书的密钥库。
    
`http.p12` and `transport.p12` are password-protected PKCS#12 keystores. Elasticsearch stores the passwords for these keystores as [secure settings](https://www.elastic.co/guide/en/elasticsearch/reference/current/secure-settings.html). To retrieve the passwords so that you can inspect or change the keystore contents, use the [bin/elasticsearch-keystore](https://www.elastic.co/guide/en/elasticsearch/reference/current/elasticsearch-keystore.html) tool.
`http.p12` 和 `transport.p12` 是受密码保护的 PKCS#12 密钥库。Elasticsearch 将这些密钥库的密码存储为 [secure settings](https://www.elastic.co/guide/en/elasticsearch/reference/current/secure-settings.html)。要检索密码以便检查或更改密钥库内容，请使用 [bin/elasticsearch-keystore](https://www.elastic.co/guide/en/elasticsearch/reference/current/elasticsearch-keystore.html) 工具。

Use the following command to retrieve the password for `http.p12`:
使用以下命令检索“http.p12”的密码：
``` shell
bin/elasticsearch-keystore show xpack.security.http.ssl.keystore.secure_password
```

Use the following command to retrieve the password for `transport.p12`:
使用以下命令检索 `transport.p12` 的密码：
``` shell
bin/elasticsearch-keystore show xpack.security.transport.ssl.keystore.secure_password
```

