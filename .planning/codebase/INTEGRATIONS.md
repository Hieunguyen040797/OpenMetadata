# OpenMetadata External Integrations

## Authentication & Identity Providers

### Supported Authentication Methods

| Provider | Type | Package | Configuration |
|----------|------|---------|--------------|
| **Basic Auth** | Username/Password | Built-in | Default for local dev |
| **Google OAuth** | OIDC | google-oauth-client | `AUTHENTICATION_AUTHORITY` |
| **Azure AD** | OIDC/OAuth2 | msal4j, azure-identity | `OIDC_TYPE=azure` |
| **Okta** | OIDC | okta-auth-js (UI), pac4j-oidc (backend) | `OIDC_*` vars |
| **Auth0** | OIDC | @auth0/auth0-react | `AUTHENTICATION_PUBLIC_KEYS` |
| **SAML 2.0** | SSO | java-saml, onelogin | `SAML_*` vars |
| **LDAP** | Directory | unboundid-ldapsdk | `AUTHENTICATION_LDAP_*` vars |
| **Custom OIDC** | OIDC | pac4j-oidc | `CUSTOM_OIDC_*` vars |

### JWT Configuration
- **JWT Issuer**: `JWT_ISSUER` (default: "open-metadata.org")
- **Key ID**: `JWT_KEY_ID`
- **RSA Keys**: Configured via file paths (`RSA_PUBLIC_KEY_FILE_PATH`, `RSA_PRIVATE_KEY_FILE_PATH`)
- **Token validation**: java-jwt 4.4.0, jwks-rsa for key rotation

### User Management
- **Registration**: Self-signup controlled by `AUTHORIZER_ALLOWED_REGISTRATION_DOMAIN`
- **Admin principals**: Configured via `AUTHORIZER_ADMIN_PRINCIPALS`
- **Domain enforcement**: Optional via `AUTHORIZER_ENFORCE_PRINCIPAL_DOMAIN`

## Database Integrations

### Primary Databases
| Database | Driver | Connection String | Notes |
|----------|--------|-------------------|-------|
| **MySQL** | mysql-connector-j 9.3.0 | jdbc:mysql://host:port/db | Default for local dev |
| **PostgreSQL** | postgresql 42.7.7 | jdbc:postgresql://host:port/db | Production alternative |

### Supported Data Sources (75+ connectors)
The Python ingestion framework connects to:

**Relational Databases**
- Snowflake
- Redshift (via JDBC and SQLAlchemy)
- BigQuery
- Databricks
- Oracle
- SQL Server (mssql, mssql-odbc)
- Db2, Db2-ibmi
- Apache Druid
- Apache Hive / Impala
- Presto / Trino
- Apache Calcite
- SingleStore
- CockroachDB
- SAP HANA
- Teradata
- Vertica
- Apache Pinot (pinotdb)
- Exasol
- Apache Doris / StarRocks

**NoSQL & Document Stores**
- MongoDB
- Apache Cassandra
- Couchbase
- Amazon DynamoDB
- Google BigTable

**Data Warehouses**
- Snowflake
- BigQuery
- Redshift
- Databricks
- Synapse / SQL DW
- Teradata

**Search & Indexing**
- Elasticsearch 7.x / 8.x
- OpenSearch

**Streaming**
- Apache Kafka (with Schema Registry)
- Amazon Kinesis
- Google Pub/Sub
- Redpanda

**Object Storage & Lake**
- S3 / MinIO (via boto3, s3fs)
- Azure Blob Storage (via adlfs)
- Google Cloud Storage (via gcsfs)
- Delta Lake (local, S3, ADLS, GCS)
- Apache Iceberg (via Spark/Delta)

**Business Intelligence**
- Tableau
- Looker
- Qlik Sense
- Power BI (via PowerBIgateway)
- Amazon QuickSight
- Redash
- Superset
- SSRS

**ML & Data Quality**
- MLflow
- Great Expectations (0.18.x and 1.x)
- dbt (via collate-dbt-artifacts-parser)
- Scikit-learn (profiling)

**Pipeline Orchestration**
- Apache Airflow (primary)
- Dagster

**Other Services**
- Amundsen (metadata to Neo4j)
- Apache Atlas
- SalesForce
- ServiceNow
- Elasticsearch

## Search Integrations

### Primary Search Engine
- **Elasticsearch 7.17+** - Default search backend
  - Configuration: `ELASTICSEARCH_HOST`, `ELASTICSEARCH_PORT`, `ELASTICSEARCH_SCHEME`
  - Authentication: Basic auth or TLS
  - Connection pooling: configurable timeouts

### Alternative Search Engine
- **OpenSearch 2.6+** - AWS-managed alternative
  - IAM authentication supported (AWS_* environment variables)
  - Same configuration interface as Elasticsearch

