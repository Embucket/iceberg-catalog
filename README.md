# TIP Catalog for Apache Iceberg

[![License](https://img.shields.io/badge/License-Apache_2.0-blue.svg)](https://opensource.org/licenses/Apache-2.0)
[![Unittests](https://github.com/hansetag/iceberg-catalog/actions/workflows/unittests.yml/badge.svg)](https://github.com/hansetag/iceberg-catalog/actions/workflows/unittests.yml)
[![Spark Integration](https://github.com/hansetag/iceberg-catalog/actions/workflows/spark-integration.yml/badge.svg)](https://github.com/hansetag/iceberg-catalog/actions/workflows/spark-integration.yml)
[![Pyiceberg Integration](https://github.com/hansetag/iceberg-catalog/actions/workflows/pyiceberg-integration.yml/badge.svg)](https://github.com/hansetag/iceberg-catalog/actions/workflows/pyiceberg-integration.yml)
[![Trino Integration](https://github.com/hansetag/iceberg-catalog/actions/workflows/trino-integration.yml/badge.svg)](https://github.com/hansetag/iceberg-catalog/actions/workflows/trino-integration.yml)
[![Starrocks Integration](https://github.com/hansetag/iceberg-catalog/actions/workflows/starrocks-integration.yml/badge.svg)](https://github.com/hansetag/iceberg-catalog/actions/workflows/starrocks-integration.yml)
[![Artifact Hub](https://img.shields.io/endpoint?url=https://artifacthub.io/badge/repository/tip-catalog)](https://artifacthub.io/packages/helm/tip-catalog/tip-catalog)
[![Docker on quay](https://img.shields.io/badge/docker-%230db7ed.svg?style=for-the-badge&logo=docker&logoColor=white)](https://quay.io/repository/hansetag/tip-catalog?tab=tags&filter_tag_name=like%3Av)
[![Helm Chart](https://img.shields.io/badge/Helm-0F1689?style=for-the-badge&logo=Helm&labelColor=0F1689)](https://github.com/hansetag/tip-catalog-charts/tree/main/charts/tip-catalog)
[![Discord](https://img.shields.io/badge/Discord-%235865F2.svg?style=for-the-badge&logo=discord&logoColor=white)](https://discord.gg/jkAGG8p93B)

This is TIP: A Rust-native implementation of the [Apache Iceberg](https://iceberg.apache.org/) REST Catalog specification based on [apache/iceberg-rust](https://github.com/apache/iceberg-rust).

If you have questions, feature requests or just want a chat, we are hanging around in [Discord](https://discord.gg/jkAGG8p93B)!

# Scope and Features

**T**he **I**ceberg **P**rotocol (**TIP**) based on REST has become the standard for catalogs in open Lakehouses. It natively enables multi-table commits, server-side deconflicting and much more. It is figuratively the (**TIP**) of the Iceberg.

We have started this implementation because we were missing customizability, support for on-premise deployments and other features that are important for us in existing Iceberg Catalogs. Please find following some of our focuses with this implementation:

- **Customizable**: Our implementation is meant to be extended. We expose the Database implementation, Secrets, Authorization, EventPublishing and ContractValidation as interfaces (Traits). This allows you to tap into any Access management system of your company or stream change events to any system you like - simply by implementing a handful methods. Please find more details in the [Customization Guide](CUSTOMIZING.md).
- **Change Events**: Built-in support to emit change events (CloudEvents), which enables you to react to any change that happen to your tables.
- **Change Approval**: Changes can also be prohibited by external systems. This can be used to prohibit changes to tables that would invalidate Data Contracts, Quality SLOs etc. Simply integrate with your own change approval via our `ContractVerification` trait.
- **Multi-Tenant capable**: A single deployment of our catalog can serve multiple projects - all with a single entrypoint. All Iceberg and Warehouse configurations are completely separated between Warehouses.
- **Written in Rust**: Single 30Mb all-in-one binary - no JVM or Python env required.
- **Storage Access Management**: Built-in S3-Signing that enables support for self-hosted as well as AWS S3 WITHOUT sharing S3 credentials with clients. We are also working on `vended-credentials`!
- **Well-Tested**: Integration-tested with `spark` and `pyiceberg` (support for S3 with this catalog from pyiceberg 0.7.0)
- **High Available & Horizontally Scalable**: There is no local state - the catalog can be scaled horizontally and updated without downtimes.
- **Openid provider integration**: Use your own identity provider to secure access to the APIs, just set `ICEBERG_REST__OPENID_PROVIDER_URI` and you are good to go.
- **Fine Grained Access (FGA) (Coming soon):** Simple Role-Based access control is not enough for many rapidly evolving Data & Analytics initiatives. We are leveraging [OpenFGA](https://openfga.dev/) based on googles [Zanzibar-Paper](https://research.google/pubs/zanzibar-googles-consistent-global-authorization-system/) to implement authorization. If your company already has a different system in place, you can integrate with it by implementing a handful of methods in the `AuthZHandler` trait.

Please find following an overview of currently supported features. Please also check the Issues if you are missing something.

# Quickstart

A Docker Container is available on [quay.io](https://quay.io/repository/hansetag/iceberg-catalog?tab=info).
We have prepared a self-contained docker-compose file to demonstrate the usage of `spark` with our catalog:

```sh
git clone https://github.com/hansetag/iceberg-catalog.git
cd iceberg-catalog/examples
docker compose up
```

Then open your browser and head to `localhost:8888`.

For more information on deployment, please check the [User Guide](USER_GUIDE.md).

# Status

### Supported Operations - Iceberg-Rest

| Operation | Status  | Description                                                              |
| --------- | :-----: | ------------------------------------------------------------------------ |
| Namespace | ![done] | All operations implemented                                               |
| Table     | ![done] | All operations implemented - additional integration tests in development |
| Views     | ![done] | Remove unused files and log entries                                      |
| Metrics   | ![open] | Endpoint is available but doesn't store the metrics                      |

### Storage Profile Support

| Storage              |    Status    | Comment                                                   |
| -------------------- | :----------: | --------------------------------------------------------- |
| S3 - AWS             | ![semi-done] | vended-credentials & remote-signing, assume role missing  |
| S3 - Custom          |   ![done]    | vended-credentials & remote-signing, tested against minio |
| Azure ADLS Gen2      |   ![done]    |                                                           |
| Azure Blob           |   ![open]    |                                                           |
| Microsoft OneLake    |   ![open]    |                                                           |
| Google Cloud Storage |   ![open]    |                                                           |

Details on how to configure the storage profiles can be found in the [Storage Guide](STORAGE.md).

### Supported Catalog Backends

| Backend  | Status  | Comment |
| -------- | :-----: | ------- |
| Postgres | ![done] |         |
| MongoDB  | ![open] |         |

### Supported Secret Stores

| Backend         | Status  | Comment       |
| --------------- | :-----: | ------------- |
| Postgres        | ![done] |               |
| kv2 (hcp-vault) | ![done] | userpass auth |

### Supported Event Stores

| Backend | Status  | Comment |
| ------- | :-----: | ------- |
| Nats    | ![done] |         |
| Kafka   | ![open] |         |

### Supported Operations - Management API

| Operation            | Status  | Description                                        |
| -------------------- | :-----: | -------------------------------------------------- |
| Warehouse Management | ![done] | Create / Update / Delete a Warehouse               |
| AuthZ                | ![open] | Manage access to warehouses, namespaces and tables |
| More to come!        | ![open] |                                                    |

### Auth(N/Z) Handlers

| Operation       | Status  | Description                                                                                                        |
| --------------- | :-----: | ------------------------------------------------------------------------------------------------------------------ |
| OIDC (AuthN)    | ![done] | Secure access to the catalog via OIDC                                                                              |
| Custom (AuthZ)  | ![done] | If you are willing to implement a single rust Trait, the `AuthZHandler` can be implement to connect to your system |
| OpenFGA (AuthZ) | ![open] | Internal Authorization management                                                                                  |

# Multiple Projects

The iceberg-rest server can host multiple independent warehouses that are again grouped by projects. The overall
structure looks like this:

```
<project-1-uuid>/
├─ foo-warehouse
├─ bar-warehouse
<project-2-uuid>/
├─ foo-warehouse
├─ bas-warehouse

```

All warehouses use isolated namespaces and can be configured in client by specifying `warehouse`
as `'<project-uuid>/<warehouse-name>'`. Warehouse Names inside Projects must be unique. We recommend using human
readable names for warehouses.

If you do not need the hierarchy level of projects, set the `ICEBERG_REST__DEFAULT_PROJECT_ID` environment variable to
the project you want to use. For single project deployments we recommend using the NULL UUID ("
00000000-0000-0000-0000-000000000000") as project-id. Users then just specify `warehouse` as `<warehouse-name>` when
connecting.

# Undrop Tables

When a table or view is dropped, it is not immediately deleted from the catalog. Instead, it is marked as dropped and a job for its cleanup is scheduled. The table, including its data if `purgeRequested=True`, is then deleted after the configured `ICEBERG_TABULAR_EXPIRATION_DELAY_SECONDS` (default: 7 days) have passed. This will allow for a recovery of tables that have been dropped by accident.


# Configuration

The basic setup of the Catalog is configured via environment variables. As this catalog supports a multi-tenant setup, each catalog ("warehouse") also comes with its own configuration options including its Storage Configuration. The documentation of the Management-API for warehouses is hosted at the unprotected `/swagger-ui` endpoint.

Following options are global and apply to all warehouses:

### General

| Variable                            | Example                                | Description                                                                                                                                                                                                                    |
| ----------------------------------- | -------------------------------------- |--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| `ICEBERG_REST__BASE_URI`            | `https://example.com:8080`            | Base URL where the catalog is externally reachable. Default: `https://localhost:8080`                                                                                                                                          |
| `ICEBERG_REST__DEFAULT_PROJECT_ID`  | `00000000-0000-0000-0000-000000000000` | The default project ID to use if the user does not specify a project when connecting. We recommend setting the Project-ID only in single Project setups. Each Project can still contain multiple Warehouses. Default: Not set. |
| `ICEBERG_REST__RESERVED_NAMESPACES` | `system,examples`                      | Reserved Namespaces that cannot be created via the REST interface                                                                                                                                                              |
| `ICEBERG_REST__METRICS_PORT`        | `9000`                                 | Port where the metrics endpoint is reachable. Default: `9000`                                                                                                                                                                  |
| `ICEBERG_REST__LISTEN_PORT`         | `8080`                                 | Port the server listens on. Default: `8080`                                                                                                                                                                                    |
| `ICEBERG_REST__SECRET_BACKEND`      | `postgres`                             | The secret backend to use. If `kv2` is chosen, you need to provide additional parameters found under []() Default: `postgres`, one-of: [`postgres`, `kv2`]                                                                     |
| `ICEBERG_DEFAULT_TABULAR_EXPIRATION_DELAY_SECONDS` | `86400`                          | Time after which a tabular, i.e. View or Table which has been dropped is going to be deleted. Default: `86400` \[7 days\]                                                                                                      |

### Task queues

Currently, the catalog uses two task queues, one to ultimately delete soft-deleted tabulars and another to purge tabulars which have been deleted with the `purgeRequested=True` query parameter. The task queues are configured as follows:

| Variable                                    | Example | Description                                                                                                 |
|---------------------------------------------|---------|-------------------------------------------------------------------------------------------------------------|
| `ICEBERG_REST__QUEUE_CONFIG__MAX_RETRIES`   | 5       | Number of retries before a task is considered failed  Default: 5                                            |
| `ICEBERG_REST__QUEUE_CONFIG__MAX_AGE`       | 3600    | Amount of seconds before a task is considered stale and could be picked up by another worker. Default: 3600 |
| `ICEBERG_REST__QUEUE_CONFIG__POLL_INTERVAL` | 10      | Amount of seconds between polling for new tasks. Default: 10                                                |

The queues are currently implemented using the `sqlx` Postgres backend. If you want to use a different backend, you need to implement the `TaskQueue` trait.

### Postgres

Configuration parameters if Postgres is used as a backend, you may either provide connection strings or use the `PG_*` environment variables, connection strings take precedence:

| Variable                                    | Example                                               | Description                                                                                                                                |
| ------------------------------------------- | ----------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------ |
| `ICEBERG_REST__PG_DATABASE_URL_READ`        | `postgres://postgres:password@localhost:5432/iceberg` | Postgres Database connection string used for reading                                                                                       |
| `ICEBERG_REST__PG_DATABASE_URL_WRITE`       | `postgres://postgres:password@localhost:5432/iceberg` | Postgres Database connection string used for writing.                                                                                      |
| `ICEBERG_REST__PG_ENCRYPTION_KEY`           | `<This is unsafe, please set a proper key>`           | If `ICEBERG_REST__SECRET_BACKEND=postgres`, this key is used to encrypt secrets. It is required to change this for production deployments. |
| `ICEBERG_REST__PG_READ_POOL_CONNECTIONS`    | `10`                                                  | Number of connections in the read pool                                                                                                     |
| `ICEBERG_REST__PG_WRITE_POOL_CONNECTIONS`   | `5`                                                   | Number of connections in the write pool                                                                                                    |
| `ICEBERG_REST__PG_HOST_R`                   | `localhost`                                           | Hostname for read operations                                                                                                               |
| `ICEBERG_REST__PG_HOST_W`                   | `localhost`                                           | Hostname for write operations                                                                                                              |
| `ICEBERG_REST__PG_PORT`                     | `5432`                                                | Port number                                                                                                                                |
| `ICEBERG_REST__PG_USER`                     | `postgres`                                            | Username for authentication                                                                                                                |
| `ICEBERG_REST__PG_PASSWORD`                 | `password`                                            | Password for authentication                                                                                                                |
| `ICEBERG_REST__PG_DATABASE`                 | `iceberg`                                             | Database name                                                                                                                              |
| `ICEBERG_REST__PG_SSL_MODE`                 | `require`                                             | SSL mode (disable, allow, prefer, require)                                                                                                 |
| `ICEBERG_REST__PG_SSL_ROOT_CERT`            | `/path/to/root/cert`                                  | Path to SSL root certificate                                                                                                               |
| `ICEBERG_REST__PG_ENABLE_STATEMENT_LOGGING` | `true`                                                | Enable SQL statement logging                                                                                                               |
| `ICEBERG_REST__PG_TEST_BEFORE_ACQUIRE`      | `true`                                                | Test connections before acquiring from the pool                                                                                            |
| `ICEBERG_REST__PG_CONNECTION_MAX_LIFETIME`  | `1800`                                                | Maximum lifetime of connections in seconds                                                                                                 |

### KV2 (HCP Vault)

Configuration parameters if a KV2 compatible storage is used as a backend. Currently, we only support the `userpass` authentication method. You may provide the envs as single values like `ICEBERG_REST__KV2__URL=http://vault.local` etc. or as a compound value like:
`ICEBERG_REST__KV2='{url="http://localhost:1234", user="test", password="test", secret_mount="secret"}'`

| Variable                          | Example               | Description                                      |
| --------------------------------- | --------------------- | ------------------------------------------------ |
| `ICEBERG_REST__KV2__URL`          | `https://vault.local` | URL of the KV2 backend                           |
| `ICEBERG_REST__KV2__USER`         | `admin`               | Username to authenticate against the KV2 backend |
| `ICEBERG_REST__KV2__PASSWORD`     | `password`            | Password to authenticate against the KV2 backend |
| `ICEBERG_REST__KV2__SECRET_MOUNT` | `kv/data/iceberg`     | Path to the secret mount in the KV2 backend      |

### Nats

If you want the server to publish events to a NATS server, set the following environment variables:

| Variable                        | Example                 | Description                                                            |
| ------------------------------- | ----------------------- | ---------------------------------------------------------------------- |
| `ICEBERG_REST__NATS_ADDRESS`    | `nats://localhost:4222` | The URL of the NATS server to connect to                               |
| `ICEBERG_REST__NATS_TOPIC`      | `iceberg`               | The subject to publish events to                                       |
| `ICEBERG_REST__NATS_USER`       | `test-user`             | User to authenticate against nats, needs `ICEBERG_REST__NATS_PASSWORD` |
| `ICEBERG_REST__NATS_PASSWORD`   | `test-password`         | Password to authenticate against nats, needs `ICEBERG_REST__NATS_USER` |
| `ICEBERG_REST__NATS_CREDS_FILE` | `/path/to/file.creds`   | Path to a file containing nats credentials                             |
| `ICEBERG_REST__NATS_TOKEN`      | `xyz`                   | Nats token to authenticate against server                              |

### OpenID Connect

If you want to limit ac
cess to the API, set `ICEBERG_REST__OPENID_PROVIDER_URI` to the URI of your OpenID Connect Provider. The catalog will then verify access tokens against this provider. The provider must have the `.well-known/openid-configuration` endpoint under `${ICEBERG_REST__OPENID_PROVIDER_URI}/.well-known/openid-configuration` and the openid-configuration needs to have the `jwks_uri` and `issuer` defined.

If `ICEBERG_REST__OPENID_PROVIDER_URI` is set, every request needs have an authorization header, e.g.

```sh
curl {your-catalog-url}/catalog/v1/transactions/commit -X POST -H "authorization: Bearer {your-token-here}" -H "content-type: application/json" -d ...
```

| Variable                            | Example                                      | Description                                                                                                                                                                                                                                                  |
| ----------------------------------- | -------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| `ICEBERG_REST__OPENID_PROVIDER_URI` | `https://keycloak.local/realms/{your-realm}` | OpenID Provider URL, with keycloak this is the url pointing to your realm, for Azure App Registration it would be something like `https://login.microsoftonline.com/{your-tenant-id-here}/v2.0/`. If this variable is not set, endpoints are **not** secured |

## License

Licensed under the [Apache License, Version 2.0](http://www.apache.org/licenses/LICENSE-2.0)

[open]: https://cdn.jsdelivr.net/gh/Readme-Workflows/Readme-Icons@main/icons/octicons/IssueNeutral.svg
[semi-done]: https://cdn.jsdelivr.net/gh/Readme-Workflows/Readme-Icons@main/icons/octicons/ApprovedChangesGrey.svg
[done]: https://cdn.jsdelivr.net/gh/Readme-Workflows/Readme-Icons@main/icons/octicons/ApprovedChanges.svg
