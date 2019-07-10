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
* connect-offsets
* connect-configs
* connect-status

**Kafka Consumer Group**
You will also need to create the Consumer Group `connect-cluster`

### Config Vars

* `KAFKA_ADDON` (Default: `KAFKA`)
* `POSTGRES_ADDON` (Default: `DATABASE`)
<<<<<<< HEAD
* `TRUSTSTORE_PASSWORD` (**Required**)
* `KEYSTORE_PASSWORD` (**Required**)
=======
>>>>>>> bc9f6afac0ded25bebfb4fa399ef38d6332412cd
* `TABLE_WHITELIST` (**Required**) - A comma separated list of Postgres tables to sync.
* `INCREMENTING_COLUMN` (Default: `id`) - The column to be used as the unique, incrementing key

### Buildpacks

* Apt: `https://github.com/heroku/heroku-buildpack-apt`
* JVM: `heroku/jvm`

## Usage

Once you have deployed this app, you will need to scale up the dyno. A `Standard-2X` 
at a minimum is required, but a Performance-M or larger dyno is recommended.

```
# Scale the dyno
heroku ps:scale worker=1:Standard-2X
```

You will then need to create the Connector (i.e. POST to the Kafka Connect REST 
<<<<<<< HEAD
API). As the REST API is not publicly accessible, you must use the provided script:

```
# Create the connector
heroku run add-jdbc-connector <name>
```


You can delete the Connector using the provided "delete" scripts:

```
# Delete a connector
heroku run delete-jdbc-connector <name>
=======
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
>>>>>>> bc9f6afac0ded25bebfb4fa399ef38d6332412cd
```
