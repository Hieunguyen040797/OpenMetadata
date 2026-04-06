# OpenMetadata Technology Stack

## Languages & Runtimes

| Layer | Language | Version | Notes |
|-------|----------|---------|-------|
| **Backend Services** | Java | 21 | LTS, uses virtual threads (Project Loom) |
| **Frontend** | TypeScript/JavaScript | Node 22+ | React 18, Yarn package manager |
| **Python Ingestion** | Python | 3.10-3.11 | 3.11 recommended, venv required |
| **Build Tool** | Maven | 3.6+ | Multi-module project |
| **Container Runtime** | Docker | - | Multiple Dockerfiles for different components |

## Backend Stack (Java)

### Framework & Core
- **Dropwizard 5.0** - REST API framework (JAX-RS 3.x, Jersey 3.x)
- **Jetty 12.1.6** - HTTP server (EE10, virtual thread support)
- **Java 21** - Runtime with Project Loom virtual threads enabled by default

### Data & Persistence
- **MySQL 9.x** - Default relational database (driver: mysql-connector-j 9.3.0)
- **PostgreSQL 42.7.x** - Alternative relational database
- **HikariCP 7.x** - Connection pooling
- **JDBI 3.37.1** - SQL object mapping
- **Flyway 9.22.3** - Database migrations (SQL-based)
- **Elasticsearch 7.17.x / OpenSearch 2.6.x** - Search and indexing

### Security & Authentication
- **pac4j 5.7.0** - Authentication/authorization library
- **JWT (java-jwt 4.4.0, jjwt 0.9.1)** - Token handling
- **BCrypt 0.10.2** - Password hashing
- **Fernet 1.5.0** - Encryption
- **OWASP HTML Sanitizer 20260101** - XSS protection

### API & Documentation
- **Swagger/OpenAPI 2.x** - API documentation (swagger-maven-plugin-jakarta 2.2.30)
- **ANTLR 4.13.2** - Grammar parsing for FQN and query parsing

### Workflow & Scheduling
- **Quartz Scheduler 2.5.0** - Job scheduling
- **Flowable 7.2.0** - Workflow engine for governance
- **Apache Airflow 3.1.7** - Pipeline orchestration (Python, integrated via REST)

### ML & AI (Vector Embeddings)
- **AWS BedRock Runtime SDK** - ML embeddings (via AWS SDK 2.x)
- **DJL (Deep Java Library) 0.34** - PyTorch-based embeddings (ai.djl:api, ai.djl.pytorch:engine, ai.djl.huggingface:tokenizers)

### Caching & Messaging
- **Redis/Lettuce 6.7.1** - Distributed caching
- **Socket.IO 4.x** - Real-time WebSocket communication
- **Kafka (confluent_kafka 2.x)** - Event streaming
- **LMAX Disruptor 3.4.4** - High-performance concurrency

### Logging & Monitoring
- **Logback 1.5.x** - Logging framework
- **Micrometer 1.14.5** - Metrics abstraction
- **Prometheus** - Metrics registry and monitoring
- **Elasticsearch** - Log aggregation

### JSON & Data Processing
- **Jackson 2.18.6** - JSON serialization (databind, annotations, blackbird)
- **Gson 2.11.0** - Alternative JSON library
- **JSON Schema Validator (networknt 2.0.1)** - Schema validation
- **JSON Patch (java-json-tools 1.13)** - RFC 6902 JSON Patch
- **Apache Calcite 1.36.0** - SQL parsing
- **JSqlParser 4.9** - SQL statement parsing
- **Apache Jena 4.10.0** - RDF/linked data processing
- **CommonMark 0.26.0** - Markdown parsing

### Email & Notifications
- **Simple Java Mail 8.12.2** - Email sending
- **Slack API (bolt-servlet 1.44.1)** - Slack notifications

### Testing
- **JUnit 5.11.4** - Unit testing
- **Mockito 5.14.2** - Mocking framework
- **TestContainers** - Docker-based integration testing
- **Awaitility** - Async testing
- **JaCoCo** - Code coverage

### Build & Quality
- **Spotless 2.41.1** - Code formatting (Google Java Format)
- **Checkstyle** - Code style enforcement
- **SonarCloud** - Code quality analysis

## Frontend Stack