### Search Configuration
```bash
SEARCH_TYPE=elasticsearch  # or opensearch
ELASTICSEARCH_SCHEME=http   # or https
ELASTICSEARCH_USER=
ELASTICSEARCH_PASSWORD=
ELASTICSEARCH_TRUST_STORE_PATH=  # For TLS
```

## Cloud Provider Integrations

### AWS
**SDK**: AWS SDK v2 (software.amazon.awssdk:bom 2.30.19)

| Service | Usage | Dependencies |
|---------|-------|-------------|
| **S3** | Object storage, data lake | s3, aws-crt-client |
| **Secrets Manager** | Secrets storage | secretsmanager, ssm |
| **RDS** | Database IAM auth | rds, sts |
| **ElastiCache / Redis** | Caching | elasticache |
| **BedRock** | Vector embeddings (AI) | bedrockruntime |
| **CloudWatch** | Monitoring/logging | cloudwatch |
| **Glue** | Data catalog integration | glue |

**Authentication**: Supports IAM roles, access keys, instance profiles

### Azure
**SDK**: Azure Identity 1.14-1.15.x, MSAL

| Service | Usage | Dependencies |
|---------|-------|-------------|
| **Azure AD** | Authentication | msal4j, azure-identity |
| **Key Vault** | Secrets management | azure-security-keyvault-secrets |
| **Blob Storage** | Data lake | azure-storage-blob |
| **Entra ID** | SSO/OIDC | msal-react (frontend) |

### Google Cloud
**SDK**: google-cloud-*

| Service | Usage | Dependencies |
|---------|-------|-------------|
| **BigQuery** | Data warehouse | google-cloud-datacatalog, sqlalchemy-bigquery |
| **Cloud Storage** | Object storage | google-cloud-storage, gcsfs |
| **Secret Manager** | Secrets | google-cloud-secret-manager |
| **BigTable** | NoSQL | google-cloud-bigtable |
| **Pub/Sub** | Messaging | google-cloud-pubsub |
| **Logging** | Logging | google-cloud-logging |

### Kubernetes
**SDK**: kubernetes-client 24.0.0

- Secrets management via Kubernetes secrets
- Deployment and job orchestration
- Custom operator (openmetadata-k8s-operator)

## Secrets Management

| Provider | Backend | Package |
|----------|---------|---------|
| **Database** | Stored in MySQL/PostgreSQL | Default |
| **AWS Secrets Manager** | AWS SM | software.amazon.awssdk:secretsmanager |
| **AWS SSM** | AWS Systems Manager | software.amazon.awssdk:ssm |
| **Azure Key Vault** | Azure KV | azure-security-keyvault-secrets |
| **Google Secret Manager** | GCP SM | google-cloud-secret-manager |
| **Kubernetes Secrets** | K8s secrets | kubernetes-client |
| **HashiCorp Vault** | Not currently supported | - |

## Notification & Alerting

### Slack Integration
**SDK**: Slack Bolt 1.44.1 (slack-api-client)

- Webhook notifications for alerts
- Bot integration for commands
- Channel-based notifications

**Configuration**: Requires Slack app with bot token

### Email / SMTP
**Library**: Simple Java Mail 8.12.2

- Alert notifications
- User invitation emails
- SMTP server configuration via environment variables:
  - `SMTP_SERVER_ENDPOINT`
  - `SMTP_SERVER_PORT`
  - `SMTP_SERVER_USERNAME`
  - `SMTP_SERVER_STRATEGY` (SMTP_TLS, etc.)

### Microsoft Teams
Not a direct integration, but webhooks can be configured via custom HTTP sinks.

### Webhooks (Generic)
- Alert/notification webhooks
- Custom event subscriptions
- Outbound HTTP callbacks

## Workflow & Orchestration

### Apache Airflow (Primary)
- **Version**: Apache Airflow 3.1.7 (Python side)
- **Integration**: REST API client (AirflowRESTClient)
- **Features**:
  - DAG generation for ingestion workflows
  - Pipeline status monitoring
  - Secret passing via Fernet encryption
  - DAG execution tracking

**Configuration**:
```bash
AIRFLOW_USERNAME=admin
AIRFLOW_PASSWORD=admin
PIPELINE_SERVICE_CLIENT_ENDPOINT=http://ingestion:8080
FERNET_KEY=...  # Encryption key for Airflow credentials
```

### Dagster
- **Version**: dagster_graphql 1.8.0+
- **Integration**: GraphQL API

### Workflow Engine
- **Flowable 7.2.0** - Internal governance workflows
- **Quartz 2.5.0** - Scheduled jobs

## Observability & Monitoring

### Metrics
- **Prometheus** - Primary metrics collection
  - Endpoint: `/metrics` via Micrometer
  - Dropwizard metrics instrumentation
  - Prometheus registry at port 8586

