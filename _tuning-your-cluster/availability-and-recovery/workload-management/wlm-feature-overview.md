---
layout: default
title: Workload management
nav_order: 70
has_children: true
parent: Availability and recovery
---

Introduced 2.18
{: .label .label-purple }

# Workload management

Workload management allows users to group search traffic and isolate network resources, preventing the overuse of network resources by specific requests. It offers the following benefits:

- Tenant-level admission control and reactive query management. When resource usage exceeds configured limits, it automatically identifies and cancels demanding queries, ensuring fair resource distribution.

- Tenant-level isolation within the cluster for search workloads, operating at a node level.

## Installing workload management

To install workload management, use the following command: 

```json
./bin/opensearch-plugin install workload-management
```
{% include copy-curl.html %}

## Query groups

A _query group_ is a logical group of tasks with defined resource limits. System administrators can dynamically manage query groups using the Workload management APIs. These query groups can be used to make search requests with resource limits. 

### Permissions

Only users with administrator-level permissions can create and update query groups using the Workload management APIs.

### Operating modes

The following operating modes determine the operating-level for the query group:

- **Disabled mode**: Workload management is disabled.

- **Enabled mode**: Workload management is enabled and will cause cancellations and rejection once the query group’s configured thresholds are reached.

- **Monitor_only mode** (Default): Workload management will monitor tasks but it will not cancel/reject any queries.

### Example request

The following example adds a query group with the named `analytics`:

```json
PUT _wlm/query_group
{
  “name”: “analytics”,
  “resiliency_mode”: “enforced”,
  “resource_limits”: {
    “cpu”: 0.4,
    “memory”: 0.2
  }
}
```
{% include copy-curl.html %}

When creating a query group, make sure that the sum of the resource limits for a single resource, such as `cpu` or `memory`, does not exceed `1`.

### Example response

OpenSearch responds with the set resource limits and the `_id` for the query group:

```json
{
  "_id":"preXpc67RbKKeCyka72_Gw",
  "name":"analytics",
  "resiliency_mode":"enforced",
  "resource_limits":{
    "cpu":0.4,
    "memory":0.2
  },
  "updated_at":1726270184642
}
```

## Using `queryGroupID`

You can associate a query request with a `queryGroupID` to manage and allocate resources within the limits defined by the query group. By utilizing this ID, requests are routed and tracked under the query group, ensuring resource quotas and task limits are maintained.

The following example query uses the `queryGroupId` to ensure that the query stays under that query group's resource limits:

```json
{
  "_id":"preXpc67RbKKeCyka72_Gw",
  "name":"analytics",
  "resiliency_mode":"enforced",
  "resource_limits":{
    "cpu":0.4,
    "memory":0.2
  },
  "updated_at":1726270184642
}
```
{% include copy-curl.html %}

## Workload management settings

The are following settings can be used to customize workload management using the `_cluster/settings` API:

| **Setting name**  | **Description**  |
| :--- | :--- |
| `wlm.query_group.duress_streak` | Determines the node duress threshold. Once the threshold is reached, the node is marked as `in duress`. |
| `wlm.query_group.enforcement_interval`  | Defines the monitoring interval. |
| `wlm.query_group.mode`  | Defines the [operating mode](#operating-modes). |
| `wlm.query_group.node.memory_rejection_threshold` | Defines the query group level `memory` threshold. When the threshold is reached, the request is rejected. |
| `wlm.query_group.node.cpu_rejection_threshold` | Defines query group level `cpu` threshold. When the threshold is reached, the request is rejected. |
| `wlm.query_group.node.memory_cancellation_threshold` | Controls whether the node is considered in duress when the `cpu` threshold is reached and the effective request cancellation threshold based on `memory` usage. |
| `wlm.query_group.node.cpu_cancellation_threshold`    | Controls whether the node is considered in duress when the `cpu` threshold is reached and the effective request cancellation threshold on `cpu` usage. |

When setting rejection and cancellation settings thresholds, remember that the rejection threshold for a resource should always be less than the cancellation threshold. 

### Operating modes

The following operating modes determine the operating-level for the query group:

- **Disabled mode**: Workload management is disabled.

- **Enabled mode**: Workload management is enabled and will cause cancellations and rejection once the query group’s configured thresholds are reached.

- **Monitor_only mode** (Default): Workload management will monitor tasks but it will not cancel/reject any queries.

## Workload management stats API

The Workload management stats API returns workload management metrics for a query group, using the following method:

```json
GET _wlm/stats
```
{% include copy-curl.html %}

### Example response 

```json
{
  “_nodes”: {
    “total”: 1,
    “successful”: 1,
    “failed”: 0
  },
  “cluster_name”: “XXXXXXYYYYYYYY”,
  “A3L9EfBIQf2anrrUhh_goA”: {
    “query_groups”: {
      “16YGxFlPRdqIO7K4EACJlw”: {
        “total_completions”: 33570,
        “total_rejections”: 0,
        “total_cancellations”: 0,
        “cpu”: {
          “current_usage”: 0.03319935314357281,
          “cancellations”: 0,
          “rejections”: 0
        },
        “memory”: {
          “current_usage”: 0.002306486276211217,
          “cancellations”: 0,
          “rejections”: 0
        }
      },
      “DEFAULT_QUERY_GROUP”: {
        “total_completions”: 42572,
        “total_rejections”: 0,
        “total_cancellations”: 0,
        “cpu”: {
          “current_usage”: 0,
          “cancellations”: 0,
          “rejections”: 0
        },
        “memory”: {
          “current_usage”: 0,
          “cancellations”: 0,
          “rejections”: 0
        }
      }
    }
  }
}
```
{% include copy-curl.html %}

### Response body fields 

| Field name | Description |
| :--- | :--- | 
| `total_completions`  |  The total number of request completions in this `query_group` at the given node. This includes all shard-level and coordinator-level requests. |
| `total_rejections`    | The total number request rejections in this `query_group` at the given node. This includes all shard-level and coordinator-level requests.    |
| `total_cancellations` | The total number of cancellations in this `query_group` at the given node. This includes all shard-level and coordinator-level requests.  |
| `cpu`   | The `cpu` resource type stats for the `query_group`  | 
| `memory`  | The `memory` resource type stats for the `query_group`  | 

### Resource type stats

| Field name  | Description   |
| :--- | :---- | 
| `current_usage` |The resource usage for `query_group` at the given node based on the last run of the monitoring thread. This value is updated based on the `wlm.query_group.enforcement_interval`. |
| `cancellations` | The cancellation count as a result of the cancellation threshold being reached.   |
| `rejections`    |  The rejection count as a result of the cancellation threshold being reached.   |
