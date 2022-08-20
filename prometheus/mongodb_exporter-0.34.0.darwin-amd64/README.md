# MongoDB exporter
[![Release](https://img.shields.io/github/release/percona/mongodb_exporter.svg?style=flat)](https://github.com/percona/mongodb_exporter/releases/latest)
[![Build Status](https://github.com/percona/mongodb_exporter/actions/workflows/go.yml/badge.svg?branch=main)](https://github.com/percona/mongodb_exporter/actions/workflows/go.yml?query=branch%3Amain)
[![codecov.io Code Coverage](https://img.shields.io/codecov/c/github/percona/mongodb_exporter.svg?maxAge=2592000)](https://codecov.io/github/percona/mongodb_exporter?branch=main)
[![Go Report Card](https://goreportcard.com/badge/github.com/percona/mongodb_exporter)](https://goreportcard.com/report/github.com/percona/mongodb_exporter)
[![CLA assistant](https://cla-assistant.percona.com/readme/badge/percona/mongodb_exporter)](https://cla-assistant.percona.com/percona/mongodb_exporter)
[![Discord](https://img.shields.io/discord/808660945513611334?style=flat)](http://per.co.na/discord)

This is the new MongoDB exporter implementation that handles ALL metrics exposed by MongoDB monitoring commands.
This new implementation loops over all the fields exposed in diagnostic commands and tries to get data from them.

Currently, these metric sources are implemented:
- $collStats
- $indexStats
- getDiagnosticData
- replSetGetStatus
- serverStatus

## Info on Percona MongoDB exporter versions

The old 0.1x.y version (ex `master` branch) has been moved to the `release-0.1x` branch.

A flag, `--compatible-mode`, which exposes metrics with 0.1x compatible metric
names has been implemented which simplifies migration from the old version to
the current version.

### Build the exporter
The build process uses the dockerized version of goreleaser so you don't need to install Go.
Just run `make release` and the new binaries will be generated under the build directory.
```
├── build
│ ├── config.yaml
│ ├── mongodb_exporter_7c73946_checksums.txt
│ ├── mongodb_exporter-7c73946.darwin-amd64.tar.gz
│ ├── mongodb_exporter-7c73946.linux-amd64.tar.gz
│ ├── mongodb_exporter_darwin_amd64
│ │ └── mongodb_exporter <--- MacOS binary
│ └── mongodb_exporter_linux_amd64
│ └── mongodb_exporter <--- Linux binary
```
### Running the exporter
If you built the exporter using the method mentioned in the previous section, the generated binaries are in `mongodb_exporter_linux_amd64/mongodb_exporter` or `mongodb_exporter_darwin_amd64/mongodb_exporter`

#### Docker
A docker image is available on the [official percona repository](https://hub.docker.com/r/percona/mongodb_exporter).

##### Examples

```sh
# with podman
podman run -d -p 9216:9216 -p 17001:17001 percona/mongodb_exporter:0.20 --mongodb.uri=mongodb://127.0.0.1:17001

# with docker
docker run -d -p 9216:9216 -p 17001:17001 percona/mongodb_exporter:0.20 --mongodb.uri=mongodb://127.0.0.1:17001
```

#### Permissions
Connecting user should have sufficient rights to query needed stats:

```
      {
         "role":"clusterMonitor",
         "db":"admin"
      },
      {
         "role":"read",
         "db":"local"
      }
```

More info about roles in MongoDB [documentation](https://docs.mongodb.com/manual/reference/built-in-roles/#mongodb-authrole-clusterMonitor).

#### Example
```
mongodb_exporter_linux_amd64/mongodb_exporter --mongodb.uri=mongodb://127.0.0.1:17001
```

#### Enabling collstats metrics gathering
`--mongodb.collstats-colls` receives a list of databases and collections to monitor using collstats.
Usage example: `--mongodb.collstats-colls=database1.collection1,database2.collection2`
```
mongodb_exporter_linux_amd64/mongodb_exporter --mongodb.uri=mongodb://127.0.0.1:17001 --mongodb.collstats-colls=db1.c1,db2.c2
```
#### Enabling compatibility mode.
When compatibility mode is enabled by the `--compatible-mode`, the exporter will expose all new metrics with the new naming and labeling schema and at the same time will expose metrics in the version 1 compatible way.
For example, if compatibility mode is enabled, the metric `mongodb_ss_wt_log_log_bytes_written` (new format)
```
# HELP mongodb_ss_wt_log_log_bytes_written serverStatus.wiredTiger.log.
# TYPE mongodb_ss_wt_log_log_bytes_written untyped
mongodb_ss_wt_log_log_bytes_written 2.6208e+06
```
will be also exposed as `mongodb_mongod_wiredtiger_log_bytes_total`  with the `unwritten` label.
```
HELP mongodb_mongod_wiredtiger_log_bytes_total mongodb_mongod_wiredtiger_log_bytes_total
# TYPE mongodb_mongod_wiredtiger_log_bytes_total untyped
mongodb_mongod_wiredtiger_log_bytes_total{type="unwritten"} 2.6208e+06
```

#### Cluster role labels
The exporter sets some topology labels in all metrics.
The labels are:

- cl_role: Cluster role according to this table:

|Server type|Label|
|-----|-----|
|mongos|mongos|
|regular instance (primary or secondary)|shardsvr|
|arbiter|shardsvr|
|standalone|(empty string)|

- cl_id: Cluster ID
- rs_nm: Replicaset name
- rs_state: Replicaset state is an integer from `getDiagnosticData()` -> `replSetGetStatus.myState`. 
Check [the official documentation](https://docs.mongodb.com/manual/reference/replica-states/) for details on replicaset status values.

## Usage Reference

See the [Reference Guide](REFERENCE.md) for details on using the exporter.
## Bug Reports / Feature PR

Refer to the [Contribution Guide](CONTRIBUTING.md).
