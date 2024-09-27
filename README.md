# PgBouncer Docker Compose Sample Code

Create a sample code with bitnami/pgbouncer connection pooling with docker compose.

## Prerequisites

- Docker
- Docker Compose
- PostgreSQL client

## Authentication Options

Source: <https://www.pgbouncer.org/config.html#authentication-file-format>

For pgBouncer, there are 3 authentication options: Plain, MD5, and SCRAM.

The authentication method in the target PostgreSQL server must match the method used in the `userlist.txt` file. The `password_encryption` parameter in the PostgreSQL server configuration file `conf/postgres/postgresql.conf` determines the authentication method.

Current value can be verified by running the following SQL query:

```sql
show password_encryption;
```

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
 pgbouncer | SCRAM-SHA-256$4096:/4mqVV2T3u2r3QrJqtSsBw==$G/2PqGPXOnxwEiTN4wkm64TXwQ7jHXV3snMxROT5JhY=:/s2LKrmf+Pnt+x0PajxdYSljDz2OouWUHS1fZBGXSLA=
```

Fill the results on `conf/pgbouncer/userlist.txt`

```txt
"pgbouncer" "SCRAM-SHA-256$4096:TQ2NPg90uXsGc2t22rdJIw==$z2Y1lmPi9zqD5yP2eg8pd3R5rS0fjEQpOSfxYal6M/o=:2J3V5qcWhh/RMDPsD44FY70IzO+YiNNERpvDeoHuUz4="
```

> Attention: On this docker compose implementation, first start postgres service, get the SCRAM hash for the desired user (in this case `pgbouncer`), update `conf/pgbouncer/userlist.txt`, and then start pgbouncer service.

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

## Show postgres instance settings

```sh
source .env
psql
```

Locate PostgreSQL configuration files.

```txt
show hba_file;

    hba_file
-----------------
/etc/pg_hba.conf

show config_file;

    config_file
---------------------
/etc/postgresql.conf
```

Display hba rules.

```txt
table pg_hba_file_rules;

rule_number |    file_name     | line_number | type  | database |  user_name  |  address  |                 netmask                 |  auth_method  | options | error
------------+------------------+-------------+-------+----------+-------------+-----------+-----------------------------------------+---------------+---------+-------
          1 | /etc/pg_hba.conf |         117 | local | {all}    | {all}       |           |                                         | trust         |         |
          2 | /etc/pg_hba.conf |         119 | host  | {all}    | {all}       | 127.0.0.1 | 255.255.255.255                         | trust         |         |
          3 | /etc/pg_hba.conf |         121 | host  | {all}    | {all}       | ::1       | ffff:ffff:ffff:ffff:ffff:ffff:ffff:ffff | trust         |         |
          4 | /etc/pg_hba.conf |         132 | host  | {all}    | {postgres}  | all       |                                         | scram-sha-256 |         |
          5 | /etc/pg_hba.conf |         133 | host  | {all}    | {pgbouncer} | all       |                                         | scram-sha-256 |         |
```

Display the encryption method.

```txt
show password_encryption;

password_encryption
--------------------
scram-sha-256
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
show help;
show config;
show databases;
show pools;
show clients;
show servers;
show users;
show version;

show fds;
show sockets;
show active_sockets;
show lists;
show mem;
show state;

show dns_hosts;
show dns_zones;

SHOW STATS|

show stats;
show stats_totals;
show stats_averages;
show totals;
```
