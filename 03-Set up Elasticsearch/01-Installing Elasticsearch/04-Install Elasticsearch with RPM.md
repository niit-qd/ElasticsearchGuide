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
    5. 为了简化操作，这里禁用了防火墙。
        `systemctl stop firewalld` `systemctl disable firewalld`
        如果新增节点成功加入
    6. VMware问题
        如果先在某个VMware实例中安装Elasticsearch，然后通过克隆创建其它cluster节点实例，则会报错。目前上不知解决方法。例如，已经创建了100节点，通过克隆方式创建101、102、103节点。*待后续待机解决。*
4. 
