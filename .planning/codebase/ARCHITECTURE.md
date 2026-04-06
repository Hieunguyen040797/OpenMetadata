# OpenMetadata Architecture

## Overview

OpenMetadata is a unified metadata platform providing data discovery, observability, and governance. The system follows a **schema-first** architecture where JSON Schema definitions drive code generation across all languages.

## Core Components

```
┌─────────────────────────────────────────────────────────────────┐
│                        OpenMetadata                              │
├─────────────────────────────────────────────────────────────────┤
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────────────┐  │
│  │  openmetadata│  │  openmetadata│  │      ingestion/       │  │
│  │    -service  │  │    -ui       │  │  (Python connectors)  │  │
│  │  (Java 21)   │  │  (React/TS)  │  │   75+ connectors     │  │
│  └──────────────┘  └──────────────┘  └──────────────────────┘  │
│           │                │                      │             │
│           └────────────────┼──────────────────────┘             │
│                            │                                     │
│                     ┌──────┴──────┐                              │
│                     │ openmetadata│                              │
│                     │    -spec    │                              │
│                     │ (JSON Schema)│                             │
│                     └─────────────┘                              │
└─────────────────────────────────────────────────────────────────┘
```

## Data Flow

```
┌─────────────┐     ┌──────────────┐     ┌──────────────────┐
│ Data Sources│────▶│   Ingestion  │────▶│  Metadata Store  │
│ (75+ types) │     │   (Python)   │     │  (MySQL/Postgres)│
└─────────────┘     └──────────────┘     └────────┬─────────┘
                                                    │
                    ┌───────────────────────────────┼───────────────┐
                    │                               │               │
                    ▼                               ▼               ▼
           ┌─────────────────┐           ┌──────────────┐  ┌──────────────┐
           │  Search Index   │           │  Change      │  │   WebSocket  │
           │(Elasticsearch/  │           │  Events     │  │  Notifications│
           │  OpenSearch)    │           │  (Audit Log) │  │  (Real-time)  │
           └─────────────────┘           └──────────────┘  └──────────────┘
                    │                                                       │
                    ▼                                                       ▼
           ┌─────────────────┐                                   ┌──────────────┐
           │  REST API       │◀──────────────────────────────────│  React UI    │
           │  (Dropwizard)   │                                   │  (TypeScript) │
           └─────────────────┘                                   └──────────────┘
```

## Architecture Patterns

### 1. Schema-First Design

JSON Schema definitions serve as the single source of truth for all code generation:

```
openmetadata-spec/src/main/resources/json/schema/
├── entity/              # Entity definitions
│   ├── data/           # Table, Topic, Dashboard, Pipeline
│   ├── services/       # Service configurations
│   │   └── connections/ # Connection schemas per connector
│   ├── teams/          # User, Team entities
│   ├── policies/       # Governance entities
│   └── feed/           # Activity feed
├── api/                 # Request/Response DTOs
└── type/               # Shared types (EntityReference, TagLabel)

Code Generation Pipeline:
  ├── Java:   jsonschema2pojo → openmetadata-spec/target/
  ├── Python: datamodel-code-generator → ingestion/src/metadata/generated/
  └── TypeScript: QuickType → openmetadata-ui/.../ui/src/generated/
```

### 2. Backend: Dropwizard REST API (Java 21)

**Framework:** Dropwizard 5.0 with Jersey JAX-RS, JDBI3 for persistence

**Key Classes:**
- `OpenMetadataApplication.java` - Main Dropwizard application entry point
- `OpenMetadataApplicationConfig.java` - YAML configuration binding
- `Entity.java` - Central entity registry singleton
- `EntityResource.java` - Base REST resource with CRUD operations
- `EntityRepository.java` - Abstract base for data access

**REST Resource Pattern:**
```java
@Path("/v1/myEntities")
@Tag(name = "MyEntities")
public class MyEntityResource extends EntityResource<MyEntity, MyEntityRepository> {
    public static final String COLLECTION_PATH = "/v1/myEntities/";
    // Endpoints inherited: GET list, GET/{id}, POST, PUT, PATCH, DELETE
}
```

**Data Access:**
- JDBI3 SQL framework (not JPA/Hibernate)
- `CollectionDAO` interface provides type-safe DAO methods
- Transactions via `@Transaction` annotation
- Flyway SQL migrations in `bootstrap/sql/migrations/`

### 3. Frontend: React/TypeScript SPA

**Framework:** React 18+ with TypeScript, Vite build tool, Yarn package manager

**Component Library:** `openmetadata-ui-core-components` (Tailwind CSS v4, react-aria-components)

**Styling:**
- Tailwind classes with `tw:` prefix for all utility classes
- CSS custom properties for design tokens (e.g., `--color-text-primary`)
- No MUI components (being migrated to core components)

**State Management:**
- Component-local: `useState`
- Feature-scoped: React Context providers
- Global: Zustand stores

