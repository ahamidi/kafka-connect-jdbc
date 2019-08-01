# Kafka Connect + JDBC Connector + Heroku Data

## Getting Started

### Prerequisites

**Addons**
* Heroku Postgres Addon
* Heroku Kafka Addon

**Kafka Topics**
For each Postgres Table that is being sync'd, you will need to create an associated
Kafka topic using the naming scheme `pg-<table_name>`.

You will also need to create the following topics for use by connect itself:
* connect-offsets - 25 or 50 partitions, replication 3, compacted
* connect-configs - 1 partition, replication 3, compacted
* connect-status - 1 partition, replication 3, compacted

**Kafka Consumer Group**
You will also need to create the Consumer Group `connect-cluster`

### Config Vars

* `KAFKA_ADDON` (Default: `KAFKA`)
* `POSTGRES_ADDON` (Default: `DATABASE`)
* `TABLE_WHITELIST` (**Required**) - A comma separated list of Postgres tables to sync.
* `INCREMENTING_COLUMN` (Default: `id`) - The column to be used as the unique, incrementing key
* `CONNECT_PASSWORD` (**Required**) - The HTTP Basic Auth password used for the REST API

### Buildpacks

* Apt: `https://github.com/amiel/heroku-buildpack-apt#feature/support-adding-keys`
* JVM: `heroku/jvm`

_Note:_ Since we're using the confluent repo, we have to use the apt buildpack fork
that adds support for importing keys.

## Usage

Once you have deployed this app, you will need to scale up the dyno. A `Standard-2X` 
at a minimum is required, but a Performance-M or larger dyno is recommended.

```
# Scale the dyno
heroku ps:scale worker=1:Standard-2X
```

You will then need to create the Connector (i.e. POST to the Kafka Connect REST 
API).

```
# Create the connector
curl -u $CONNECT_USERNAME -X POST https://$APP_URL/connectors -H "Content-Type: application/json" --data @- << EOF
{
  "name": "$CONNECTOR_NAME",
  "config": {
    "connector.class": "io.confluent.connect.jdbc.JdbcSourceConnector",
    "tasks.max": "1",
    "connection.url":"$JDBC_URL",
    "connection.attempts":"3",
    "connection.backoff.ms":"10000",
    "table.whitelist":"$TABLE_WHITELIST",
    "mode":"incrementing",
    "incrementing.column.name":"$INCREMENTING_COLUMN",
    "poll.interval.ms":"5000",
    "batch.max.rows":"100",
    "table.poll.interval.ms":"60000",
    "topic.prefix":"pg-",
    "table.types":"TABLE",
    "timestamp.delay.interval.ms":"0"
  }
}
```


You can delete the Connector using the following:

```
# Delete a connector
curl -u $CONNECT_USERNAME -X DELETE https://$APP_URL/connectors/$CONNECTOR_NAME
```
