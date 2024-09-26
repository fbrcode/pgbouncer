# PgBouncer Docker Compose Sample Code

Create a sample code with bitnami/pgbouncer connection pooling with docker compose.

## Prerequisites

- Docker
- Docker Compose
- PostgreSQL client

## How to run it

This will get postgres and pgbouncer images and start the container services.

```sh
docker compose up -d
```

Restart the container services.

```sh
docker compose down -v && docker compose up -d
```

Output pgbouncer logs.

```sh
docker compose logs -f pgbouncer
```

## Connect to the postgres instance directly

```sh
source .env
psql
# or
psql -d sample

# check server activity
\pset pager off
SELECT pid, usename, datname, client_addr, state, query FROM pg_stat_activity;
```

## Connect through pgbouncer

This step will create 50 connections to the database and make them ready to use.

```sh
source .env.pgbouncer
psql
# or
psql -d sample
```

## Connect to the admin panel of pgbouncer

```sh
source .env.pgb.admin
psql
# or
psql -d pgbouncer

# issue admin show commands
show pools; # to see the pool status
show stats; # to see the statistics
show clients; # to see the client connections
show servers; # to see the server connections
show lists; # to see the list of databases
show config; # to see the configuration
show version; # to see the version
show help; # to see the help
show dns_hosts; # to see the DNS hosts
show dns_zones; # to see the DNS zones
show dns_records; # to see the DNS records
show dns_query; # to see the DNS query
show dns_all; # to see the DNS all
show dns_failover; # to see the DNS failover
show dns_srv; # to see the DNS SRV
show dns_srv_all; # to see the DNS SRV all
show dns_srv_failover; # to see the DNS SRV failover
show dns_srv_order; # to see the DNS SRV order
show dns_srv_order_all; # to see the DNS SRV order all
show dns_srv_order_failover; # to see the DNS SRV order failover
```