### Core Framework
- **React 18.2.x** - UI library
- **TypeScript 5.x** - Type safety
- **Vite 7.x** - Build tool and dev server
- **Tailwind CSS 4.x** - Utility-first CSS (with `tw:` prefix)
- **React Router 6.x** - Client-side routing

### State Management
- **Zustand** - Lightweight state management
- **React Hook Form 7.x** - Form state management
- **React Query / SWR patterns** - Server state

### UI Libraries
- **Ant Design 4.x** - Component library (legacy, being phased out)
- **openmetadata-ui-core-components** - New component library (Tailwind v4 + react-aria-components)
- **MUI 7.x** - Material UI (in transition)
- **Recharts 2.x** - Data visualization charts
- **React Flow 11.x** - Node-based graph visualization
- **Dagre** - Graph layout algorithms
- **React Grid Layout** - Dashboard layouts
- **TipTap** - Rich text editor

### Forms & Validation
- **React JSON Schema Form (RJSF) 5.x** - Dynamic form generation from JSON schemas
- **React Hook Form + Zod** - Form validation

### Internationalization
- **i18next 21.x** - Internationalization framework
- **react-i18next 15.x** - React bindings
- **17 locale files** - Supported languages

### Data Visualization
- **AntV G6 5.x** - Graph visualization (lineage, data quality)
- **ELK.js** - Elastic diagram rendering
- **Recharts** - Charting library

### Authentication (Frontend)
- **Auth0 React SDK** - Auth0 integration
- **Azure MSAL** - Microsoft identity platform
- **Okta React SDK** - Okta integration
- **OIDC Client** - Generic OpenID Connect

### HTTP & WebSocket
- **Axios** - HTTP client
- **Socket.IO Client** - Real-time communication
- **React Query patterns** - Data fetching

### Code Editors
- **CodeMirror 5.x** - Code/markdown editing
- **Monaco Editor** - Advanced code editing (planned)

### Testing
- **Jest 29.x** - Test runner
- **React Testing Library** - Component testing
- **Playwright 1.57** - E2E testing
- **ESLint 9.x** - Linting
- **Prettier** - Code formatting

## Python Ingestion Stack

### Runtime
- **Python 3.10-3.11** - Ingestion framework runtime
- **pip / setuptools** - Package management

### Data Modeling
- **Pydantic 2.x** (>=2.7.0, <2.12) - Data validation
- **Pydantic Settings 2.x** - Configuration management
- **SQLAlchemy 2.x** - ORM and SQL toolkit

### Database Connectors (75+)
Core drivers and connectors include:
- **MySQL** - pymysql, mysql-connector-python
- **PostgreSQL** - psycopg2-binary, sqlalchemy with geoalchemy2
- **Snowflake** - snowflake-sqlalchemy, snowflake-connector-python
- **BigQuery** - google-cloud-datacatalog, sqlalchemy-bigquery
- **Redshift** - redshift-jdbc (JDBC), sqlalchemy-redshift
- **MongoDB** - pymongo
- **Elasticsearch** - elasticsearch8
- **Delta Lake** - delta-spark, deltalake
- **Databricks** - databricks-sql-connector, databricks-sdk
- **Apache Kafka** - confluent_kafka
- **dbt** - collate-dbt-artifacts-parser
- **Great Expectations** - Data quality validation

### Data Processing
- **pandas 2.x** - Data manipulation
- **pyarrow 16.x** - Apache Arrow integration
- **numpy** - Numerical computing
- **scikit-learn** - ML for data profiling

### Lineage & Quality
- **collate-sqllineage** - SQL lineage extraction
- **collate-data-diff** - Data diffing
- **great-expectations** - Data quality testing
- **mlflow-skinny** - MLflow integration

### Cloud & Storage
- **boto3** - AWS SDK
- **azure-identity** - Azure authentication
- **google-cloud-*** - GCP libraries
- **s3fs, gcsfs, adlfs** - Cloud filesystem mounting

### Secrets Management
- **kubernetes** - Kubernetes client for secrets
- **azure-keyvault-secrets** - Azure Key Vault
- **google-cloud-secret-manager** - GCP Secret Manager
- **aws-secretsmanager, aws-ssm** - AWS Secrets Manager

### API Clients
- **requests** - HTTP client
- **httpx 0.28.x** - Async HTTP client
- **elasticsearch8** - ES/OpenSearch client
- **opensearch-py** - OpenSearch client

