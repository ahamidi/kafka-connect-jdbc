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
* `TRUSTSTORE_PASSWORD` (**Required**)
* `KEYSTORE_PASSWORD` (**Required**)
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
API). As the REST API is not publicly accessible, you must use the provided script:

```
# Create the connector
heroku run add-jdbc-connector <name>
```


You can delete the Connector using the provided "delete" scripts:

```
# Delete a connector
heroku run delete-jdbc-connector <name>
```
