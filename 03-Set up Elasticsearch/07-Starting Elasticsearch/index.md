[TOC]

# [Starting Elasticsearch | 启动 Elasticsearch](https://www.elastic.co/guide/en/elasticsearch/reference/current/starting-elasticsearch.html)

The method for starting Elasticsearch varies depending on how you installed it.
启动 Elasticsearch 的方法取决于您的安装方式。


---

## [Archive packages (.tar.gz)| 存档包（.tar.gz）](https://www.elastic.co/guide/en/elasticsearch/reference/current/starting-elasticsearch.html#start-targz)

If you installed Elasticsearch with a `.tar.gz` package, you can start Elasticsearch from the command line.
如果您使用 `.tar.gz` 包安装了 Elasticsearch，则可以从命令行启动 Elasticsearch。

---

### [Run Elasticsearch from the command line | 从命令行运行 Elasticsearch](https://www.elastic.co/guide/en/elasticsearch/reference/current/starting-elasticsearch.html#_run_elasticsearch_from_the_command_line)


Run the following command to start Elasticsearch from the command line:
运行以下命令从命令行启动 Elasticsearch：

``` shell
./bin/elasticsearch
```

When starting Elasticsearch for the first time, security features are enabled and configured by default. The following security configuration occurs automatically:
首次启动 Elasticsearch 时，安全功能已默认启用并配置。以下安全配置会自动发生：

- Authentication and authorization are enabled, and a password is generated for the elastic built-in superuser.
  启用身份验证和授权，并为 elastic 内置超级用户生成密码。
- Certificates and keys for TLS are generated for the transport and HTTP layer, and TLS is enabled and configured with these keys and certificates.
  为传输层和 HTTP 层生成 TLS 的证书和密钥，并使用这些密钥和证书启用和配置 TLS。
- An enrollment token is generated for Kibana, which is valid for 30 minutes.
  为 Kibana 生成一个注册令牌，有效期为 30 分钟。

The password for the `elastic` user and the enrollment token for Kibana are output to your terminal.
`elastic` 用户的密码和 Kibana 的注册令牌将输出到您的终端。

We recommend storing the `elastic` password as an environment variable in your shell. Example:
我们建议将`elastic`密码作为环境变量存储在您的 shell 中。例如：

``` shell
export ELASTIC_PASSWORD="your_password"
```

