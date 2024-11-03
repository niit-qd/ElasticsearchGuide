[TOC]

---

# [Auditing security settings | 审核安全设置](https://www.elastic.co/guide/en/elasticsearch/reference/current/auditing-settings.html)

You can use [audit logging](https://www.elastic.co/guide/en/elasticsearch/reference/current/enable-audit-logging.html) to record security-related events, such as authentication failures, refused connections, and data-access events. In addition, changes via the APIs to the security configuration, such as creating, updating and removing [native](https://www.elastic.co/guide/en/elasticsearch/reference/current/native-realm.html) and [built-in](https://www.elastic.co/guide/en/elasticsearch/reference/current/built-in-users.html) users, [roles](https://www.elastic.co/guide/en/elasticsearch/reference/current/security-api-put-role.html), [role mappings](https://www.elastic.co/guide/en/elasticsearch/reference/current/security-api-put-role-mapping.html) and [API keys](https://www.elastic.co/guide/en/elasticsearch/reference/current/security-api-create-api-key.html) are also recorded.
您可以使用 [审计日志](https://www.elastic.co/guide/en/elasticsearch/reference/current/enable-audit-logging.html) 记录与安全相关的事件，例如身份验证失败、拒绝连接和数据访问事件。此外，还会记录通过 API 对安全配置所做的更改，例如创建、更新和删除 [本机](https://www.elastic.co/guide/en/elasticsearch/reference/current/native-realm.html) 和 [内置](https://www.elastic.co/guide/en/elasticsearch/reference/current/built-in-users.html) 用户、[角色](https://www.elastic.co/guide/en/elasticsearch/reference/current/security-api-put-role.html)、[角色映射](https://www.elastic.co/guide/en/elasticsearch/reference/current/security-api-put-role-mapping.html) 和 [API 密钥](https://www.elastic.co/guide/en/elasticsearch/reference/current/security-api-create-api-key.html)。

> **TIP**
> Audit logs are only available on certain subscription levels. For more information, see https://www.elastic.co/subscriptions.
> 审核日志仅在某些订阅级别可用。有关更多信息，请参阅 https://www.elastic.co/subscriptions。

If configured, auditing settings must be set on every node in the cluster. Static settings, such as `xpack.security.audit.enabled`, must be configured in elasticsearch.yml on each node. For dynamic auditing settings, use the [cluster update settings API](https://www.elastic.co/guide/en/elasticsearch/reference/current/cluster-update-settings.html) to ensure the setting is the same on all nodes.
如果已配置，则必须在集群中的每个节点上设置审核设置。必须在每个节点上的 elasticsearch.yml 中配置静态设置，例如“xpack.security.audit.enabled”。对于动态审核设置，请使用[集群更新设置 API](https://www.elastic.co/guide/en/elasticsearch/reference/current/cluster-update-settings.html) 确保设置与所有节点。

---

## [General Auditing Settings | 常规审核设置](https://www.elastic.co/guide/en/elasticsearch/reference/current/auditing-settings.html#general-audit-settings)

- **`xpack.security.audit.enabled`**
    ([Static](https://www.elastic.co/guide/en/elasticsearch/reference/current/settings.html#static-cluster-setting)) Set to `true` to enable auditing on the node. The default value is `false`. This puts the auditing events in a dedicated file named `<clustername>_audit.json` on each node.
    （[Static](https://www.elastic.co/guide/en/elasticsearch/reference/current/settings.html#static-cluster-setting)）设置为 `true` 以在节点上启用审计。默认值为 `false`。这会将审计事件放在每个节点上名为 `<clustername>_audit.json` 的专用文件中。

    If enabled, this setting must be configured in `elasticsearch.yml` on all nodes in the cluster.
    如果启用，则必须在集群中所有节点的`elasticsearch.yml`中配置此设置。

---

## [Audited Event Settings | 审核事件设置](https://www.elastic.co/guide/en/elasticsearch/reference/current/auditing-settings.html#event-audit-settings)

The events and some other information about what gets logged can be controlled by using the following settings:
可以使用以下设置来控制记录的事件和其他一些信息：

- **`xpack.security.audit.logfile.events.include`**
  ([Dynamic](https://www.elastic.co/guide/en/elasticsearch/reference/current/settings.html#dynamic-cluster-setting)) Specifies the [kind of events](https://www.elastic.co/guide/en/elasticsearch/reference/current/audit-event-types.html) to print in the auditing output. In addition, `_all` can be used to exhaustively audit all the events, but this is usually discouraged since it will get very verbose. The default list value contains: `access_denied`, `access_granted`, `anonymous_access_denied`, `authentication_failed`, `connection_denied`, `tampered_request`, `run_as_denied`, `run_as_granted`, `security_config_change`.
  （[Dynamic](https://www.elastic.co/guide/en/elasticsearch/reference/current/settings.html#dynamic-cluster-setting)）指定要在审计输出中打印的[事件类型](https://www.elastic.co/guide/en/elasticsearch/reference/current/audit-event-types.html)。此外，`_all` 可用于详尽审计所有事件，但通常不建议这样做，因为它会变得非常冗长。默认列表值包含：`access_denied`、`access_granted`、`anonymous_access_denied`、`authentication_failed`、`connection_denied`、`tampered_request`、`run_as_denied`、`run_as_granted`、`security_config_change`。
- **`xpack.security.audit.logfile.events.exclude`**
  ([Dynamic](https://www.elastic.co/guide/en/elasticsearch/reference/current/settings.html#dynamic-cluster-setting)) Excludes the specified [kind of events](https://www.elastic.co/guide/en/elasticsearch/reference/current/audit-event-types.html) from the include list. This is useful in the case where the `events.include` setting contains the special value `_all`. The default is the empty list.
  （[Dynamic](https://www.elastic.co/guide/en/elasticsearch/reference/current/settings.html#dynamic-cluster-setting)）从包含列表中排除指定的[事件类型](https://www.elastic.co/guide/en/elasticsearch/reference/current/audit-event-types.html)。这在 `events.include` 设置包含特殊值 `_all` 的情况下很有用。默认值为空列表。
- **`xpack.security.audit.logfile.events.emit_request_body`**
  ([Dynamic](https://www.elastic.co/guide/en/elasticsearch/reference/current/settings.html#dynamic-cluster-setting)) Specifies whether to include the full request body from REST requests as an attribute of certain kinds of audit events. This setting can be used to [audit search queries](https://www.elastic.co/guide/en/elasticsearch/reference/current/auditing-search-queries.html).
  （[Dynamic](https://www.elastic.co/guide/en/elasticsearch/reference/current/settings.html#dynamic-cluster-setting)）指定是否将 REST 请求的完整请求正文作为某些类型的审计事件的属性。此设置可用于[审计搜索查询](https://www.elastic.co/guide/en/elasticsearch/reference/current/auditing-search-queries.html)。
  
  The default value is `false`, so request bodies are not printed.
  默认值为“false”，因此不会打印请求正文。

  > **IMPORTANT**
  > Be advised that sensitive data may be audited in plain text when including the request body in audit events, even though all the security APIs, such as those that change the user’s password, have the credentials filtered out when audited.
  > 请注意，在审计事件中包含请求正文时，敏感数据可能会以纯文本形式被审计，即使所有安全 API（例如更改用户密码的 API）在审计时都会过滤掉凭据。

---

## [Local Node Info Settings | 本地节点信息设置](https://www.elastic.co/guide/en/elasticsearch/reference/current/auditing-settings.html#node-audit-settings)

- **`xpack.security.audit.logfile.emit_node_name`**
  ([Dynamic](https://www.elastic.co/guide/en/elasticsearch/reference/current/settings.html#dynamic-cluster-setting)) Specifies whether to include the [node name](https://www.elastic.co/guide/en/elasticsearch/reference/current/important-settings.html#node-name) as a field in each audit event. The default value is `false`.
  （[Dynamic](https://www.elastic.co/guide/en/elasticsearch/reference/current/settings.html#dynamic-cluster-setting)）指定是否在每个审计事件中包含[节点名称](https://www.elastic.co/guide/en/elasticsearch/reference/current/important-settings.html#node-name)作为字段。默认值为`false`。
- **`xpack.security.audit.logfile.emit_node_host_address`**
  ([Dynamic](https://www.elastic.co/guide/en/elasticsearch/reference/current/settings.html#dynamic-cluster-setting)) Specifies whether to include the node’s IP address as a field in each audit event. The default value is `false`.
  （[Dynamic](https://www.elastic.co/guide/en/elasticsearch/reference/current/settings.html#dynamic-cluster-setting)）指定是否在每个审计事件中包含节点的 IP 地址作为字段。默认值为`false`。
- **`xpack.security.audit.logfile.emit_node_host_name`**
  ([Dynamic](https://www.elastic.co/guide/en/elasticsearch/reference/current/settings.html#dynamic-cluster-setting)) Specifies whether to include the node’s host name as a field in each audit event. The default value is `false`.
  （[Dynamic](https://www.elastic.co/guide/en/elasticsearch/reference/current/settings.html#dynamic-cluster-setting)）指定是否在每个审计事件中包含节点的主机名作为字段。默认值为`false`。
- **`xpack.security.audit.logfile.emit_node_id`**
  ([Dynamic](https://www.elastic.co/guide/en/elasticsearch/reference/current/settings.html#dynamic-cluster-setting)) Specifies whether to include the node id as a field in each audit event. Unlike [node name](https://www.elastic.co/guide/en/elasticsearch/reference/current/important-settings.html#node-name), whose value might change if the administrator changes the setting in the config file, the node id will persist across cluster restarts and the administrator cannot change it. The default value is `true`.
  （[Dynamic](https://www.elastic.co/guide/en/elasticsearch/reference/current/settings.html#dynamic-cluster-setting)）指定是否在每个审计事件中包含节点 ID 作为字段。与 [节点名称](https://www.elastic.co/guide/en/elasticsearch/reference/current/important-settings.html#node-name) 不同，如果管理员更改配置文件中的设置，其值可能会更改，节点 ID 将在集群重启后保持不变，管理员无法更改它。默认值为 `true`。

---

## [Audit Logfile Event Ignore Policies | 审计日志文件事件忽略策略](https://www.elastic.co/guide/en/elasticsearch/reference/current/auditing-settings.html#audit-event-ignore-policies)

The following settings affect the [ignore policies](https://www.elastic.co/guide/en/elasticsearch/reference/current/audit-log-ignore-policy.html    ) that enable fine-grained control over which audit events are printed to the log file. All of the settings with the same policy name combine to form a single policy. If an event matches all the conditions of any policy, it is ignored and not printed. Most audit events are subject to the ignore policies. The sole exception are events of the `security_config_change` type, which cannot be filtered out, unless [excluded](https://www.elastic.co/guide/en/elasticsearch/reference/current/auditing-settings.html#xpack-sa-lf-events-exclude) altogether.
以下设置会影响 [忽略策略](https://www.elastic.co/guide/en/elasticsearch/reference/current/audit-log-ignore-policy.html )，这些策略可对将哪些审计事件打印到日志文件进行细粒度控制。所有具有相同策略名称的设置组合在一起形成单个策略。如果事件符合任何策略的所有条件，则会被忽略且不会打印。大多数审计事件都受忽略策略的约束。唯一的例外是 `security_config_change` 类型的事件，除非 [完全排除](https://www.elastic.co/guide/en/elasticsearch/reference/current/auditing-settings.html#xpack-sa-lf-events-exclude)，否则无法过滤掉这些事件。

- **`xpack.security.audit.logfile.events.ignore_filters.<policy_name>.users`**
  ([Dynamic](https://www.elastic.co/guide/en/elasticsearch/reference/current/settings.html#dynamic-cluster-setting)) A list of user names or wildcards. The specified policy will not print audit events for users matching these values.
  （[Dynamic](https://www.elastic.co/guide/en/elasticsearch/reference/current/settings.html#dynamic-cluster-setting)）用户名或通配符列表。指定的策略不会打印与这些值匹配的用户的审计事件。
- **`xpack.security.audit.logfile.events.ignore_filters.<policy_name>.realms`**
  ([Dynamic](https://www.elastic.co/guide/en/elasticsearch/reference/current/settings.html#dynamic-cluster-setting)) A list of authentication realm names or wildcards. The specified policy will not print audit events for users in these realms.
  （[动态](https://www.elastic.co/guide/en/elasticsearch/reference/current/settings.html#dynamic-cluster-setting)）身份验证领域名称或通配符的列表。指定的策略不会打印这些领域中用户的审计事件。
- **`xpack.security.audit.logfile.events.ignore_filters.<policy_name>.actions`**
  ([Dynamic](https://www.elastic.co/guide/en/elasticsearch/reference/current/settings.html#dynamic-cluster-setting)) A list of action names or wildcards. Action name can be found in the action field of the audit event. The specified policy will not print audit events for actions matching these values.
  （[Dynamic](https://www.elastic.co/guide/en/elasticsearch/reference/current/settings.html#dynamic-cluster-setting)）操作名称或通配符列表。操作名称可以在审计事件的操作字段中找到。指定的策略不会打印与这些值匹配的操作的审计事件。
- **`xpack.security.audit.logfile.events.ignore_filters.<policy_name>.roles`**
  ([Dynamic](https://www.elastic.co/guide/en/elasticsearch/reference/current/settings.html#dynamic-cluster-setting)) A list of role names or wildcards. The specified policy will not print audit events for users that have these roles. If the user has several roles, some of which are **not** covered by the policy, the policy will **not** cover this event.
  （[Dynamic](https://www.elastic.co/guide/en/elasticsearch/reference/current/settings.html#dynamic-cluster-setting)）角色名称或通配符的列表。指定的策略不会打印具有这些角色的用户的审计事件。如果用户有多个角色，其中一些角色**未**受策略覆盖，则策略**不会**覆盖此事件。
- **`xpack.security.audit.logfile.events.ignore_filters.<policy_name>.indices`**
  ([Dynamic](https://www.elastic.co/guide/en/elasticsearch/reference/current/settings.html#dynamic-cluster-setting)) A list of index names or wildcards. The specified policy will not print audit events when all the indices in the event match these values. If the event concerns several indices, some of which are **not** covered by the policy, the policy will **not** cover this event.
  （[Dynamic](https://www.elastic.co/guide/en/elasticsearch/reference/current/settings.html#dynamic-cluster-setting)）索引名称或通配符的列表。当事件中的所有索引都与这些值匹配时，指定的策略将不会打印审计事件。如果事件涉及多个索引，其中一些索引**未**受策略覆盖，则策略**不会**覆盖此事件。
