---
title: Overview
---

Rke2 datastore options:

* **Embedded [Etcd](https://etcd.io/)**  
  Embedded Etcd is the default datastore, and will be used if no other datastore configuration is present.
* **Embedded [SQLite](https://www.sqlite.org/index.html)**  
  SQLite cannot be used on clusters with multiple servers.
* **External Database**  
  The following external datastores:
  * [etcd](https://etcd.io/) (certified against version 3.5.4)
  * [MySQL](https://www.mysql.com) (certified against versions 5.7 and 8.0)
  * [MariaDB](https://mariadb.org/) (certified against version 10.6.8)
  * [PostgreSQL](https://www.postgresql.org/) (certified against versions 12.16, 13.12, 14.9 and 15.4)

:::warning Experimental
Rke2 only oficially supports Embedded etcd, the other datastores are experimental.
:::

:::warning Prepared Statement Support
Rke2 requires prepared statements support from the DB. This means that connection poolers such as [PgBouncer](https://www.pgbouncer.org/faq.html#how-to-use-prepared-statements-with-transaction-pooling) may require additional configuration to work with Rke2.
:::

### SQLite Configuration Parameters
If you wish to use SQLite as the database for your Rke2 server, you must set `--disable-etcd` parameter without a `--server` parameter so that Rke2 knows that it needs to disable etcd and run with SQLite.

### External Datastore Configuration Parameters
If you wish to use an external datastore such as PostgreSQL, MySQL, or etcd you must set the `datastore-endpoint` parameter so that Rke2 knows how to connect to it. You may also specify parameters to configure the authentication and encryption of the connection. The below table summarizes these parameters, which can be passed as either CLI flags or environment variables.

| CLI Flag | Environment Variable | Description
|------------|-------------|------------------
| `--datastore-endpoint` | `RKE2_DATASTORE_ENDPOINT` | Specify a PostgreSQL, MySQL, or etcd connection string. This is a string used to describe the connection to the datastore. The structure of this string is specific to each backend and is detailed below. |
| `--datastore-cafile` | `RKE2_DATASTORE_CAFILE` | TLS Certificate Authority (CA) file used to help secure communication with the datastore. If your datastore serves requests over TLS using a certificate signed by a custom certificate authority, you can specify that CA using this parameter so that the K3s client can properly verify the certificate. |
| `--datastore-certfile` | `RKE2_DATASTORE_CERTFILE` | TLS certificate file used for client certificate based authentication to your datastore. To use this feature, your datastore must be configured to support client certificate based authentication. If you specify this parameter, you must also specify the `datastore-keyfile` parameter. |
| `--datastore-keyfile` | `RKE2_DATASTORE_KEYFILE` | TLS key file used for client certificate based authentication to your datastore. See the previous `datastore-certfile` parameter for more details. |

As a best practice we recommend setting these parameters as environment variables rather than command line arguments so that your database credentials or other sensitive information aren't exposed as part of the process info.

### Datastore Endpoint Format and Functionality
As mentioned, the format of the value passed to the `datastore-endpoint` parameter is dependent upon the datastore backend. The following details this format and functionality for each supported external datastore.

<Tabs queryString="ext-db">
<TabItem value="PostgreSQL">


  In its most common form, the datastore-endpoint parameter for PostgreSQL has the following format:

  `postgres://username:password@hostname:port/database-name`

  More advanced configuration parameters are available. For more information on these, please see https://godoc.org/github.com/lib/pq.

  If you specify a database name and it does not exist, the server will attempt to create it.

  If you only supply `postgres://`  as the endpoint, Rke2 will attempt to do the following:

  - Connect to localhost using `postgres` as the username and password
  - Create a database named `kubernetes`

</TabItem>
<TabItem value="MySQL / MariaDB">

  In its most common form, the `datastore-endpoint` parameter for MySQL and MariaDB has the following format:

  `mysql://username:password@tcp(hostname:3306)/database-name`

  More advanced configuration parameters are available. For more information on these, please see https://github.com/go-sql-driver/mysql#dsn-data-source-name

  Note that due to a [known issue](https://github.com/k3s-io/k3s/issues/1093) in Rke2, you cannot set the `tls` parameter. TLS communication is supported, but you cannot, for example, set this parameter to "skip-verify" to cause Rke2 to skip certificate verification.

  If you specify a database name and it does not exist, the server will attempt to create it.

  If you only supply `mysql://` as the endpoint, Rke2 will attempt to do the following:

  - Connect to the MySQL socket at `/var/run/mysqld/mysqld.sock` using the `root` user and no password
  - Create a database with the name `kubernetes`

</TabItem>

<TabItem value="etcd">

  In its most common form, the `datastore-endpoint` parameter for etcd has the following format:

  `https://etcd-host-1:2379,https://etcd-host-2:2379,https://etcd-host-3:2379`

  The above assumes a typical three node etcd cluster. The parameter can accept one more comma separated etcd URLs.

</TabItem>
</Tabs>
