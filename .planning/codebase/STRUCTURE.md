# OpenMetadata Directory Structure

## Root Level

```
D:/OpenMetadata/
├── openmetadata-service/     # Java backend (Dropwizard REST API)
├── openmetadata-ui/         # React frontend application
├── openmetadata-spec/       # JSON Schema definitions + generated code
├── ingestion/               # Python ingestion framework
├── openmetadata-airflow-apis/
├── openmetadata-clients/    # Generated API clients (JS, Python)
├── openmetadata-sdk/        # Java SDK for API access
├── openmetadata-ui-core-components/ # Shared React component library
├── openmetadata-dist/       # Distribution packaging
├── openmetadata-k8s-operator/
├── openmetadata-mcp/        # MCP server implementation
├── openmetadata-integration-tests/
├── bootstrap/               # Database migrations & sample data
├── conf/                    # Configuration files
├── docker/                  # Docker configurations
├── docs/                    # Documentation
├── examples/                # Example code and data
├── scripts/                 # Build and utility scripts
├── common/                  # Shared utilities
├── openmetadata-shaded-deps/
├── CLAUDE.md                # AI development guide (critical)
├── DEVELOPER.md             # Architecture deep dives
└── pom.xml                  # Maven root (defines 12 modules)
```

## openmetadata-service/ - Java Backend

```
openmetadata-service/
├── pom.xml
├── src/main/java/org/openmetadata/service/
│   ├── OpenMetadataApplication.java      # Dropwizard entry point
│   ├── OpenMetadataApplicationConfig.java # YAML config binding
│   ├── Entity.java                       # Central entity registry
│   ├── ResourceRegistry.java
│   ├── TypeRegistry.java
│   │
│   ├── resources/                        # REST API endpoints
│   │   ├── EntityResource.java          # Base REST resource
│   │   ├── Collection.java
│   │   ├── ai/                          # AI application endpoints
│   │   ├── analytics/
│   │   ├── apis/
│   │   ├── bots/
│   │   ├── databases/                   # Table, Schema, Database
│   │   ├── dashboards/
│   │   ├── data/
│   │   ├── datainsight/
│   │   ├── dqtests/                     # Data quality tests
│   │   ├── feeds/                       # Activity feed
│   │   ├── glossary/
│   │   ├── lineage/
│   │   ├── pipelines/
│   │   ├── policies/
│   │   ├── query/
│   │   ├── search/
│   │   ├── services/                    # Service management
│   │   ├── tags/
│   │   ├── teams/                       # User/Team management
│   │   ├── topics/
│   │   └── types/
│   │
│   ├── jdbi3/                            # Data access layer
│   │   ├── CollectionDAO.java           # All DAO interfaces
│   │   ├── EntityRepository.java        # Base repository
│   │   ├── TableRepository.java
│   │   ├── TagRepository.java
│   │   ├── GlossaryRepository.java
│   │   ├── GlossaryTermRepository.java
│   │   ├── TestCaseRepository.java
│   │   └── locator/
│   │
│   ├── search/                           # Elasticsearch indexing
│   │   ├── IndexMappingLoader.java
│   │   └── indexes/                     # Search index classes
│   │
│   ├── security/                         # Auth & authorization
│   ├── auth/                            # Authentication handlers
│   ├── events/                          # Event system
│   ├── notifications/                   # WebSocket notifications
│   ├── workflows/                       # Background job handlers
│   ├── apps/                            # Application plugins
│   ├── migration/                        # Database migrations
│   ├── dataInsight/
│   ├── governance/
│   ├── clients/                         # External service clients
│   ├── jdbi3/api/                       # JDBI SQL objects
│   └── [other packages]                 # Various utilities
│
└── src/main/resources/
    ├── openmetadata.yaml                # Default configuration
    └── [other resources]
```

## openmetadata-spec/ - Schema Definitions

