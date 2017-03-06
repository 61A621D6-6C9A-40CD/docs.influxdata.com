---
title: Cluster Commands
menu:
  enterprise_1_2:
    weight: 10
    parent: Features
---

Use the command line tools [`influxd-ctl`](#influxd-ctl) and [`influx`](#influx) to interact with your cluster and data.

## influxd-ctl

The `influxd-ctl` tool is available on all [meta nodes](/enterprise/v1.2/concepts/glossary/#meta-node).
Use `influxd-ctl` to manage your cluster nodes, backup data, restore data, and rebalance your cluster.

### Syntax

```
influxd-ctl [global-options] <command> [arguments]
```

### Global Options

`-auth-type [ none | basic | jwt ]`  
&emsp;&emsp;&emsp;Specify the type of authentication to use. The default authentication type is `none`.

`-bind <hostname>:<port>`  
&emsp;&emsp;&emsp;Specify the bind HTTP address of a meta node to connect to. The default is `localhost:8091`.

`-bind-tls`  
&emsp;&emsp;&emsp;Use TLS.

`-config '<path-to-configuration-file>'`  
&emsp;&emsp;&emsp;Specify the path to the configuration file.

`-pwd <password>`  
&emsp;&emsp;&emsp;Specify the user’s password. This option is ignored if `-auth-type basic` isn’t specified.

`-k`  
&emsp;&emsp;&emsp;Skip certificate verification; use this option a self-signed certificate. `-k` is ignored if `-bind-tls` isn't specified.

`-secret <JWT-shared-secret>`  
&emsp;&emsp;&emsp;Specify the JWT shared secret. This option is ignored if `-auth-type jwt` isn't specified.

`-user <username>`  
&emsp;&emsp;&emsp;Specify the user’s username. This option is ignored if `-auth-type basic` isn’t specified.

### Examples of Global Options

The following examples use the tool's [`show` argument](#show).
See the sections below for more information about `influxd-ctl`'s arguments. 

#### Example 1: Bind to a remote meta node

```
$ influxd-ctl -bind meta-node-02:8091 show
```
The tool binds to the meta node with the hostname `meta-node-02` at port `8091`.

#### Example 2: Authenticate with a JSON Web Token (JWT)

```
$ influxd-ctl -auth-type jwt -secret oatclusters show
```
The tool uses JWT authentication with the shared secret `oatclusters`.

If authentication is enabled in the cluster's [meta node configuration files](/enterprise/v1.2/administration/configuration/#auth-enabled-false) and [data node configuration files](/enterprise/v1.2/administration/configuration/#meta-auth-enabled-false) and the `influxd-ctl` command does not include authentication details, the system returns:
```
Error: unable to parse authentication credentials.
```

If authentication is enabled and the `influxd-ctl` command provides the incorrect shared secret, the system returns:
```
Error: signature is invalid.
```

#### Example 3: Authenticate with basic authentication

```
$ influxd-ctl -auth-type basic -user admini -pwd mouse show
```
The tool uses basic authentication for the cluster user `admini` with the password `mouse`.

If authentication is enabled in the cluster's [meta node configuration files](/enterprise/v1.2/administration/configuration/#auth-enabled-false) and [data node configuration files](/enterprise/v1.2/administration/configuration/#meta-auth-enabled-false) and the `influxd-ctl` command does not include authentication details, the system returns:
```
Error: unable to parse authentication credentials.
```

If authentication is enabled and the `influxd-ctl` command provides the incorrect username or password, the system returns:
```
Error: authorization failed.
```

### Arguments

#### add-data

Adds a data node to a cluster.
Use `add-data` instead of the [`join` argument](#join) when performing a [Production Installation](/enterprise/v1.2/production_installation/data_node_installation/) of an InfluxEnterprise cluster.

```
add-data <data-node-TCP-bind-address>
```

**Resources:** [Production Installation](/enterprise/v1.2/production_installation/data_node_installation/)   

#### Examples

##### Example 1: Add a data node to a cluster
<br>
```
$ influxd-ctl add-data cluster-data-node:8088

Added data node 7 at cluster-data-node:8088
```

The command adds a data node to a cluster.
The data node has the hostname `cluster-data-node` and runs on port `8088`.

#### add-meta         

Adds a meta node to a cluster.
Use `add-meta` instead of the [`join` argument](#join) when performing a [Production Installation](/enterprise/v1.2/production_installation/meta_node_installation/) of an InfluxEnterprise cluster.
```
add-meta <meta-node-TCP-bind-address>
```

**Resources:** [Production Installation](/enterprise/v1.2/production_installation/data_node_installation/)   

#### Examples

##### Example 1: Add a meta node to a cluster
<br>
```
$ influxd-ctl add-meta cluster-meta-node:8091

Added meta node 3 at cluster-meta-node:8091
```

The command adds a meta node to a cluster.
The meta node has the hostname `cluster-meta-node` and runs on port `8091`.


#### backup           
  
Creates a backup of a cluster's [metastore](/influxdb/v1.2/concepts/glossary/#metastore) and [shard](/influxdb/v1.2/concepts/glossary/#shard) data at that point in time and stores the copy in the specified directory.
Backups are incremental by default; they create a copy of the metastore and shard data that have changed since the previous incremental backup.
If there are no existing incremental backups, the system automatically performs a complete backup.
```
backup [ -db <database> | -from <data-node-TCP-bind-address> | -full | -rp <retention-policy> | -shard <shard-id> ] <backup-directory>
```
Options:

`-db <database>`:  
&emsp;&emsp;&emsp;The name of the single database to back up.

`-from <data-node-TCP-address>`:  
&emsp;&emsp;&emsp;The TCP address of the target data node.

`-full`:  
&emsp;&emsp;&emsp;Perform a [full](/enterprise/v1.2/guides/backup-and-restore/#backup) backup. 

`-rp <retention-policy>`:  
&emsp;&emsp;&emsp;The name of the single [retention policy](/influxdb/v1.2/concepts/glossary/#retention-policy-rp) to back up (requires the `-db` flag).

`-shard <shard-id>`:   
&emsp;&emsp;&emsp;The ID of the single shard to back up.

> Restoring a `-full` backup and restoring an incremental backup require different syntax.
To prevent issues with [`restore`](#restore), keep `-full` backups and incremental backups in separate directories.

<dt> In versions 1.2.0 and 1.2.1, there is a known issue with restores from a backup directory
that stores several **different** incremental backups.
For a [restore](#restore) to function properly, incremental backups that specify different
options (for example: they specify a different database with `-db` or a
different retention policy with `-rp`) must be stored in different directories.
If a single backup directory stores several different incremental backups, a
restore only restores the most recent incremental backup.
This issue is fixed in version 1.2.2.
</dt>

**Resources:** [Backup and Restore](/enterprise/v1.2/guides/backup-and-restore/)

#### Examples

##### Example 1: Perform an incremental backup
<br>
```
$ influxd-ctl backup .
```

Output:
```
Backing up meta data... Done. 421 bytes transferred
Backing up node 7ba671c7644b:8088, db telegraf, rp autogen, shard 4... Done. Backed up in 903.539567ms, 307712 bytes transferred
Backing up node bf5a5f73bad8:8088, db _internal, rp monitor, shard 1... Done. Backed up in 138.694402ms, 53760 bytes transferred
Backing up node 9bf0fa0c302a:8088, db _internal, rp monitor, shard 2... Done. Backed up in 101.791148ms, 40448 bytes transferred
Backing up node 7ba671c7644b:8088, db _internal, rp monitor, shard 3... Done. Backed up in 144.477159ms, 39424 bytes transferred
Backed up to . in 1.293710883s, transferred 441765 bytes

$ ls
20160803T222310Z.manifest  20160803T222310Z.s1.tar.gz  20160803T222310Z.s3.tar.gz
20160803T222310Z.meta      20160803T222310Z.s2.tar.gz  20160803T222310Z.s4.tar.gz
```

The command performs an incremental backup and stores it in the current directory.
If there are any existing backups the current directory, the system performs an incremental backup.
If there aren’t any existing backups in the current directory, the system performs a complete backup of the cluster.

##### Example 2: Perform a full backup
<br>
```
$ influxd-ctl backup -full backup_dir
```

Output:
```
Backing up meta data... Done. 481 bytes transferred
Backing up node <hostname>:8088, db _internal, rp monitor, shard 1... Done. Backed up in 33.207375ms, 238080 bytes transferred
Backing up node <hostname>:8088, db telegraf, rp autogen, shard 2... Done. Backed up in 15.184391ms, 95232 bytes transferred
Backed up to backup_dir in 51.388233ms, transferred 333793 bytes

~# ls backup_dir
20170130T184058Z.manifest
20170130T184058Z.meta
20170130T184058Z.s1.tar.gz
20170130T184058Z.s2.tar.gz
```

The command performs a full backup of the cluster and stores the backup in the existing directory `backup_dir`.

#### copy-shard       

Copies a shard from a source data node to a destination data node.
```
copy-shard <data-node-source-TCP-address> <data-node-destination-TCP-address> <shard-id>
```

**Resources:** [Cluster Rebalance](/enterprise/v1.2/guides/rebalance/) 

#### Examples

##### Example 1: Copy a shard from one data node to another data node
<br>
```
$ influxd-ctl copy-shard data-node-01:8088 data-node-02:8088 22'

Copied shard 22 from data-node-01:8088 to data-node-02:8088
```

The command copies the shard with the id `22` from the data node at `data-node-01:8088` to the data node at `data-node-02:8088`.

#### copy-shard-status

Shows all in-progress copy shard operations, including the relevant shard's source node, destination node, database, retention policy, shard id, total size, current size, and the operation's start time.
```
copy-shard-status
```

#### Examples

##### Example 1: Show all in-progress copy shard operations
<br>
```
~# influxd-ctl copy-shard-status

Source	Dest	Database	Policy	ShardID	TotalSize	CurrentSize	StartedAt

TODO
```

The command returns all in-progress copy shard operations.

#### join             

Joins a meta node and/or data node to the cluster.
Use `join` instead of the [`add-meta`](#add-meta) or [`add-data`](#add-data) arguments when performing a [QuickStart Installation](/enterprise/v1.2/quickstart_installation/cluster_installation/) of an InfluxEnterprise cluster.
```
join [-v] <meta-node-TCP-bind-address>
```

Options:

`-v`  
&emsp;&emsp;&emsp;Prints verbose information about the join.

**Resources:** [QuickStart Installation](/enterprise/v1.2/quickstart_installation/cluster_installation/)

#### Examples

##### Example 1: Join a meta and data node to an existing cluster
<br>
```
$ influxd-ctl join meta-node-02:8091

Joining meta node at meta-node-02:8091
Searching for meta node on meta-node-03:8091...
Searching for data node on data-node-03:8088...

Successfully joined cluster

  * Added meta node 3 at meta-node-03:8091
  * Added data node 4 at data-node-03:8088
```

The command joins the meta node running on `meta-node-03:8091` and the data node running on `meta-node-03:8088` to an existing cluster.
The existing cluster includes the meta node running on `meta-node-02:8091`.
 
##### Example 2: Join a meta node to an existing cluster
<br>
```
~# influxd-ctl join meta-node-02:8091

Joining meta node at meta-node-02:8091
Searching for meta node on meta-node-03:8091...
Searching for data node on data-node-03:8088...

Successfully joined cluster

  * Added meta node 18 at meta-node-03:8091
  * No data node added.  Run with -v to see more information
```

The command joins the meta node running on `meta-node-03:8091` to an existing cluster.
The existing cluster includes the meta node running on `meta-node-02:8091`.

##### Example 3: Join a meta node to an existing cluster and show detailed information about the join
<br>
```
~# influxd-ctl join -v meta-node-02:8091

Joining meta node at meta-node-02:8091
Searching for meta node on meta-node-03:8091...
Searching for data node on data-node-03:8088...

No data node found on data-node-03:8091!

  If a data node is running on this host,
  you may need to add it manually using the following command:

  influxd-ctl -bind meta-node-02:8091 add-data <dataAddr:port>

  Common problems:

    * The influxd process is using a non-standard port (default 8088)
    * The influxd process it not running.  Check the logs for startup errors

Successfully joined cluster

  * Added meta node 18 at meta-node-03:8091
  * No data node added.  Run with -v to see more information
```

The command joins the meta node running on `meta-node-03:8091` to an existing cluster.
The existing cluster includes the meta node running on `meta-node-02:8091`.

#### kill-copy-shard  

Aborts an in-progress `copy-shard` command.
```
kill-copy-shard <data-node-source-TCP-address> <data-node-destination-TCP-address> <shard-id>
```

#### leave            

Removes a meta node from the cluster.
```
leave [-y]
```
`-y` assumes `Yes` to all prompts.

#### remove-data      

Removes a data node from the cluster.
```
remove-data [-force] <data-node-TCP-bind-address>
```
`-force` forces the removal of the data node.
Use `-force` if the data node is down.

#### remove-meta      

Removes a meta node from the cluster.
```
remove-meta <meta-node-TCP-bind-address> [-force | -tcpAddr <meta-node-TCP-bind_address> | -y]
```
* `-force` forces the removal of the meta node. Use `-force` if the meta node is down.
* `-y` assumes `Yes` to all prompts.

#### remove-shard     

Removes a shard from a data node.
Please see the [Cluster Rebalance](/enterprise/v1.2/guides/rebalance/) guide for more information.
```
remove-shard <data-node-source-TCP-address> <shard-id>
```

#### restore          

Restores an existing backup of a cluster.
Please see the [Backup and Restore](/enterprise/v1.2/guides/backup-and-restore/) guide for more information.
```
restore [ -db <database> | -full | -list | -newdb <new-database> | -newrf <int> | -newrp <retention-policy> | -rp <retention policy | shard <shard-id> ] ( <path-to-backup-manifest-file> | <path-to-backup-directory> )
```

#### show             

Shows all meta nodes and data nodes that are part of the cluster.
```
show
```

#### show-shards      

Shows the cluster's existing shards, including the shard id, database, retention policy, shard group, start time, end time, expiry time, and data node owners.
```
show-shards
```

#### update-data      

Updates a data node's address in the [meta store](/enterprise/v1.2/concepts/glossary/#meta-service).
```
update-data <data-node-old-TCP-bind-address> <data-node-new-TCP-bind-address>
```

#### token            

Generates a signed JWT token.
```
token [-exp <duration>]
```
`-exp` determines the time after which the token expires.
By default, the token expires after one minute.

#### truncate-shards  

Truncates hot shards.
Please see the [Cluster Rebalance](/enterprise/v1.2/guides/rebalance/) guide for more information.
```
truncate-shards [-delay <duration>]
```
`-delay` determines when to truncate shards after [`now()`](/influxdb/v1.2/concepts/glossary/#now).
By default, the delay is set to one minute.

## influx

The `influx` tool, also known as the Command Line Interface (CLI), is available on all [data nodes](/enterprise/v1.2/concepts/glossary/#data-node).
Use `influx` to write data to your cluster, query data interactively, and view query output in different formats.

The complete description of the `influx` tool is available in the [InfluxDB documentation](/influxdb/v1.2/tools/shell/).
