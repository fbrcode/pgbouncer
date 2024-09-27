# PgBouncer Docker Compose Sample Code

Create a sample code with bitnami/pgbouncer connection pooling with docker compose.

## Prerequisites

- Docker
- Docker Compose
- PostgreSQL client

## Authentication Options

Source: <https://www.pgbouncer.org/config.html#authentication-file-format>

For pgBouncer, there are 3 authentication options:

### Plain

Fill the results on `conf/pgbouncer/userlist.txt`

```txt
"pgbouncer" "pgbouncer"
```

### MD5

Get user and password md5 hash with the following SQL query.

```sql
-- connected to any postgres instance, run
WITH credentials AS (SELECT 'pgbouncer' AS usr, 'pgbouncer' as pwd)
SELECT pwd || usr AS str, 'md5' || md5(pwd || usr) AS pgbouncer_md5_hash
FROM credentials;

        str         |         pgbouncer_md5_hash
--------------------+-------------------------------------
 pgbouncerpgbouncer | md5be5544d3807b54dd0637f2439ecb03b9
```

Fill the results on `conf/pgbouncer/userlist.txt`

```txt
"pgbouncer" "md5be5544d3807b54dd0637f2439ecb03b9"
```

### SCRAM (Salted Challenge Response Authentication Mechanism)

Get user and password SCRAM hash with the following SQL query.

```sql
-- check if the password_encryption is set to scram-sha-256
show password_encryption;

 password_encryption
---------------------
 scram-sha-256

-- If it is not set to scram-sha-256, you can change it in your postgresql.conf or for your session:
SET password_encryption = 'scram-sha-256';

-- get the SCRAM hash
SELECT rolname, rolpassword FROM pg_authid WHERE rolname = 'pgbouncer';

  rolname  |                                                              rolpassword
-----------+---------------------------------------------------------------------------------------------------------------------------------------
 pgbouncer | SCRAM-SHA-256$4096:TQ2NPg90uXsGc2t22rdJIw==$z2Y1lmPi9zqD5yP2eg8pd3R5rS0fjEQpOSfxYal6M/o=:2J3V5qcWhh/RMDPsD44FY70IzO+YiNNERpvDeoHuUz4=
```

Fill the results on `conf/pgbouncer/userlist.txt`

```txt
"pgbouncer" "SCRAM-SHA-256$4096:TQ2NPg90uXsGc2t22rdJIw==$z2Y1lmPi9zqD5yP2eg8pd3R5rS0fjEQpOSfxYal6M/o=:2J3V5qcWhh/RMDPsD44FY70IzO+YiNNERpvDeoHuUz4="
```

> Attention: On this docker compose implementation, first start postgres service, get the SCRAM hash, update `userlist.txt`, and then start pgbouncer service.

## How to run it

This will get postgres image and start the container service.

```sh
docker compose up -d postgres
```

This will get pgbouncer image and start the container service.

```sh
docker compose up -d pgbouncer
```

Restart the pgbouncer service.

```sh
docker compose restart pgbouncer
```

Drop services with their volumes.

```sh
docker compose down --volumes --remove-orphans pgbouncer
docker compose down --volumes --remove-orphans postgres
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

Need to add the appropriate grants in `pgBouncer.ini` file (i.e. `admin_users=pgbouncer`)

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
