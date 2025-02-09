# Monitoring the state of a cluster and hosts

Using monitoring tools in the management console, you can track the status of a{{ mkf-name }} cluster and its individual hosts. These tools display diagnostic information in the form of charts.

{% include [monitoring-provides](../../_includes/mdb/monitoring-provides.md) %}

{% include [monitoring-freq](../../_includes/mdb/monitoring-freq.md) %}

{% include [note-monitoring-auto-units](../../_includes/mdb/note-monitoring-auto-units.md) %}

## Monitoring cluster status {#monitoring-cluster}

To view detailed information about the {{ mkf-name }} cluster status:

1. Go to the folder page and select **{{ mkf-name }}**.
1. Click on the name of the cluster and open the **Monitoring** tab.

The page displays the following charts:

* **Alive brokers**: Number of functional brokers for each host with the `KAFKA` role.
* **Bytes In/Out**: Write and message read speed for each host with the `KAFKA` role (bytes per second).
* **Errors**: Number of failed requests by error type.
* **Free space**: Free disk space for each host with the `KAFKA` role (in bytes).
* **Messages in**: Message write rate (messages per second).
* **Offline partitions**: Displays the parameter values below:
    * `OfflineReplicaCount`: Number of partitions without a leader. These partitions do not support message writes or reads.
    * `Uderreplicated partitions`: Number of partitions with a number of in-sync replicas (ISRs) below the replication factor.
    * `Under min ISR partitions`: Number of partitions with a number of in-sync replicas below the minimum allowed value specified in the [settings](../concepts/settings-list.md).
* **Replicas**: Total number of partition replicas.
* **Replicated bytes**: Replication data streaming rate for each host with the `KAFKA` role (bytes per second).
* **Replication lag**: Greatest replication lag for each host with the `KAFKA` role (messages).
* **Request time (0.95 quantile)**: Request processing time in the 0.95 quantile by request type.
* **Requests**: Request rate by request type (requests per second).
* **Top 5 topics by size**: Amount of data for each of the 5 largest topics (bytes).

## Monitoring the state of hosts {#monitoring-hosts}

To view detailed information about the status of individual {{ mkf-name }} hosts:

1. Go to the folder page and select **{{mkf-name }}**.
1. Click the name of the desired cluster and select **Hosts** → **Monitoring**.
1. Select the host from the drop-down list.

This page displays charts showing the load on an individual host in the cluster:

* **CPU**: The loading of processor cores. As the load goes up, the `Idle` value goes down.
* **Disk Bytes**: The speed of disk operations (bytes per second).
* **Disk IOPS**: The number of disk operations per second.
* **Memory**: The use of RAM in bytes. At high loads, the value of `Free` goes down while the other values go up.
* **Network Bytes**: The speed of data exchange over the network (bytes per second).
* **Network Packets**: The number of packets exchanged over the network per second.

## Integration with {{ monitoring-full-name }} {#monitoring-integration}

To set up [cluster](#monitoring-cluster) and [host](#monitoring-hosts) status metric alerts:

1. In the Management console, select the folder with the cluster you wish to configure alerts for.
1. Click the ![image](../../_assets/ugly-sandwich.svg) icon and select **Monitoring**.
1. Under **Service dashboards**, select **{{ mkf-name }} — Cluster Overview**.
1. On the desired metrics chart, click ![options](../../_assets/horizontal-ellipsis.svg) and select **Create alert**.
1. If there are multiple metrics on a chart, select a data query to generate a metric and click **Continue**. For more on the query language, [see the {{ monitoring-full-name }} documentation](../../monitoring/concepts/querying.md).
1. Set the `Alarm` and `Warning` notification threshold values.
1. Click **Create alert**.

To have other cluster health indicators monitored automatically:

1. [Create an alert](../../monitoring/operations/alert/create-alert.md).
1. Add a status metric.
1. Set the alert threshold values in the alert settings.

Recommended threshold values:

| Metric                       | Parameter                                               | `Alarm`                 | `Warning`               |
| ---------------------------: | :-----------------------------------------------------: | :---------------------: | :---------------------: |
| Number of healthy hosts      | `kafka_is_alive`                                        | `<number of hosts> - 2` | `<number of hosts> - 1` |
| Partition replication status | `kafka_server_ReplicaManager_UnderReplicatedPartitions` | —                       | `Greater than 0`        |
| Number of lagging replicas   | `kafka_server_ReplicaManager_UnderMinIsrPartitionCount` | `Greater than 0`        | —                       |
| Storage space used           | `disk.used_bytes`                                       | `90% of storage size`   | `80% of storage size`   |