```
openmetadata-spec/
├── pom.xml
└── src/main/resources/json/schema/
    ├── entity/
    │   ├── data/
    │   │   ├── table.json
    │   │   ├── view.json
    │   │   ├── column.json
    │   │   ├── topic.json
    │   │   ├── dashboard.json
    │   │   ├── pipeline.json
    │   │   ├── container.json
    │   │   ├── mlmodel.json
    │   │   └── ...
    │   │
    │   ├── services/
    │   │   ├── databaseService.json
    │   │   ├── dashboardService.json
    │   │   ├── pipelineService.json
    │   │   ├── messagingService.json
    │   │   ├── mlmodelService.json
    │   │   ├── searchService.json
    │   │   ├── storageService.json
    │   │   ├── metadataService.json
    │   │   ├── securityService.json
    │   │   ├── apiService.json
    │   │   ├── llmService.json
    │   │   ├── mcpService.json
    │   │   ├── driveService.json
    │   │   └── ingestionPipelines/
    │   │
    │   ├── services/connections/
    │   │   ├── database/              # Database connector configs
    │   │   │   ├── mysql.json
    │   │   │   ├── postgres.json
    │   │   │   ├── snowflake.json
    │   │   │   ├── bigquery.json
    │   │   │   ├── redshift.json
    │   │   │   ├── databricks.json
    │   │   │   └── [40+ more]
    │   │   ├── dashboard/
    │   │   ├── pipeline/
    │   │   └── [other types]
    │   │
    │   ├── teams/
    │   │   ├── user.json
    │   │   ├── team.json
    │   │   └── role.json
    │   │
    │   ├── policies/
    │   │   ├── policy.json
    │   │   └── dataProduct.json
    │   │
    │   ├── feed/
    │   │   ├── thread.json
    │   │   └── activityFeed.json
    │   │
    │   ├── classification/
    │   │   ├── tag.json
    │   │   └── classification.json
    │   │
    │   ├── glossary/
    │   │   ├── glossary.json
    │   │   └── glossaryTerm.json
    │   │
    │   └── [other entity types]
    │
    ├── api/
    │   └── [Create/Update request DTOs]
    │
    └── type/
        ├── entityReference.json       # Cross-entity references
        ├── tagLabel.json
        ├── changeEvent.json
        └── [shared types]
```

## ingestion/ - Python Framework

```
ingestion/
├── README.md
├── Makefile
├── requirements.txt
├── setup.py
├── src/metadata/
│   ├── __init__.py
│   ├── __main__.py                    # CLI entry point
│   ├── cmd.py                         # Command handlers
│   ├── __version__.py
│   │
│   ├── ingestion/                     # Core ingestion framework
│   │   ├── api/                       # REST sink for API calls
│   │   ├── source/
│   │   │   ├── database/              # 40+ database connectors
│   │   │   │   ├── mysql/
│   │   │   │   ├── postgres/
│   │   │   │   ├── snowflake/
│   │   │   │   ├── bigquery/
│   │   │   │   ├── databricks/
│   │   │   │   ├── [40+ more]
│   │   │   │   ├── common_db_source.py
│   │   │   │   ├── database_service.py
│   │   │   │   └── lineage_source.py
│   │   │   ├── dashboard/             # Dashboard connectors
│   │   │   ├── pipeline/             # Pipeline connectors
│   │   │   ├── messaging/            # Messaging connectors
│   │   │   ├── mlmodel/              # ML model connectors
│   │   │   ├── storage/              # Storage connectors
│   │   │   ├── metadata/             # Metadata service connectors
│   │   │   ├── search/              # Search connectors
│   │   │   └── security/            # Security connectors
│   │   │
│   │   ├── sink/                     # Output sinks
│   │   ├── processor/                # Data processors
│   │   ├── stage/                    # Staging intermediate data
│   │   ├── models/                   # Pydantic models
│   │   ├── ometa/                    # OpenMetadata client
│   │   ├── connections/              # Connection utilities
│   │   ├── bulksink/                 # Bulk write operations
│   │   └── lineage/                  # Lineage processing
│   │
│   ├── generated/                     # Generated Pydantic models
│   ├── sdk/                          # Python SDK
│   ├── profiler/                     # Data profiling
│   │   ├── api/                     # Profiler API
│   │   ├── metrics/                 # Metric implementations
│   │   └── orm/                      # SQLAlchemy ORM profiling
│   │
│   ├── data_quality/                 # Data quality testing
│   │   ├── validators/
│   │   └── validations/
│   │
│   ├── workflow/                     # Workflow definitions
│   ├── applications/                 # Application integrations
│   ├── automations/                  # Automation utilities
│   ├── cli/                          # CLI utilities
│   ├── clients/                      # API clients
│   ├── config/                       # Configuration
│   ├── utils/                        # Utilities
│   │   ├── dependency_injector/     # DI container
│   │   └── [other utils]
│   └── readers/                      # Data readers
│
├── operators/                        # Airflow operators
├── plugins/                          # Extension plugins
├── tests/
│   ├── unit/
│   ├── integration/
│   └── cli_e2e/
├── airflow-provider-openmetadata/
└── [other test directories]
```

## openmetadata-ui/ - React Frontend

