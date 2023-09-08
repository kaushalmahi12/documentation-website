---
layout: default
title: Metric records
nav_order: 30
parent: Metrics
---

# Metric records

OpenSearch Benchmark stores metrics in the `benchmark-metrics-*` indices, which creates a new index each month. The following is in example metric record stored in the `benchmark-metrics-2023-08` index.

```json
{
  "_index": "benchmark-metrics-2023-08",
  "_id": "UiNY4YkBpMtdJ7uj2rUe",
  "_version": 1,
  "_score": null,
  "_source": {
    "@timestamp": 1691702842821,
    "relative-time-ms": 65.90720731765032,
    "test-execution-id": "8c43ee4c-cb34-494b-81b2-181be244f832",
    "test-execution-timestamp": "20230810T212711Z",
    "environment": "local",
    "workload": "geonames",
    "test_procedure": "append-no-conflicts",
    "provision-config-instance": "external",
    "name": "service_time",
    "value": 607.8001195564866,
    "unit": "ms",
    "sample-type": "normal",
    "meta": {
      "source_revision": "unknown",
      "distribution_version": "1.1.0",
      "distribution_flavor": "oss",
      "index": "geonames",
      "took": 13,
      "success": true,
      "success-count": 125,
      "error-count": 0
    },
    "task": "index-append",
    "operation": "index-append",
    "operation-type": "bulk"
  },
  "fields": {
    "@timestamp": [
      "2023-08-10T21:27:22.821Z"
    ],
    "test-execution-timestamp": [
      "2023-08-10T21:27:11.000Z"
    ]
  },
  "highlight": {
    "workload": [
      "@opensearch-dashboards-highlighted-field@geonames@/opensearch-dashboards-highlighted-field@"
    ],
    "meta.index": [
      "@opensearch-dashboards-highlighted-field@geonames@/opensearch-dashboards-highlighted-field@"
    ]
  },
  "sort": [
    1691702831000
  ]
}
```

The following fields are configurable in the `opensearch-benchmarks-metrics-*` file, as found in the `_source` section of the metric's record:

## @timestamp

The timestamp in milliseconds since the epoch determined when the sample was taken. For request-related metrics, such as `latency`` or `service_time`` this is the timestamp when OpenSearch Benchmark has issued the request.

## relative-time-ms

The relative time in milliseconds since the start of the benchmark. This is useful for comparing time-series graphs over multiple tests. For example, you could compare the indexing throughput over time across multiple tests. 

## test-execution-id

A UUID which changes on every invocation of the workload. It is intended to group all samples of a benchmarking run.

## test-execution-timestamp

The timestamp (always in UTC) for when the workload was invoked.

## environment

The environment describes the origin of a metric record. This is defined when initially [configuring]({{site.url}}{{site.baseurl}}/benchmark/configuring-benchmark) OpenSearch Benchmark. You can use seperate environments for different Benchmarks, but store the metric records in the same index.

## workload, test_procedure, provision-config-instance

The workload, test procedures, and the configuration instances for which the metrics are produced.

## name, value, unit

The actual metric name and value with an optional unit. Depending on the nature of a metric, it is either sampled periodically by OpenSearch Benchmark, for example, CPU utilization or query latency, or measured once, for example, the final size of the index.

## sample-type

Determines whether to configure a benchmark to run in warmup mode by setting to `warmup` or `normal`. Only `normal` samples are considered for the results that are reported.

## meta

The meta information for each metric record, including

- CPU info: The number of physical and logical cores and also the model name
- OS info: OS name and version
- Host name
- Node name: A unique name for each node when OpenSearch Benchmark provisions the cluster.
- Source revision: The git hash of the version of OpenSearch that is benchmarked. 
- Distribution version: The distribution version of Elasticsearch that is benchmarked. 
- Custom tags: You can define custom tags with the command line flag --user-tags. The tags are prefixed by tag_ in order to avoid accidental clashes with OpenSearch Benchmark's internal tags.
- Operation-specific: The optional substructure operation contains additional information depending on the type of operation. For bulk requests, this may be the number of documents or for searches the number of hits.

Depending on the level of metric record, some meta information might be missing.

## Next steps

- for more information about how to access OpenSearch Benchmark metrics, see [Metrics]({{site.url}}{{site.baseurl}}/benchmark/metrics/index/).
- For more information about the metrics stored in OpenSearch Benchmark, see [Metric keys]({{site.url}}{{site.baseurl}}/benchmark/metrics/metric-keys).
