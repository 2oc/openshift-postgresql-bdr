# Openshift docker image for postgres 9.4 with BDR support

Based on Debian Jessie. Includes a patched postgres with support for [BDR](http://bdr-project.org/)

Image heavily borrowed from the [official image](https://github.com/docker-library/postgres) for postgres.
Look there for further usage instructions.

## Quick setup

```sh
oc new-app https://github.com/weepee-org/openshift-postgresql-bdr.git --name pgsql01
oc new-app https://github.com/weepee-org/openshift-postgresql-bdr.git --name pgsql02
```

Create a database on each node, eg named bdrdemo. Then, connect to the
database on the first node and run:

```SQL
CREATE EXTENSION IF NOT EXISTS btree_gist;
CREATE EXTENSION IF NOT EXISTS bdr;

SELECT bdr.bdr_group_create(
  local_node_name := 'psql01',
  node_external_dsn := 'host=psql01.host port=5432 dbname=bdrdemo'
);
```

Then, on all other nodes run:

```SQL
CREATE EXTENSION IF NOT EXISTS btree_gist;
CREATE EXTENSION IF NOT EXISTS bdr;

SELECT bdr.bdr_group_join(
  local_node_name := 'psqlXX',
  node_external_dsn := 'host=psqlXX port=5432 dbname=bdrdemo',
  join_using_dsn := 'host=psqlXX port=5432 dbname=bdrdemo'
);
```

(Make sure to replace node names and hosts with appropriate values)