```
openmetadata-ui/
├── src/main/resources/ui/
│   ├── package.json
│   ├── yarn.lock
│   ├── tsconfig.json
│   ├── vite.config.ts               # Vite build config
│   ├── eslint.config.mjs
│   ├── jest.config.js
│   ├── playwright.config.ts
│   │
│   ├── src/
│   │   ├── App.tsx                 # React entry point
│   │   ├── index.tsx
│   │   ├── pages/                  # Page components (140+ pages)
│   │   │   ├── TableDetailsPageV1/
│   │   │   ├── DatabaseDetailsPage/
│   │   │   ├── DashboardDetailsPage/
│   │   │   ├── ExplorePage/
│   │   │   ├── SettingsPage/
│   │   │   ├── LoginPage/
│   │   │   └── [130+ more]
│   │   │
│   │   ├── components/             # Reusable components
│   │   ├── pages/                 # Page-level components
│   │   ├── hooks/                 # Custom React hooks
│   │   ├── context/              # React Context providers
│   │   ├── utils/                # Utility functions
│   │   ├── constants/            # Application constants
│   │   ├── enums/               # TypeScript enums
│   │   │
│   │   ├── generated/            # Generated TypeScript types
│   │   ├── rest/               # API client functions
│   │   │
│   │   ├── locale/             # i18n translations
│   │   │   └── languages/
│   │   │       └── en-us.json
│   │   │
│   │   ├── styles/             # Global styles
│   │   ├── assets/            # Static assets
│   │   └── [other directories]
│   │
│   ├── public/                  # Static public assets
│   ├── scripts/                # Build scripts
│   └── playwright/            # E2E tests
│
└── [other files]
```

## openmetadata-ui-core-components/ - Component Library

```
openmetadata-ui-core-components/
└── src/main/resources/ui/
    ├── src/
    │   ├── components/
    │   │   ├── base/           # Base components
    │   │   ├── application/   # Application components
    │   │   ├── foundations/   # Foundation components
    │   │   └── index.ts
    │   ├── styles/
    │   │   └── globals.css    # CSS custom properties
    │   └── [other]
    └── package.json
```

## bootstrap/ - Database Migrations

```
bootstrap/
├── sql/
│   ├── migrations/
│   │   └── native/
│   │       └── [version]/
│   │           ├── mysql/
│   │           │   └── schemaChanges.sql
│   │           └── postgres/
│   │               └── schemaChanges.sql
│   │
│   └── schema/
│       └── [schema files]
│
└── [other bootstrap files]
```

## Configuration Files

```
conf/
├── openmetadata.yaml              # Main backend config
├── log4j2.xml                     # Logging config
├── users.json                     # Sample users
└── [environment variants]
```

## Docker

```
docker/
├── development/
│   └── docker-compose.yml         # Local dev environment
├── [other docker configs]
└── run_local_docker.sh            # Dev startup script
```

## Key Files and Their Purposes

| File | Purpose |
|------|---------|
| `CLAUDE.md` | AI-assisted development guide |
| `DEVELOPER.md` | Architecture deep dives, patterns |
| `pom.xml` (root) | Maven multi-module definition |
| `pom.xml` (service) | Backend build config |
| `openmetadata.yaml` | Backend runtime configuration |
| `package.json` (ui) | Frontend dependencies and scripts |
| `Makefile` | Build automation |
| `DEVELOPER_HANDBOOK.md` (ui) | UI-specific development guide |

## Naming Conventions

### Java
- Classes: PascalCase (e.g., `TableRepository`, `EntityResource`)
- Methods: camelCase (e.g., `setFullyQualifiedName`, `getEntityById`)
- Constants: UPPER_SNAKE_CASE (e.g., `COLLECTION_PATH`, `FIELD_OWNERS`)
- Packages: lowercase with dots (e.g., `org.openmetadata.service.resources.databases`)

### Python
- Classes: PascalCase (e.g., `DatabaseServiceSource`, `TopologyNode`)
- Functions/Methods: snake_case (e.g., `yield_table`, `get_database_names`)
- Modules: snake_case (e.g., `common_db_source.py`)
- Constants: UPPER_SNAKE_CASE

### TypeScript/React
- Components: PascalCase (e.g., `TableDetailsPage`)
- Functions/Hooks: camelCase (e.g., `useTranslation`, `useTableStore`)
- Files: PascalCase for components (e.g., `Table.component.tsx`)
- i18n keys: kebab-case (e.g., `add-entity`, `activity-feed`)

### JSON Schema
- Files: camelCase (e.g., `table.json`, `createTable.json`)
- Property names: camelCase
- Entity names in `$id`: PascalCase matching filename

## Entry Points

| Component | Path |
|-----------|------|
| Backend | `openmetadata-service/src/main/java/.../OpenMetadataApplication.java` |
| Frontend | `openmetadata-ui/src/main/resources/ui/src/App.tsx` |
| Ingestion CLI | `ingestion/src/metadata/__main__.py` |
| Maven Build | Root `pom.xml` |

## Service Resource Structure (REST API)

```
/v1/aiApplications/
/v1/analytics/
/v1/bots/
/v1/charts/
/v1/databases/
/v1/databases/{id}/columns/
/v1/databases/{id}/version
/v1/dashboards/
/v1/dataProducts/
/v1/domains/
/v1/dqTests/
/v1/feeds/
/v1/glossaries/
/v1/lineage/
/v1/metrics/
/v1/pipelines/
/v1/policies/
/v1/queries/
/v1/search/
/v1/services/
/v1/tags/
/v1/teams/
/v1/topics/
/v1/types/
/v1/users/
/v1/webhooks/
```