### Workflow
- **apache-airflow 3.1.7** - Pipeline orchestration
- **dagster** - Alternative orchestrator
- **opentelemetry-exporter-otlp** - Observability

### Dev Tools
- **pytest** - Testing framework
- **pylint 3.2.x** - Code linting
- **basedpyright** - Type checking
- **black, isort, pycln** - Code formatting
- **factory-boy** - Test fixtures

## Infrastructure & DevOps

### Containers
- **Docker** - Containerization
- **docker-compose** - Local development orchestration
- **Kubernetes** - Production orchestration
- **Helm** - Kubernetes package manager

### Service Images
| Service | Base Image | Version |
|---------|-----------|---------|
| MySQL | docker.getcollate.io/openmetadata/db | 1.12.0-SNAPSHOT |
| Server | docker.getcollate.io/openmetadata/server | 1.12.0-SNAPSHOT |
| Ingestion | openmetadata/ingestion | 1.12.0-SNAPSHOT |
| Elasticsearch | elasticsearch:9.3.0 / 7.10.2 | - |

### Kubernetes Operator
- **Java Operator SDK 4.9.x** - Custom operator framework
- **Fabric8 Kubernetes Client** - Kubernetes API access
- **Spring Boot 3.3.x** - Operator runtime
- **Quarkus alternative** - Not used (uses Spring Boot)

### CI/CD
- **GitHub Actions** - CI pipeline
- **SonarCloud** - Code quality gates
- **Snyk** - Security scanning

### Observability
- **Prometheus** - Metrics collection
- **OpenTelemetry** - Tracing (exported via OTLP)
- **Elasticsearch** - Log aggregation

## Build System

### Root Makefile Targets
```bash
make prerequisites     # Check system requirements
make generate         # Generate Pydantic/ANTLR models
make yarn_install_cache  # Install UI dependencies
make run_e2e_tests    # Full E2E test suite
```

### Maven Modules
1. **openmetadata-spec** - JSON Schema definitions and generated Java types
2. **openmetadata-service** - Core REST API server
3. **openmetadata-sdk** - Client SDK
4. **openmetadata-ui-core-components** - Shared UI component library
5. **openmetadata-ui** - React frontend application
6. **openmetadata-k8s-operator** - Kubernetes operator
7. **openmetadata-clients** - Language-specific clients
8. **openmetadata-dist** - Distribution packaging
9. **openmetadata-mcp** - Model Context Protocol integration
10. **common** - Shared utilities

### Python Module Structure
```
ingestion/src/metadata/
├── generated/          # Auto-generated from JSON schemas
├── ingestion/          # Connector implementations
│   ├── source/         # Database/service connectors (75+)
│   ├── sink/           # Output destinations
│   └── transform/     # Data transformers
├── profiler/           # Data profiling
├── lineage/            # Lineage extraction
├── api/                # REST API client
└── test_suite.py       # Test orchestration
```

## Configuration Management

### Environment Variables
All configuration uses environment variables with sensible defaults:

**Database**
- `DB_SCHEME` - mysql or postgresql
- `DB_HOST`, `DB_PORT`, `DB_USER`, `DB_PASSWORD`
- `OM_DATABASE` - Database name

**Search**
- `SEARCH_TYPE` - elasticsearch or opensearch
- `ELASTICSEARCH_HOST`, `ELASTICSEARCH_PORT`
- `AWS_*` - For OpenSearch IAM authentication

**Authentication**
- `AUTHENTICATION_PROVIDER` - basic, google, okta, azure, auth0, saml, ldap, custom-oidc
- `OIDC_*` - OIDC configuration parameters
- `RSA_PUBLIC_KEY_FILE_PATH`, `RSA_PRIVATE_KEY_FILE_PATH` - JWT signing

**Secrets Manager**
- `SECRET_MANAGER` - db, aws, azure, google, kubernetes
- `OM_SM_*` - Provider-specific credentials

**Pipeline Client**
- `PIPELINE_SERVICE_CLIENT_ENDPOINT` - Airflow/Workflow endpoint
- `PIPELINE_SERVICE_CLIENT_CLASS_NAME` - Client implementation class

### Configuration Files
- `conf/openmetadata.yaml` - Main server configuration
- `conf/operations.yaml` - Operations/monitoring settings
- `docker/docker-compose-*.yml` - Container orchestration configs
