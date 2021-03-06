# MetricsServlet JSON Metrics Sink

`MetricsServlet` is a [metrics sink](Sink.md) that gives [metrics snapshots](#getMetricsSnapshot) in [JSON](#mapper) format.

`MetricsServlet` is a ["special" sink](spark-metrics-MetricsSystem.adoc#metricsServlet) as it is only available to the metrics instances with a web UI:

* Driver of a Spark application
* Spark Standalone's `Master` and `Worker`

You can access the metrics from `MetricsServlet` at [/metrics/json](#path) URI by default. The entire URL depends on a metrics instance, e.g. http://localhost:4040/metrics/json/ for a running Spark application.

```text
$ http http://localhost:4040/metrics/json/
HTTP/1.1 200 OK
Cache-Control: no-cache, no-store, must-revalidate
Content-Length: 5005
Content-Type: text/json;charset=utf-8
Date: Mon, 11 Jun 2018 06:29:03 GMT
Server: Jetty(9.3.z-SNAPSHOT)
X-Content-Type-Options: nosniff
X-Frame-Options: SAMEORIGIN
X-XSS-Protection: 1; mode=block

{
    "counters": {
        "local-1528698499919.driver.HiveExternalCatalog.fileCacheHits": {
            "count": 0
        },
        "local-1528698499919.driver.HiveExternalCatalog.filesDiscovered": {
            "count": 0
        },
        "local-1528698499919.driver.HiveExternalCatalog.hiveClientCalls": {
            "count": 0
        },
        "local-1528698499919.driver.HiveExternalCatalog.parallelListingJobCount": {
            "count": 0
        },
        "local-1528698499919.driver.HiveExternalCatalog.partitionsFetched": {
            "count": 0
        },
        "local-1528698499919.driver.LiveListenerBus.numEventsPosted": {
            "count": 7
        },
        "local-1528698499919.driver.LiveListenerBus.queue.appStatus.numDroppedEvents": {
            "count": 0
        },
        "local-1528698499919.driver.LiveListenerBus.queue.executorManagement.numDroppedEvents": {
            "count": 0
        }
    },
    ...
```

`MetricsServlet` is <<creating-instance, created>> exclusively when `MetricsSystem` is link:spark-metrics-MetricsSystem.adoc#start[started] (and requested to link:spark-metrics-MetricsSystem.adoc#registerSinks[register metrics sinks]).

`MetricsServlet` can be configured using configuration properties with *sink.servlet* prefix (in link:spark-metrics-MetricsConfig.adoc[metrics configuration]). That is not required since `MetricsConfig` link:spark-metrics-MetricsConfig.adoc#setDefaultProperties[makes sure] that `MetricsServlet` is always configured.

`MetricsServlet` uses https://fasterxml.github.io/jackson-databind/[jackson-databind], the general data-binding package for Jackson (as <<mapper, ObjectMapper>>) with https://metrics.dropwizard.io/3.1.0/[Dropwizard Metrics] library (i.e. registering a Coda Hale `MetricsModule`).

[[properties]]
.MetricsServlet's Configuration Properties
[cols="1,1,2",options="header",width="100%"]
|===
| Name
| Default
| Description

| `path`
| `/metrics/json/`
| [[path]] Path URI prefix to bind to

| `sample`
| `false`
| [[sample]] Whether to show entire set of samples for histograms
|===

[[internal-registries]]
.MetricsServlet's Internal Properties (e.g. Registries, Counters and Flags)
[cols="1,2",options="header",width="100%"]
|===
| Name
| Description

| `mapper`
| [[mapper]] Jaxson's https://fasterxml.github.io/jackson-databind/javadoc/2.6/com/fasterxml/jackson/databind/ObjectMapper.html[com.fasterxml.jackson.databind.ObjectMapper] that _"provides functionality for reading and writing JSON, either to and from basic POJOs (Plain Old Java Objects), or to and from a general-purpose JSON Tree Model (JsonNode), as well as related functionality for performing conversions."_

When created, `mapper` is requested to register a Coda Hale https://metrics.dropwizard.io/3.1.0/apidocs/com/codahale/metrics/json/MetricsModule.html[com.codahale.metrics.json.MetricsModule].

Used exclusively when `MetricsServlet` is requested to <<getMetricsSnapshot, getMetricsSnapshot>>.

| `servletPath`
| [[servletPath]] Value of <<path, path>> configuration property

| `servletShowSample`
| [[servletShowSample]] Flag to control whether to show samples (`true`) or not (`false`).

`servletShowSample` is the value of <<sample, sample>> configuration property (if defined) or `false`.

Used when <<mapper, ObjectMapper>> is requested to register a Coda Hale https://metrics.dropwizard.io/3.1.0/apidocs/com/codahale/metrics/json/MetricsModule.html[com.codahale.metrics.json.MetricsModule].
|===

=== [[creating-instance]] Creating MetricsServlet Instance

`MetricsServlet` takes the following when created:

* [[property]] Configuration Properties (as Java `Properties`)
* [[registry]] Dropwizard Metrics' https://metrics.dropwizard.io/3.1.0/apidocs/com/codahale/metrics/MetricRegistry.html[MetricRegistry]
* [[securityMgr]] `SecurityManager`

`MetricsServlet` initializes the <<internal-registries, internal registries and counters>>.

=== [[getMetricsSnapshot]] Requesting Metrics Snapshot -- `getMetricsSnapshot` Method

[source, scala]
----
getMetricsSnapshot(request: HttpServletRequest): String
----

`getMetricsSnapshot` simply requests the <<mapper, Jackson ObjectMapper>> to serialize the <<registry, MetricRegistry>> to a JSON string (using link:++https://fasterxml.github.io/jackson-databind/javadoc/2.6/com/fasterxml/jackson/databind/ObjectMapper.html#writeValueAsString-java.lang.Object-++[ObjectMapper.writeValueAsString]).

NOTE: `getMetricsSnapshot` is used exclusively when `MetricsServlet` is requested to <<getHandlers, getHandlers>>.

=== [[getHandlers]] Requesting JSON Servlet Handler -- `getHandlers` Method

[source, scala]
----
getHandlers(conf: SparkConf): Array[ServletContextHandler]
----

`getHandlers` returns just a single `ServletContextHandler` (in a collection) that gives <<getMetricsSnapshot, metrics snapshot>> in JSON format at every request at <<servletPath, servletPath>> URI path.

NOTE: `getHandlers` is used exclusively when `MetricsSystem` is requested for link:spark-metrics-MetricsSystem.adoc#getServletHandlers[metrics ServletContextHandlers].