### Health Checks
- **Liveness**: `/health/liveness` (port 8586)
- **Readiness**: `/health/readiness`
- **Info**: `/health/info`

### Logging
- **Logback** - Structured logging
- **Elasticsearch** - Log storage and search
- **Log format**: JSON or text (configurable via `LOG_FORMAT`)

### Tracing
- **OpenTelemetry** - Distributed tracing
  - Exporter: OTLP (opentelemetry-exporter-otlp 1.37.0)
  - Context propagation

### Event Monitoring
- **Prometheus** - Event aggregation
- **Configuration**: `EVENT_MONITOR_BATCH_SIZE`, `EVENT_MONITOR_PATH_PATTERN`

## Data Lineage

### Lineage Sources
- **SQL Parsing**: Apache Calcite, JSqlParser
- **Python**: sqllineage (collate-sqllineage 2.0.2)
- **Airflow**: OpenMetadata lineage backend plugin
- **Visualization**: AntV G6, React Flow

### Lineage Integrations
- **dbt**: Artifact parsing (collate-dbt-artifacts-parser)
- **Data Diff**: collate-data-diff for column-level lineage

## API Integrations

### REST API
- **Framework**: Dropwizard 5.0 with JAX-RS 3.x
- **Documentation**: OpenAPI 3.0 (Swagger)
- **Formats**: JSON, YAML
- **Authentication**: JWT Bearer tokens

### WebSocket
- **Library**: Socket.IO (server 4.x, client 2.x)
- **Usage**: Real-time updates, notifications
- **Ports**: 8585 (main), 8586 (admin)

### GraphQL
- **Dagster integration**: GraphQL API for pipeline queries

### Model Context Protocol (MCP)
- **SDK**: mcp-sdk 1.1.1 (io.modelcontextprotocol.sdk)
- **Usage**: AI/agent integrations

## Third-Party Tool Integrations

### Data Quality
- **Great Expectations** - Data validation (0.18.x and 1.x)
- **Collate Data Diff** - Column-level data comparison

### Data Catalog
- **Amundsen** - Metadata export to Neo4j
- **Apache Atlas** - Metadata indexing

### Visualization
- **Tableau** - Dashboard embedding
- **Looker** - Look integration
- **Superset** - SQL editor

### Security
- **Presidio** - PII detection and masking (presidio-analyzer)
- **OWASP** - HTML sanitization, security headers

## SDK Clients

### Java Client
- **openmetadata-java-client** - Generated from OpenAPI spec
- **Maven**: org.openmetadata:openmetadata-java-client

### Python Client
- **ingestion module** - Includes REST API client
- **Pydantic models** - Generated from JSON schemas

### JavaScript/TypeScript Client
- **Generated SDK** - Auto-generated from OpenAPI
- **Frontend SDK** - React hooks and utilities

## CI/CD Integrations

### GitHub Actions
- Build and test automation
- Docker image publishing
- Release management

### SonarCloud
- Code quality gates
- Coverage tracking
- Security hotspots

### Snyk
- Dependency vulnerability scanning
- Container image scanning
- Code security analysis

## Configuration Reference

### Authentication Flow
```
User → Frontend (React) → Backend (Dropwizard)
                          ↓
                    pac4j / OIDC
                          ↓
                    Identity Provider
                          ↓
                    JWT Token
                          ↓
                    Token stored in client
```

### Data Ingestion Flow
```
OpenMetadata Server → Airflow → DAG Generation
                              ↓
                    Python Ingestion (venv)
                              ↓
                    Source Connectors (75+)
                              ↓
                    OpenMetadata API
                              ↓
                    Elasticsearch / MySQL
```

### Search Flow
```
UI Search Query → Backend → Elasticsearch/OpenSearch
                  ↓
            Index Updates ← Ingestion Pipeline
```

## Environment Variable Summary

| Category | Variables | Description |
|----------|-----------|-------------|
| **Server** | `SERVER_PORT`, `SERVER_HOST`, `BASE_PATH` | HTTP configuration |
| **Database** | `DB_*`, `OM_DATABASE` | MySQL/PostgreSQL connection |
| **Search** | `SEARCH_TYPE`, `ELASTICSEARCH_*` | Search engine config |
| **Auth** | `AUTHENTICATION_PROVIDER`, `OIDC_*`, `SAML_*` | Identity provider |
| **Pipeline** | `PIPELINE_SERVICE_CLIENT_*`, `AIRFLOW_*` | Airflow integration |
| **Secrets** | `SECRET_MANAGER`, `OM_SM_*` | Secrets provider |
| **Cloud** | `AWS_*`, `AZURE_*`, `GOOGLE_*` | Cloud credentials |
| **Email** | `SMTP_*`, `OM_EMAIL_*` | Notification settings |
| **Monitoring** | `EVENT_MONITOR_*`, `LOG_LEVEL` | Observability |