If you have password-protected the Elasticsearch keystore, you will be prompted to enter the keystore’s password. See [Secure settings](https://www.elastic.co/guide/en/elasticsearch/reference/current/secure-settings.html) for more details.
如果您已对 Elasticsearch 密钥库设置了密码保护，系统将提示您输入密钥库的密码。有关更多详细信息，请参阅 [安全设置](https://www.elastic.co/guide/en/elasticsearch/reference/current/secure-settings.html)。

By default Elasticsearch prints its logs to the console (`stdout`) and to the `<cluster name>.log` file within the [logs directory](https://www.elastic.co/guide/en/elasticsearch/reference/current/important-settings.html#path-settings). Elasticsearch logs some information while it is starting, but after it has finished initializing it will continue to run in the foreground and won’t log anything further until something happens that is worth recording. While Elasticsearch is running you can interact with it through its HTTP interface which is on port ``` by default.
默认情况下，Elasticsearch 会将其日志打印到控制台 (`stdout`) 和 [logs 目录](https://www.elastic.co/guide/en/elasticsearch/reference/current/important-settings.html#path-settings) 中的 `<cluster name>.log` 文件中。Elasticsearch 在启动时会记录一些信息，但在完成初始化后，它将继续在前台运行，并且不会再记录任何内容，直到发生值得记录的事情。Elasticsearch 运行时，您可以通过其 HTTP 接口与其交互，默认情况下该接口位于端口 ``` 上。

To stop Elasticsearch, press `Ctrl-C`.
要停止 Elasticsearch，请按`Ctrl-C`。

> **NOTE**
> All scripts packaged with Elasticsearch require a version of Bash that supports arrays and assume that Bash is available at `/bin/bash`. As such, Bash should be available at this path either directly or via a symbolic link.
> 使用 Elasticsearch 打包的所有脚本都需要支持数组的 Bash 版本，并假定 Bash 可在`/bin/bash`中使用。因此，Bash 应该可在此路径上直接使用或通过符号链接使用。

---

### [Enroll nodes in an existing cluster | 在现有集群中注册节点](https://www.elastic.co/guide/en/elasticsearch/reference/current/starting-elasticsearch.html#_enroll_nodes_in_an_existing_cluster_3)

When Elasticsearch starts for the first time, the security auto-configuration process binds the HTTP layer to `0.0.0.0`, but only binds the transport layer to localhost. This intended behavior ensures that you can start a single-node cluster with security enabled by default without any additional configuration.
当 Elasticsearch 首次启动时，安全自动配置过程会将 HTTP 层绑定到 `0.0.0.0`，但仅将传输层绑定到 localhost。此预期行为可确保您无需进行任何额外配置即可启动默认启用安全性的单节点集群。

Before enrolling a new node, additional actions such as binding to an address other than `localhost` or satisfying bootstrap checks are typically necessary in production clusters. During that time, an auto-generated enrollment token could expire, which is why enrollment tokens aren’t generated automatically.
在注册新节点之前，在生产集群中通常需要执行其他操作，例如绑定到除`localhost`以外的地址或满足引导检查。在此期间，自动生成的注册令牌可能会过期，这就是注册令牌不会自动生成的原因。

Additionally, only nodes on the same host can join the cluster without additional configuration. If you want nodes from another host to join your cluster, you need to set `transport.host` to a [supported value](https://www.elastic.co/guide/en/elasticsearch/reference/8.17/modules-network.html#network-interface-values) (such as uncommenting the suggested value of `0.0.0.0`), or an IP address that’s bound to an interface where other hosts can reach it. Refer to [transport settings](https://www.elastic.co/guide/en/elasticsearch/reference/8.17/modules-network.html#transport-settings) for more information.
此外，只有同一主机上的节点才能加入集群，而无需额外配置。如果您希望其他主机上的节点加入您的集群，则需要将`transport.host`设置为[支持的值](https://www.elastic.co/guide/en/elasticsearch/reference/8.17/modules-network.html#network-interface-values)（例如取消注释建议值`0.0.0.0`），或设置为绑定到其他主机可以访问的接口的 IP 地址。有关更多信息，请参阅[传输设置](https://www.elastic.co/guide/en/elasticsearch/reference/8.17/modules-network.html#transport-settings)。

To enroll new nodes in your cluster, create an enrollment token with the `elasticsearch-create-enrollment-token` tool on any existing node in your cluster. You can then start a new node with the `--enrollment-token` parameter so that it joins an existing cluster.
要将新节点注册到集群中，请使用`elasticsearch-create-enrollment-token`工具在集群中任何现有节点上创建注册令牌。然后，您可以使用`--enrollment-token`参数启动新节点，以便其加入现有集群。

1. In a separate terminal from where Elasticsearch is running, navigate to the directory where you installed Elasticsearch and run the [elasticsearch-create-enrollment-token](https://www.elastic.co/guide/en/elasticsearch/reference/current/create-enrollment-token.html) tool to generate an enrollment token for your new nodes.
   在与 Elasticsearch 运行位置不同的终端中，导航到安装 Elasticsearch 的目录并运行 [elasticsearch-create-enrollment-token](https://www.elastic.co/guide/en/elasticsearch/reference/current/create-enrollment-token.html) 工具为新节点生成注册令牌。

   ``` shell
   bin/elasticsearch-create-enrollment-token -s node
   ```

   Copy the enrollment token, which you’ll use to enroll new nodes with your Elasticsearch cluster.
   复制注册令牌，您将使用该令牌在 Elasticsearch 集群中注册新节点。
2. From the installation directory of your new node, start Elasticsearch and pass the enrollment token with the `--enrollment-token` parameter.
   从新节点的安装目录启动 Elasticsearch 并使用 `--enrollment-token` 参数传递注册令牌。

   ``` shell
   bin/elasticsearch --enrollment-token <enrollment-token>
   ```

   Elasticsearch automatically generates certificates and keys in the following directory:
   Elasticsearch 在以下目录中自动生成证书和密钥：

   ```
   config/certs
   ```
3. Repeat the previous step for any new nodes that you want to enroll.
   对您想要注册的任何新节点重复上一步。

---

### [Run as a daemon | 作为守护进程运行](https://www.elastic.co/guide/en/elasticsearch/reference/current/starting-elasticsearch.html#_run_as_a_daemon)

To run Elasticsearch as a daemon, specify `-d` on the command line, and record the process ID in a file using the `-p` option:
要将 Elasticsearch 作为守护进程运行，请在命令行上指定 `-d`，并使用 `-p` 选项将进程 ID 记录在文件中：

``` shell
./bin/elasticsearch -d -p pid
```

If you have password-protected the Elasticsearch keystore, you will be prompted to enter the keystore’s password. See [Secure settings](https://www.elastic.co/guide/en/elasticsearch/reference/current/secure-settings.html) for more details.
如果您已对 Elasticsearch 密钥库设置了密码保护，系统将提示您输入密钥库的密码。有关更多详细信息，请参阅 [安全设置](https://www.elastic.co/guide/en/elasticsearch/reference/current/secure-settings.html)。

Log messages can be found in the `$ES_HOME/logs/` directory.
日志消息可以在`$ES_HOME/logs/`目录中找到。

To shut down Elasticsearch, kill the process ID recorded in the `pid` file:
要关闭 Elasticsearch，请终止 `pid` 文件中记录的进程 ID：

``` shell
pkill -F pid
```

> **NOTE**
> The Elasticsearch `.tar.gz` package does not include the `systemd` module. To manage Elasticsearch as a service, use the [Debian](https://www.elastic.co/guide/en/elasticsearch/reference/current/starting-elasticsearch.html#start-deb) or [RPM](https://www.elastic.co/guide/en/elasticsearch/reference/current/starting-elasticsearch.html#start-rpm) package instead.


---

## [Archive packages (.zip) | 存档包 (.zip)](https://www.elastic.co/guide/en/elasticsearch/reference/current/starting-elasticsearch.html#start-zip)

If you installed Elasticsearch on Windows with a `.zip` package, you can start Elasticsearch from the command line. If you want Elasticsearch to start automatically at boot time without any user interaction, [install Elasticsearch as a service](https://www.elastic.co/guide/en/elasticsearch/reference/current/zip-windows.html#windows-service).
如果您在 Windows 上使用 `.zip` 包安装了 Elasticsearch，则可以从命令行启动 Elasticsearch。如果您希望 Elasticsearch 在启动时自动启动而无需任何用户交互，请[将 Elasticsearch 安装为服务](https://www.elastic.co/guide/en/elasticsearch/reference/current/zip-windows.html#windows-service)。

---

### [Run Elasticsearch from the command line | 从命令行运行 Elasticsearch](https://www.elastic.co/guide/en/elasticsearch/reference/current/starting-elasticsearch.html#_run_elasticsearch_from_the_command_line_2)

Run the following command to start Elasticsearch from the command line:
运行以下命令从命令行启动 Elasticsearch：

``` shell
.\bin\elasticsearch.bat
```

When starting Elasticsearch for the first time, security features are enabled and configured by default. The following security configuration occurs automatically:
首次启动 Elasticsearch 时，安全功能已默认启用并配置。以下安全配置会自动发生：

- Authentication and authorization are enabled, and a password is generated for the `elastic` built-in superuser.
  启用身份验证和授权，并为`elastic`内置超级用户生成密码。
- Certificates and keys for TLS are generated for the transport and HTTP layer, and TLS is enabled and configured with these keys and certificates.
  为传输层和 HTTP 层生成 TLS 的证书和密钥，并使用这些密钥和证书启用和配置 TLS。
- An enrollment token is generated for Kibana, which is valid for 30 minutes.
  为 Kibana 生成一个注册令牌，有效期为 30 分钟。

The password for the `elastic` user and the enrollment token for Kibana are output to your terminal.
`elastic` 用户的密码和 Kibana 的注册令牌将输出到您的终端。

We recommend storing the `elastic` password as an environment variable in your shell. Example:
我们建议将`elastic`密码作为环境变量存储在您的 shell 中。例如：

``` shell
$ELASTIC_PASSWORD = "your_password"
```

If you have password-protected the Elasticsearch keystore, you will be prompted to enter the keystore’s password. See [Secure settings](https://www.elastic.co/guide/en/elasticsearch/reference/current/secure-settings.html) for more details.
如果您已对 Elasticsearch 密钥库设置了密码保护，系统将提示您输入密钥库的密码。有关更多详细信息，请参阅 [安全设置](https://www.elastic.co/guide/en/elasticsearch/reference/current/secure-settings.html)。

By default Elasticsearch prints its logs to the console (`STDOUT`) and to the `<cluster name>.log` file within the [logs directory](https://www.elastic.co/guide/en/elasticsearch/reference/current/important-settings.html#path-settings). Elasticsearch logs some information while it is starting, but after it has finished initializing it will continue to run in the foreground and won’t log anything further until something happens that is worth recording. While Elasticsearch is running you can interact with it through its HTTP interface which is on port `9200` by default.
默认情况下，Elasticsearch 会将其日志打印到控制台 (`STDOUT`) 和 [logs 目录](https://www.elastic.co/guide/en/elasticsearch/reference/current/important-settings.html#path-settings) 中的 `<cluster name>.log` 文件中。Elasticsearch 在启动时会记录一些信息，但在完成初始化后，它将继续在前台运行，并且不会再记录任何内容，直到发生值得记录的事情。Elasticsearch 运行时，您可以通过其 HTTP 接口与其交互，默认情况下该接口位于端口 `9200` 上。

To stop Elasticsearch, press `Ctrl-C`.
要停止 Elasticsearch，请按`Ctrl-C`。

---

#### [Enroll nodes in an existing cluster | 在现有集群中注册节点](https://www.elastic.co/guide/en/elasticsearch/reference/current/starting-elasticsearch.html#_enroll_nodes_in_an_existing_cluster_4)

When Elasticsearch starts for the first time, the security auto-configuration process binds the HTTP layer to `0.0.0.0`, but only binds the transport layer to localhost. This intended behavior ensures that you can start a single-node cluster with security enabled by default without any additional configuration.
当 Elasticsearch 首次启动时，安全自动配置过程会将 HTTP 层绑定到 `0.0.0.0`，但仅将传输层绑定到 localhost。此预期行为可确保您无需进行任何额外配置即可启动默认启用安全性的单节点集群。

Before enrolling a new node, additional actions such as binding to an address other than `localhost` or satisfying bootstrap checks are typically necessary in production clusters. During that time, an auto-generated enrollment token could expire, which is why enrollment tokens aren’t generated automatically.
在注册新节点之前，在生产集群中通常需要执行其他操作，例如绑定到除`localhost`以外的地址或满足引导检查。在此期间，自动生成的注册令牌可能会过期，这就是注册令牌不会自动生成的原因。

Additionally, only nodes on the same host can join the cluster without additional configuration. If you want nodes from another host to join your cluster, you need to set `transport.host` to a [supported value](https://www.elastic.co/guide/en/elasticsearch/reference/8.17/modules-network.html#network-interface-values) (such as uncommenting the suggested value of `0.0.0.0`), or an IP address that’s bound to an interface where other hosts can reach it. Refer to [transport settings](https://www.elastic.co/guide/en/elasticsearch/reference/8.17/modules-network.html#transport-settings) for more information.
此外，只有同一主机上的节点才能加入集群，而无需额外配置。如果您希望其他主机上的节点加入您的集群，则需要将`transport.host`设置为[支持的值](https://www.elastic.co/guide/en/elasticsearch/reference/8.17/modules-network.html#network-interface-values)（例如取消注释建议值`0.0.0.0`），或设置为绑定到其他主机可以访问的接口的 IP 地址。有关更多信息，请参阅[传输设置](https://www.elastic.co/guide/en/elasticsearch/reference/8.17/modules-network.html#transport-settings)。


To enroll new nodes in your cluster, create an enrollment token with the `elasticsearch-create-enrollment-token` tool on any existing node in your cluster. You can then start a new node with the `--enrollment-token` parameter so that it joins an existing cluster.
要将新节点注册到集群中，请使用`elasticsearch-create-enrollment-token`工具在集群中任何现有节点上创建注册令牌。然后，您可以使用`--enrollment-token`参数启动新节点，以便其加入现有集群。

1. In a separate terminal from where Elasticsearch is running, navigate to the directory where you installed Elasticsearch and run the [elasticsearch-create-enrollment-token](https://www.elastic.co/guide/en/elasticsearch/reference/current/create-enrollment-token.html) tool to generate an enrollment token for your new nodes.
   在与 Elasticsearch 运行位置不同的终端中，导航到安装 Elasticsearch 的目录并运行 [elasticsearch-create-enrollment-token](https://www.elastic.co/guide/en/elasticsearch/reference/current/create-enrollment-token.html) 工具为新节点生成注册令牌。

   ```shell
   bin\elasticsearch-create-enrollment-token -s node
   ```
2. From the installation directory of your new node, start Elasticsearch and pass the enrollment token with the `--enrollment-token` parameter.
   从新节点的安装目录启动 Elasticsearch 并使用 `--enrollment-token` 参数传递注册令牌。

   ``` shell
   bin\elasticsearch --enrollment-token <enrollment-token>
   ```

   Elasticsearch automatically generates certificates and keys in the following directory:
   Elasticsearch 在以下目录中自动生成证书和密钥：

   ```
   config\certs
   ```
3. Repeat the previous step for any new nodes that you want to enroll.
   对您想要注册的任何新节点重复上一步。


---

## [Debian packages | Debian 软件包](https://www.elastic.co/guide/en/elasticsearch/reference/current/starting-elasticsearch.html#start-deb)

---

### [Running Elasticsearch with `systemd` | 使用 `systemd` 运行 Elasticsearch](https://www.elastic.co/guide/en/elasticsearch/reference/current/starting-elasticsearch.html#start-es-deb-systemd)

To configure Elasticsearch to start automatically when the system boots up, run the following commands:
要将 Elasticsearch 配置为在系统启动时自动启动，请运行以下命令：

``` shell
sudo /bin/systemctl daemon-reload
sudo /bin/systemctl enable elasticsearch.service
```

Elasticsearch can be started and stopped as follows:
Elasticsearch 可以按如下方式启动和停止：

``` shell
sudo systemctl start elasticsearch.service
sudo systemctl stop elasticsearch.service
```

These commands provide no feedback as to whether Elasticsearch was started successfully or not. Instead, this information will be written in the log files located in `/var/log/elasticsearch/`.
这些命令不提供有关 Elasticsearch 是否启动成功的反馈。相反，此信息将写入位于`/var/log/elasticsearch/`的日志文件中。

If you have password-protected your Elasticsearch keystore, you will need to provide `systemd` with the keystore password using a local file and systemd environment variables. This local file should be protected while it exists and may be safely deleted once Elasticsearch is up and running.
如果您已对 Elasticsearch 密钥库进行密码保护，则需要使用本地文件和 systemd 环境变量向`systemd`提供密钥库密码。此本地文件应在存在时受到保护，并且可以在 Elasticsearch 启动并运行后安全删除。

``` shell
echo "keystore_password" > /path/to/my_pwd_file.tmp
chmod 600 /path/to/my_pwd_file.tmp
sudo systemctl set-environment ES_KEYSTORE_PASSPHRASE_FILE=/path/to/my_pwd_file.tmp
sudo systemctl start elasticsearch.service
```

By default the Elasticsearch service doesn’t log information in the `systemd` journal. To enable `journalctl` logging, the `--quiet` option must be removed from the ExecStart command line in the `elasticsearch.service` file.
默认情况下，Elasticsearch 服务不会在`systemd`日志中记录信息。要启用`journalctl`日志记录，必须从`elasticsearch.service`文件中的 ExecStart 命令行中删除`--quiet`选项。

When `systemd` logging is enabled, the logging information are available using the `journalctl` commands:
当启用`systemd`日志记录时，可以使用`journalctl`命令获取日志信息：

To tail the journal:
跟踪日志：

``` shell
sudo journalctl -f
```

To list journal entries for the elasticsearch service:
列出 elasticsearch 服务的日记帐分录：

``` shell
sudo journalctl --unit elasticsearch
```

To list journal entries for the elasticsearch service starting from a given time:
列出从给定时间开始的 elasticsearch 服务的日记帐分录：

``` shell
sudo journalctl --unit elasticsearch --since  "2016-10-30 18:17:16"
```

Check `man journalctl` or https://www.freedesktop.org/software/systemd/man/journalctl.html for more command line options.
检查`man journalctl`或https://www.freedesktop.org/software/systemd/man/journalctl.html以获取更多命令行选项。

> **TIP**
> **Startup timeouts with older `systemd` versions**
> **旧版 `systemd` 的启动超时**
> 
> By default Elasticsearch sets the `TimeoutStartSec` parameter to `systemd` to `900s`. If you are running at least version 238 of `systemd` then Elasticsearch can automatically extend the startup timeout, and will do so repeatedly until startup is complete even if it takes longer than 900s.
> 默认情况下，Elasticsearch 将 `TimeoutStartSec` 参数设置为 `systemd`，设置为 `900s`。如果您运行的是 `systemd` 的 238 以上版本，那么 Elasticsearch 可以自动延长启动超时时间，并且会重复执行此操作，直到启动完成，即使启动时间超过 900 秒。
>
> Versions of `systemd` prior to 238 do not support the timeout extension mechanism and will terminate the Elasticsearch process if it has not fully started up within the configured timeout. If this happens, Elasticsearch will report in its logs that it was shut down normally a short time after it started:
> 238 之前的 `systemd` 版本不支持超时延长机制，如果 Elasticsearch 进程在配置的超时时间内未完全启动，则会终止该进程。如果发生这种情况，Elasticsearch 将在其日志中报告它在启动后不久就正常关闭：
>
> ``` log
> [2022-01-31T01:22:31,077][INFO ][o.e.n.Node               ] [instance-0000000123] starting ...
> ...
> [2022-01-31T01:37:15,077][INFO ][o.e.n.Node               ] [instance-0000000123] stopping ...
> ```
>
> However the `systemd` logs will report that the startup timed out:
> 然而，`systemd` 日志会报告启动超时：
>
> ``` log
> Jan 31 01:22:30 debian systemd[1]: Starting Elasticsearch...
> Jan 31 01:37:15 debian systemd[1]: elasticsearch.service: Start operation timed out. Terminating.
> Jan 31 01:37:15 debian systemd[1]: elasticsearch.service: Main process exited, code=killed, > status=15/TERM
> Jan 31 01:37:15 debian systemd[1]: elasticsearch.service: Failed with result 'timeout'.
> Jan 31 01:37:15 debian systemd[1]: Failed to start Elasticsearch.
> ```
> 
> To avoid this, upgrade your `systemd` to at least version 238. You can also temporarily work around the problem by extending the `TimeoutStartSec` parameter.
> 为了避免这种情况，请将您的`systemd`升级到至少 238 版本。您也可以通过扩展`TimeoutStartSec`参数来暂时解决此问题。


---

## [Docker images | Docker 镜像](https://www.elastic.co/guide/en/elasticsearch/reference/current/starting-elasticsearch.html#start-docker)

If you installed a Docker image, you can start Elasticsearch from the command line. There are different methods depending on whether you’re using development mode or production mode. See [Run Elasticsearch in Docker](https://www.elastic.co/guide/en/elasticsearch/reference/current/docker.html#docker-cli-run-dev-mode).
如果您安装了 Docker 映像，则可以从命令行启动 Elasticsearch。根据您使用的是开发模式还是生产模式，方法有所不同。请参阅[在 Docker 中运行 Elasticsearch](https://www.elastic.co/guide/en/elasticsearch/reference/current/docker.html#docker-cli-run-dev-mode)。


---

## [RPM packages | RPM 软件包](https://www.elastic.co/guide/en/elasticsearch/reference/current/starting-elasticsearch.html#start-rpm)

---

### [Running Elasticsearch with `systemd` | 使用 `systemd` 运行 Elasticsearch](https://www.elastic.co/guide/en/elasticsearch/reference/current/starting-elasticsearch.html#start-es-rpm-systemd)

To configure Elasticsearch to start automatically when the system boots up, run the following commands:
要将 Elasticsearch 配置为在系统启动时自动启动，请运行以下命令：

``` shell
sudo /bin/systemctl daemon-reload
sudo /bin/systemctl enable elasticsearch.service
```

Elasticsearch can be started and stopped as follows:
Elasticsearch 可以按如下方式启动和停止：

``` shell
sudo systemctl start elasticsearch.service
sudo systemctl stop elasticsearch.service
```

These commands provide no feedback as to whether Elasticsearch was started successfully or not. Instead, this information will be written in the log files located in `/var/log/elasticsearch/`.
这些命令不提供有关 Elasticsearch 是否启动成功的反馈。相反，此信息将写入位于`/var/log/elasticsearch/`的日志文件中。

If you have password-protected your Elasticsearch keystore, you will need to provide `systemd` with the keystore password using a local file and systemd environment variables. This local file should be protected while it exists and may be safely deleted once Elasticsearch is up and running.
如果您已对 Elasticsearch 密钥库进行密码保护，则需要使用本地文件和 systemd 环境变量向`systemd`提供密钥库密码。此本地文件应在存在时受到保护，并且可以在 Elasticsearch 启动并运行后安全删除。

``` shell
echo "keystore_password" > /path/to/my_pwd_file.tmp
chmod 600 /path/to/my_pwd_file.tmp
sudo systemctl set-environment ES_KEYSTORE_PASSPHRASE_FILE=/path/to/my_pwd_file.tmp
sudo systemctl start elasticsearch.service
```

By default the Elasticsearch service doesn’t log information in the `systemd` journal. To enable `journalctl` logging, the `--quiet` option must be removed from the `ExecStart` command line in the `elasticsearch.service` file.
默认情况下，Elasticsearch 服务不会在`systemd`日志中记录信息。要启用`journalctl`日志记录，必须从`elasticsearch.service`文件中的`ExecStart`命令行中删除`--quiet`选项。

When `systemd` logging is enabled, the logging information are available using the `journalctl` commands:
当启用`systemd`日志记录时，可以使用`journalctl`命令获取日志信息：

To tail the journal:
跟踪日志：

``` shell
sudo journalctl -f
```

To list journal entries for the elasticsearch service:
列出 elasticsearch 服务的日记帐分录：

``` shell
sudo journalctl --unit elasticsearch
```

To list journal entries for the elasticsearch service starting from a given time:
列出从给定时间开始的 elasticsearch 服务的日记帐分录：

``` shell
sudo journalctl --unit elasticsearch --since  "2016-10-30 18:17:16"
```

Check `man journalctl` or https://www.freedesktop.org/software/systemd/man/journalctl.html for more command line options.
检查`man journalctl`或https://www.freedesktop.org/software/systemd/man/journalctl.html以获取更多命令行选项。

> **TIP**
> **Startup timeouts with older `systemd` versions**
> **旧版 `systemd` 的启动超时**
>
> By default Elasticsearch sets the `TimeoutStartSec` parameter to `systemd` to `900s`. If you are running at least version 238 of `systemd` then Elasticsearch can automatically extend the startup timeout, and will do so repeatedly until startup is complete even if it takes longer than 900s.
> 默认情况下，Elasticsearch 将 `TimeoutStartSec` 参数设置为 `systemd`，设置为 `900s`。如果您运行的是 `systemd` 的 238 以上版本，那么 Elasticsearch 可以自动延长启动超时时间，并且会重复执行此操作，直到启动完成，即使启动时间超过 900 秒。
>
> Versions of `systemd` prior to 238 do not support the timeout extension mechanism and will terminate the Elasticsearch process if it has not fully started up within the configured timeout. If this happens, Elasticsearch will report in its logs that it was shut down normally a short time after it started:
> 238 之前的 `systemd` 版本不支持超时延长机制，如果 Elasticsearch 进程在配置的超时时间内未完全启动，则会终止该进程。如果发生这种情况，Elasticsearch 将在其日志中报告它在启动后不久就正常关闭：
>
> ``` log
> [2022-01-31T01:22:31,077][INFO ][o.e.n.Node               ] [instance-0000000123] starting ...
> ...
> [2022-01-31T01:37:15,077][INFO ][o.e.n.Node               ] [instance-0000000123] stopping ...
> ```
> 
> However the `systemd` logs will report that the startup timed out:
> 然而，`systemd` 日志会报告启动超时：
>
> ``` log
> Jan 31 01:22:30 debian systemd[1]: Starting Elasticsearch...
> Jan 31 01:37:15 debian systemd[1]: elasticsearch.service: Start operation timed out. Terminating.
> Jan 31 01:37:15 debian systemd[1]: elasticsearch.service: Main process exited, code=killed, status=15/TERM
> Jan 31 01:37:15 debian systemd[1]: elasticsearch.service: Failed with result 'timeout'.
> Jan 31 01:37:15 debian systemd[1]: Failed to start Elasticsearch.
> ```
> 
> To avoid this, upgrade your `systemd` to at least version 238. You can also temporarily work around the problem by extending the `TimeoutStartSec` parameter.
> 为了避免这种情况，请将您的`systemd`升级到至少 238 版本。您也可以通过扩展`TimeoutStartSec`参数来暂时解决此问题。