**i18n:** react-i18next with keys in `locale/languages/en-us.json`

### 4. Python Ingestion Framework

**Pattern:** Declarative topology-based connectors

**Core Concepts:**
```python
class DatabaseServiceTopology(ServiceTopology):
    root = TopologyNode(
        producer="get_services",
        stages=[NodeStage(type_=DatabaseService, ...)],
        children=["database"],
    )
    database = TopologyNode(
        producer="get_database_names",
        stages=[NodeStage(type_=Database, processor="yield_database", ...)],
        children=["database_schema"],
    )
```

**Pipeline Architecture:**
```
Source (extracts metadata)
  → Processor (transforms/validates)
  → Stage (intermediate representation)
  → Sink (writes to OpenMetadata API)
```

**Source Hierarchy:**
```
Source (abstract)
  → DatabaseServiceSource (SQLAlchemy-based)
      → CommonDbSourceService
          → PostgresSource, MySQLSource, SnowflakeSource, etc.
```

### 5. Search Infrastructure

**Engine:** Elasticsearch 7.17+ or OpenSearch 2.6+

**Index Classes:** `openmetadata-service/src/main/java/.../search/indexes/`

**Indexing Flow:**
1. Entity created/updated via REST API
2. `EntityRepository` sets `supportsSearch = true`
3. Search document built in `*Index.java` class
4. Indexed to ES/OpenSearch
5. UI queries search index for discovery

### 6. Security & Authorization

**Authentication:** JWT-based with OAuth2/SAML support

**Authorization:** RBAC (Role-Based Access Control)

**Key Components:**
- `Authorizer` interface - injected into all resources
- `PolicyEvaluator` - evaluates rules against context
- `SubjectCache` - permission evaluation cache
- `OperationContext` - wraps operation type (CREATE, EDIT_TAGS, etc.)

### 7. Event System

**Change Events:**
- Automatic via `ChangeEventHandler` (JAX-RS ContainerResponseFilter)
- Persisted to `changeEvent` table
- Broadcast via WebSocket for real-time UI updates

**Event Types:** ENTITY_CREATED, ENTITY_UPDATED, ENTITY_SOFT_DELETED, ENTITY_DELETED

### 8. Fully Qualified Names (FQN)

Entities form a hierarchy with `.` separator:

```
databaseService.database.databaseSchema.table
databaseService.database.databaseSchema.table.column
dashboardService.dashboard
pipelineService.pipeline
```

Each repository must implement `setFullyQualifiedName()` to build FQN from parent + name.

## Key Configuration

**Backend Config:** `conf/openmetadata.yaml`
- Database: HikariCP connection pooling
- Elasticsearch: Search index configuration
- Authentication: JWT/OAuth2 settings
- Authorizer: RBAC policy configuration

**Frontend Config:** `openmetadata-ui/src/main/resources/ui/.env`

**Ingestion Config:** YAML-based workflow definitions

## Entry Points

| Component | Entry Point |
|-----------|-------------|
| Backend | `OpenMetadataApplication.java` |
| Frontend | `openmetadata-ui/src/main/resources/ui/src/App.tsx` |
| Ingestion CLI | `ingestion/src/metadata/__main__.py` |
| Maven Build | Root `pom.xml` |

## Maven Module Structure

```
openmetadata-platform (root pom)
├── openmetadata-spec          # JSON Schema definitions + generated POJOs
├── openmetadata-sdk           # Java SDK for API clients
├── common                     # Shared utilities
├── openmetadata-shaded-deps   # Bundled dependencies
├── openmetadata-service       # Dropwizard REST API
├── openmetadata-k8s-operator  # Kubernetes operator
├── openmetadata-integration-tests # Integration tests
├── openmetadata-mcp           # MCP server implementation
├── openmetadata-ui-core-components # Shared UI component library
├── openmetadata-ui            # React frontend application
├── openmetadata-dist          # Distribution packaging
└── openmetadata-clients       # Generated API clients
```

## Database Migrations

**Tool:** Flyway SQL migrations

**Location:** `bootstrap/sql/migrations/`

**Pattern:** Native SQL migrations with MySQL/PostgreSQL variants

```sql
bootstrap/sql/migrations/native/{version}/
├── mysql/schemaChanges.sql
└── postgres/schemaChanges.sql
```

## Design Patterns Summary

| Pattern | Location | Purpose |
|---------|----------|---------|
| Schema-first | Everywhere | JSON Schema drives code generation |
| Registry | `Entity.java` | Central entity type lookup |
| Template Method | Repositories | Base class defines skeleton |
| Strategy | Profilers/Samplers | SQLAlchemy vs Pandas implementations |
| Topology | Python connectors | Declarative entity traversal |
| Either monad | Python ingestion | Unified error handling |
| Factory | Interface resolution | Right implementation per service type |
