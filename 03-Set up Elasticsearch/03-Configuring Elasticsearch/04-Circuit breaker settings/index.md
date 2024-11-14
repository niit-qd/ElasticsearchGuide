[TOC]

---

# [Circuit breaker settings | 断路器设置](https://www.elastic.co/guide/en/elasticsearch/reference/current/circuit-breaker.html)

---

Elasticsearch contains multiple circuit breakers used to prevent operations from using an excessive amount of memory. Each breaker tracks the memory used by certain operations and specifies a limit for how much memory it may track. Additionally, there is a parent-level breaker that specifies the total amount of memory that may be tracked across all breakers.
Elasticsearch 包含多个断路器，用于防止操作使用过多内存。每个断路器都会跟踪某些操作使用的内存，并指定其可以跟踪的内存量限制。此外，还有一个父级断路器，用于指定所有断路器可以跟踪的总内存量。

When a circuit breaker reaches its limit, Elasticsearch will reject further operations. See [Circuit breaker errors](https://www.elastic.co/guide/en/elasticsearch/reference/current/circuit-breaker-errors.html) for information about errors raised by circuit breakers.
当断路器达到其极限时，Elasticsearch 将拒绝进一步的操作。有关断路器引发的错误的信息，请参阅[断路器错误](https://www.elastic.co/guide/en/elasticsearch/reference/current/circuit-breaker-errors.html)。

Circuit breakers do not track all memory usage in Elasticsearch and therefore provide only incomplete protection against excessive memory usage. If Elasticsearch uses too much memory then it may suffer from performance issues and nodes may even fail with an `OutOfMemoryError`. See [High JVM memory pressure](https://www.elastic.co/guide/en/elasticsearch/reference/current/high-jvm-memory-pressure.html) for help with troubleshooting high heap usage.
断路器不会跟踪 Elasticsearch 中的所有内存使用情况，因此只能提供不完整的保护以防止过度使用内存。如果 Elasticsearch 使用过多内存，则可能会出现性能问题，节点甚至可能会因“OutOfMemoryError”而失败。请参阅 [高 JVM 内存压力](https://www.elastic.co/guide/en/elasticsearch/reference/current/high-jvm-memory-pressure.html) 以获取有关解决高堆使用率问题的帮助。

Except where noted otherwise, these settings can be dynamically updated on a live cluster with the [cluster-update-settings](https://www.elastic.co/guide/en/elasticsearch/reference/current/cluster-update-settings.html) API.
除非另有说明，这些设置均可通过 [cluster-update-settings](https://www.elastic.co/guide/en/elasticsearch/reference/current/cluster-update-settings.html) API 在实时集群上动态更新。

For information about circuit breaker errors, see [Circuit breaker errors](https://www.elastic.co/guide/en/elasticsearch/reference/current/circuit-breaker-errors.html).
有关断路器错误的信息，请参阅[断路器错误](https://www.elastic.co/guide/en/elasticsearch/reference/current/circuit-breaker-errors.html)。

---

## [Parent circuit breaker | 家长断路器](https://www.elastic.co/guide/en/elasticsearch/reference/current/circuit-breaker.html#parent-circuit-breaker)

The parent-level breaker can be configured with the following settings:
父级断路器可以配置以下设置：

- **`indices.breaker.total.use_real_memory`**
  ([Static](https://www.elastic.co/guide/en/elasticsearch/reference/current/settings.html#static-cluster-setting)) Determines whether the parent breaker should take real memory usage into account (`true`) or only consider the amount that is reserved by child circuit breakers (`false`). Defaults to true.
  （[静态](https://www.elastic.co/guide/en/elasticsearch/reference/current/settings.html#static-cluster-setting)）确定父断路器是否应考虑实际内存使用情况（"true"）或仅考虑子断路器保留的内存量（"false"）。默认为 true。
- **`indices.breaker.total.limit logo cloud`**
  ([Dynamic](https://www.elastic.co/guide/en/elasticsearch/reference/current/settings.html#dynamic-cluster-setting)) Starting limit for overall parent breaker. Defaults to 70% of JVM heap if `indices.breaker.total.use_real_memory` is `false`. If `indices.breaker.total.use_real_memory` is `true`, defaults to 95% of the JVM heap.
  （[动态](https://www.elastic.co/guide/en/elasticsearch/reference/current/settings.html#dynamic-cluster-setting)）整体父断路器的起始限制。如果 `indices.breaker.total.use_real_memory` 为 `false`，则默认为 JVM 堆的 70%。如果 `indices.breaker.total.use_real_memory` 为 `true`，则默认为 JVM 堆的 95%。

---

## [Field data circuit breaker | 现场数据断路器](https://www.elastic.co/guide/en/elasticsearch/reference/current/circuit-breaker.html#fielddata-circuit-breaker)

The field data circuit breaker estimates the heap memory required to load a field into the [field data cache](https://www.elastic.co/guide/en/elasticsearch/reference/current/modules-fielddata.html). If loading the field would cause the cache to exceed a predefined memory limit, the circuit breaker stops the operation and returns an error.
字段数据断路器会估算将字段加载到 [字段数据缓存](https://www.elastic.co/guide/en/elasticsearch/reference/current/modules-fielddata.html) 所需的堆内存。如果加载字段会导致缓存超出预定义的内存限制，则断路器会停止操作并返回错误。

- **`indices.breaker.fielddata.limit logo cloud`**
  ([Dynamic](https://www.elastic.co/guide/en/elasticsearch/reference/current/settings.html#dynamic-cluster-setting)) Limit for fielddata breaker. Defaults to 40% of JVM heap.
  （[动态](https://www.elastic.co/guide/en/elasticsearch/reference/current/settings.html#dynamic-cluster-setting)）fielddata 断路器的限制。默认为 JVM 堆的 40%。
- **`indices.breaker.fielddata.overhead logo cloud`**
  ([Dynamic](https://www.elastic.co/guide/en/elasticsearch/reference/current/settings.html#dynamic-cluster-setting)) A constant that all field data estimations are multiplied with to determine a final estimation. Defaults to `1.03`.
  （[动态](https://www.elastic.co/guide/en/elasticsearch/reference/current/settings.html#dynamic-cluster-setting)）所有现场数据估算值都与其相乘以确定最终估算值的常数。默认为 `1.03`。

---

## [Request circuit breaker | 请求断路器](https://www.elastic.co/guide/en/elasticsearch/reference/current/circuit-breaker.html#request-circuit-breaker)

The request circuit breaker allows Elasticsearch to prevent per-request data structures (for example, memory used for calculating aggregations during a request) from exceeding a certain amount of memory.
请求断路器允许 Elasticsearch 防止每个请求的数据结构（例如，用于在请求期间计算聚合的内存）超过一定量的内存。

- **`indices.breaker.request.limit logo cloud`**
  ([Dynamic](https://www.elastic.co/guide/en/elasticsearch/reference/current/settings.html#dynamic-cluster-setting)) Limit for request breaker, defaults to 60% of JVM heap.
  ([动态](https://www.elastic.co/guide/en/elasticsearch/reference/current/settings.html#dynamic-cluster-setting)) 请求断路器的限制，默认为 JVM 堆的 60%。
- **`indices.breaker.request.overhead logo cloud`**
  ([Dynamic](https://www.elastic.co/guide/en/elasticsearch/reference/current/settings.html#dynamic-cluster-setting)) A constant that all request estimations are multiplied with to determine a final estimation. Defaults to `1`.
  （[动态](https://www.elastic.co/guide/en/elasticsearch/reference/current/settings.html#dynamic-cluster-setting)）所有请求估算值都与其相乘以确定最终估算值的常数。默认为 `1`。

---

## [In flight requests circuit breaker | 飞行中请求断路器](https://www.elastic.co/guide/en/elasticsearch/reference/current/circuit-breaker.html#in-flight-circuit-breaker)

The in flight requests circuit breaker allows Elasticsearch to limit the memory usage of all currently active incoming requests on transport or HTTP level from exceeding a certain amount of memory on a node. The memory usage is based on the content length of the request itself. This circuit breaker also considers that memory is not only needed for representing the raw request but also as a structured object which is reflected by default overhead.
正在传输的请求断路器允许 Elasticsearch 限制传输或 HTTP 级别上所有当前活动的传入请求的内存使用量，使其不超过节点上的一定内存量。内存使用量基于请求本身的内容长度。此断路器还认为，内存不仅需要用于表示原始请求，而且还需要作为默认开销反映的结构化对象。

- **`network.breaker.inflight_requests.limit`**
  ([Dynamic](https://www.elastic.co/guide/en/elasticsearch/reference/current/settings.html#dynamic-cluster-setting)) Limit for in flight requests breaker, defaults to 100% of JVM heap. This means that it is bound by the limit configured for the parent circuit breaker.
  （[动态](https://www.elastic.co/guide/en/elasticsearch/reference/current/settings.html#dynamic-cluster-setting)）断路器的运行中请求限制，默认为 JVM 堆的 100%。这意味着它受为父断路器配置的限制约束。
- **`network.breaker.inflight_requests.overhead`**
  ([Dynamic](https://www.elastic.co/guide/en/elasticsearch/reference/current/settings.html#dynamic-cluster-setting)) A constant that all in flight requests estimations are multiplied with to determine a final estimation. Defaults to 2.
  （[动态](https://www.elastic.co/guide/en/elasticsearch/reference/current/settings.html#dynamic-cluster-setting)）一个常数，所有正在进行的请求估计值都会与其相乘，以确定最终估计值。默认为 2。

---

## [Script compilation circuit breaker | 脚本编译断路器](https://www.elastic.co/guide/en/elasticsearch/reference/current/circuit-breaker.html#script-compilation-circuit-breaker)

Slightly different than the previous memory-based circuit breaker, the script compilation circuit breaker limits the number of inline script compilations within a period of time.
与之前基于内存的断路器稍有不同，脚本编译断路器限制了一段时间内内联脚本编译的次数。

See the "prefer-parameters" section of the [scripting](https://www.elastic.co/guide/en/elasticsearch/reference/current/modules-scripting-using.html) documentation for more information.
有关详细信息，请参阅 [脚本](https://www.elastic.co/guide/en/elasticsearch/reference/current/modules-scripting-using.html) 文档的“prefer-parameters”部分。

- **`script.max_compilations_rate`**
  ([Dynamic](https://www.elastic.co/guide/en/elasticsearch/reference/current/settings.html#dynamic-cluster-setting)) Limit for the number of unique dynamic scripts within a certain interval that are allowed to be compiled. Defaults to `150/5m`, meaning 150 every 5 minutes.
  （[动态](https://www.elastic.co/guide/en/elasticsearch/reference/current/settings.html#dynamic-cluster-setting)）在一定时间间隔内允许编译的唯一动态脚本数量限制。默认值为 `150/5m`，即每 5 分钟编译 150 个。

If the cluster regularly hits the given `max_compilation_rate`, it’s possible the script cache is undersized, use [Nodes Stats](https://www.elastic.co/guide/en/elasticsearch/reference/current/cluster-nodes-stats.html) to inspect the number of recent cache evictions, `script.cache_evictions_history` and compilations `script.compilations_history`. If there are a large number of recent cache evictions or compilations, the script cache may be undersized, consider doubling the size of the script cache via the setting `script.cache.max_size`.
如果集群定期达到给定的“max_compilation_rate”，则脚本缓存可能不足，请使用[节点统计信息](https://www.elastic.co/guide/en/elasticsearch/reference/current/cluster-nodes-stats.html)检查最近的缓存驱逐次数、“script.cache_evictions_history”和编译“script.compilations_history”。如果最近有大量缓存驱逐或编译，则脚本缓存可能不足，请考虑通过设置“script.cache.max_size”将脚本缓存的大小加倍。

---

## [Regex circuit breaker | 正则表达式断路器](https://www.elastic.co/guide/en/elasticsearch/reference/current/circuit-breaker.html#regex-circuit-breaker)

Poorly written regular expressions can degrade cluster stability and performance. The regex circuit breaker limits the use and complexity of [regex in Painless scripts](https://www.elastic.co/guide/en/elasticsearch/painless/8.16/painless-regexes.html).
编写不当的正则表达式会降低集群稳定性和性能。正则表达式断路器限制了 [Painless 脚本中正则表达式的使用和复杂性](https://www.elastic.co/guide/en/elasticsearch/painless/8.16/painless-regexes.html)。

- **`script.painless.regex.enabled`**
  ([Static](https://www.elastic.co/guide/en/elasticsearch/reference/current/settings.html#static-cluster-setting)) Enables regex in Painless scripts. Accepts:
  ([Static](https://www.elastic.co/guide/en/elasticsearch/reference/current/settings.html#static-cluster-setting)) 在 Painless 脚本中启用正则表达式。接受：
  - **`limited` (Default)**
    Enables regex but limits complexity using the [`script.painless.regex.limit-factor`](https://www.elastic.co/guide/en/elasticsearch/reference/current/circuit-breaker.html#script-painless-regex-limit-factor) cluster setting.
    启用正则表达式，但使用 [`script.painless.regex.limit-factor`](https://www.elastic.co/guide/en/elasticsearch/reference/current/circuit-breaker.html#script-painless-regex-limit-factor) 集群设置限制复杂性。
  - **`true`**
    Enables regex with no complexity limits. Disables the regex circuit breaker.
    启用没有复杂度限制的正则表达式。禁用正则表达式断路器。
  - **`false`**
    Disables regex. Any Painless script containing a regular expression returns an error.
    禁用正则表达式。任何包含正则表达式的 Painless 脚本都会返回错误。
- **`script.painless.regex.limit-factor`**
  ([Static](https://www.elastic.co/guide/en/elasticsearch/reference/current/settings.html#static-cluster-setting)) Limits the number of characters a regular expression in a Painless script can consider. Elasticsearch calculates this limit by multiplying the setting value by the script input’s character length.
  （[静态](https://www.elastic.co/guide/en/elasticsearch/reference/current/settings.html#static-cluster-setting)）限制 Painless 脚本中正则表达式可以考虑的字符数。Elasticsearch 通过将设置值乘以脚本输入的字符长度来计算此限制。

  For example, the input `foobarbaz` has a character length of 9. If `script.painless.regex.limit-factor` is 6, a regular expression on `foobarbaz` can consider up to 54 (9 * 6) characters. If the expression exceeds this limit, it triggers the regex circuit breaker and returns an error.
  例如，输入的`foobarbaz`的字符长度为 9。如果`script.painless.regex.limit-factor`为 6，则`foobarbaz`上的正则表达式最多可以考虑 54（9 * 6）个字符。如果表达式超出此限制，则会触发正则表达式断路器并返回错误。

  Elasticsearch only applies this limit if [script.painless.regex.enabled](https://www.elastic.co/guide/en/elasticsearch/reference/current/circuit-breaker.html#script-painless-regex-enabled) is limited.
  仅当 [script.painless.regex.enabled](https://www.elastic.co/guide/en/elasticsearch/reference/current/circuit-breaker.html#script-painless-regex-enabled) 受限时，Elasticsearch 才会应用此限制。

---

  ## [EQL circuit breaker | EQL断路器](https://www.elastic.co/guide/en/elasticsearch/reference/current/circuit-breaker.html#circuit-breakers-page-eql)

When a [sequence](https://www.elastic.co/guide/en/elasticsearch/reference/current/eql-syntax.html#eql-sequences) query is executed, the node handling the query needs to keep some structures in memory, which are needed by the algorithm implementing the sequence matching. When large amounts of data need to be processed, and/or a large amount of matched sequences is requested by the user (by setting the [size](https://www.elastic.co/guide/en/elasticsearch/reference/current/eql-search-api.html#eql-search-api-params-size) query param), the memory occupied by those structures could potentially exceed the available memory of the JVM. This would cause an `OutOfMemory` exception which would bring down the node.
执行 [sequence](https://www.elastic.co/guide/en/elasticsearch/reference/current/eql-syntax.html#eql-sequences) 查询时，处理查询的节点需要在内存中保留一些结构，这些结构是实现序列匹配的算法所必需的。当需要处理大量数据和/或用户请求大量匹配的序列时（通过设置 [size](https://www.elastic.co/guide/en/elasticsearch/reference/current/eql-search-api.html#eql-search-api-params-size) 查询参数），这些结构占用的内存可能会超过 JVM 的可用内存。这将导致 `OutOfMemory` 异常，从而导致节点关闭。

To prevent this from happening, a special [circuit breaker](https://www.elastic.co/guide/en/elasticsearch/reference/current/circuit-breaker.html) is used, which limits the memory allocation during the execution of a [sequence](https://www.elastic.co/guide/en/elasticsearch/reference/current/eql-syntax.html#eql-sequences) query. When the breaker is triggered, an `org.elasticsearch.common.breaker.CircuitBreakingException` is thrown and a descriptive error message including `circuit_breaking_exception` is returned to the user.
为了防止这种情况发生，我们使用了一种特殊的断路器，它可以限制执行 [sequence](https://www.elastic.co/guide/en/elasticsearch/reference/current/circuit-breaker.html) 查询期间的内存分配。当断路器被触发时，会抛出 `org.elasticsearch.common.breaker.CircuitBreakingException`，并向用户返回包含 `circuit_breaking_exception` 的描述性错误消息。

This [circuit breaker](https://www.elastic.co/guide/en/elasticsearch/reference/current/circuit-breaker.html) can be configured using the following settings:
可以使用以下设置来配置此[断路器](https://www.elastic.co/guide/en/elasticsearch/reference/current/circuit-breaker.html)：

- **`breaker.eql_sequence.limit`**
  ([Dynamic](https://www.elastic.co/guide/en/elasticsearch/reference/current/cluster-update-settings.html)) The limit for circuit breaker used to restrict the memory utilisation during the execution of an EQL sequence query. This value is defined as a percentage of the JVM heap. Defaults to `50%`. If the [parent circuit breaker](https://www.elastic.co/guide/en/elasticsearch/reference/current/circuit-breaker.html#parent-circuit-breaker) is set to a value less than `50%`, this setting uses that value as its default instead.
  （[动态](https://www.elastic.co/guide/en/elasticsearch/reference/current/cluster-update-settings.html)）用于限制执行 EQL 序列查询期间内存利用率的断路器限制。此值定义为 JVM 堆的百分比。默认为 `50%`。如果 [父断路器](https://www.elastic.co/guide/en/elasticsearch/reference/current/circuit-breaker.html#parent-circuit-breaker) 设置为小于 `50%` 的值，则此设置将使用该值作为其默认值。
- **`breaker.eql_sequence.overhead`**
  ([Dynamic](https://www.elastic.co/guide/en/elasticsearch/reference/current/cluster-update-settings.html)) A constant that sequence query memory estimates are multiplied by to determine a final estimate. Defaults to `1`.
  （[动态]（https://www.elastic.co/guide/en/elasticsearch/reference/current/cluster-update-settings.html））序列查询内存估计值乘以该常数以确定最终估计值。默认为 `1`。
- **`breaker.eql_sequence.type`**
  ([Static](https://www.elastic.co/guide/en/elasticsearch/reference/current/settings.html#static-cluster-setting)) Circuit breaker type. Valid values are:
  （[静态]（https://www.elastic.co/guide/en/elasticsearch/reference/current/settings.html#static-cluster-setting））断路器类型。有效值为：
  - **`memory` (Default)**
    The breaker limits memory usage for EQL sequence queries.
    断路器限制了 EQL 序列查询的内存使用量。
  - **`noop`**
    Disables the breaker.
    禁用断路器。

---

## [Machine learning circuit breaker | 机器学习断路器](https://www.elastic.co/guide/en/elasticsearch/reference/current/circuit-breaker.html#circuit-breakers-page-model-inference)

- **`breaker.model_inference.limit`**
  ([Dynamic](https://www.elastic.co/guide/en/elasticsearch/reference/current/cluster-update-settings.html)) The limit for the trained model circuit breaker. This value is defined as a percentage of the JVM heap. Defaults to `50%`. If the [parent circuit breaker](https://www.elastic.co/guide/en/elasticsearch/reference/current/circuit-breaker.html#parent-circuit-breaker) is set to a value less than `50%`, this setting uses that value as its default instead.
  （[动态](https://www.elastic.co/guide/en/elasticsearch/reference/current/cluster-update-settings.html)）训练模型断路器的限制。此值定义为 JVM 堆的百分比。默认为 `50%`。如果 [父断路器](https://www.elastic.co/guide/en/elasticsearch/reference/current/circuit-breaker.html#parent-circuit-breaker) 设置为小于 `50%` 的值，则此设置将使用该值作为其默认值。
- **`breaker.model_inference.overhead`**
  ([Dynamic](https://www.elastic.co/guide/en/elasticsearch/reference/current/cluster-update-settings.html)) A constant that all trained model estimations are multiplied by to determine a final estimation. See [Circuit breaker settings](https://www.elastic.co/guide/en/elasticsearch/reference/current/circuit-breaker.html). Defaults to `1`.
  （[动态]（https://www.elastic.co/guide/en/elasticsearch/reference/current/cluster-update-settings.html））所有经过训练的模型估计值都乘以该常数以确定最终估计值。请参阅[断路器设置]（https://www.elastic.co/guide/en/elasticsearch/reference/current/circuit-breaker.html）。默认为 `1`。
- **`breaker.model_inference.type`**
  ([Static](https://www.elastic.co/guide/en/elasticsearch/reference/current/settings.html#static-cluster-setting)) The underlying type of the circuit breaker. There are two valid options: `noop` and `memory`. `noop` means the circuit breaker does nothing to prevent too much memory usage. `memory` means the circuit breaker tracks the memory used by trained models and can potentially break and prevent `OutOfMemory` errors. The default value is `memory`.
  （[Static](https://www.elastic.co/guide/en/elasticsearch/reference/current/settings.html#static-cluster-setting)）断路器的底层类型。有两个有效选项：`noop` 和 `memory`。`noop` 表示断路器不采取任何措施来防止过多的内存使用。`memory` 表示断路器会跟踪经过训练的模型使用的内存，并可能中断和防止 `OutOfMemory` 错误。默认值为 `memory`。