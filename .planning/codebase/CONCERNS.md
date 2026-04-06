# OpenMetadata Codebase Concerns

Technical debt, known issues, security concerns, performance issues, and fragile areas identified through codebase analysis.

## Critical Technical Debt

### 1. Monster Classes (Code Size Violations)

The project violates its own Kafka-grade code standards with excessively large classes:

| File | Lines | Severity |
|------|-------|----------|
| `CollectionDAO.java` | 11,646 | CRITICAL |
| `EntityRepository.java` | 10,388 | CRITICAL |
| `EntityResource.java` | ~3,000+ | HIGH |

**Impact:** These files are nearly impossible to maintain, test, and review. They contain:
- Complex business logic mixed with data access
- Deep inheritance hierarchies that are hard to follow
- High cyclomatic complexity

### 2. N+1 Query Problem (Historical)

The codebase contains dedicated test files for N+1 query validation:
- `N1QueryFixValidationTest.java`
- `EntityRelationshipPerformanceTest.java`

**Current State:** The N+1 issue was partially addressed with batch fetching optimization. When 3+ fields are requested, batch fetching is used. However:
- Single-field queries still use individual fetches
- The optimization threshold (3 fields) is arbitrary
- Legacy code paths may still trigger N+1 behavior

**Affected repositories:**
- `TableRepository`
- `GlossaryTermRepository`
- `TagRepository`
- `TestCaseRepository`

### 3. Deprecated Code Still in Production

Multiple `@Deprecated` classes and methods remain active:

**Entirely deprecated (Release 1.1+):**
- `SetEntityCertificationTask.java` + `SetEntityCertificationImpl.java`
- `SetGlossaryTermStatusTask.java` + `SetGlossaryTermStatusImpl.java`
- `RollbackEntityImpl.java`

**Deprecated methods:**
- `EntityRepository.java:1225` - legacy method
- `EntityRepository.java:3064` - deprecated
- `EntityTimeSeriesDAO.java:403` - since 1.1.1
- `EntityTimeSeriesDAO.java:776` - since 1.1.1
- `CollectionDAO.java` - multiple methods since Release 1.1
- `SearchResource.java:151` - `forRemoval = true`

**Risk:** Deprecated code may be removed without notice, breaking dependent functionality.

## Security Concerns

### 1. Authentication Infrastructure

**Files requiring review:**
- `NoopAuthenticator.java` - Marked "deprecated unused" but still exists
- `GoogleAuthValidator.java` - Deprecated authenticator
- `BasicAuthenticator.java:433` - TODO: "currently allow single login from a place, later multiple login can be added"

### 2. Authorization Gaps

**Known TODO in security code:**
- `ThreadResourceContext.java:35`:
  ```java
  // TODO: Fix this this should be thread.getEntity().getDomain()
  @Override
  public List<EntityReference> getDomains() {
    return null;  // Returns null instead of actual domain
  }
  ```

This creates an authorization bypass where conversation threads may not properly enforce domain-based access control.

### 3. Secrets Management

The project has extensive secrets management infrastructure:
- AWS Secrets Manager
- GCP Secrets Manager
- Azure Key Vault
- HashiCorp Vault

**Concern:** Integration tests contain mock credentials:
- `MOCK_GET_SOURCE_CONNECTION = "XXXXX-XXXX-XXXXX"` in test files

### 4. JWT and Token Handling

- `JwtFilterTest.java` - extensive JWT testing suggests complex security logic
- `JWTTokenGeneratorTest.java` - token generation testing
- `MultiUrlJwkProviderTest.java` - multi-URL JWK key handling

## Performance Concerns

### 1. Entity Relationship Queries

The `entity_relationship` table is heavily used with multiple index strategies:
- `idx_entity_relationship_from_composite`
- `idx_entity_relationship_to_composite`
- `idx_entity_relationship_bidirectional`

**Performance test expectations:**
- findFrom queries: < 100ms with index
- findTo queries: < 100ms with index
- Bidirectional queries: < 50ms with index

### 2. Search Infrastructure

Dual search engine support creates complexity:
- Elasticsearch 7.17+
- OpenSearch 2.6+

**Code duplication:** Nearly identical classes for both engines with switch dispatching:
- `ElasticSearchSearchManager.java`
- `OpenSearchSearchManager.java`
- `ElasticSearchClient.java`
- `OpenSearchClient.java`

### 3. Large File Ingestion Risks

Search index handling for large datasets:
- Bulk indexing operations with configurable batch sizes
- Circuit breaker patterns for resilience
- Distributed indexing coordination

## Fragile Areas

### 1. Complex Inheritance

70+ repository classes extend `EntityRepository<T>`:
- Steep inheritance hierarchy
- Many repository-specific overrides
- Hard to trace method resolution

### 2. Field Parsing System

The `EntityUtil.Fields` system is:
- Critical for all entity operations
- String-based field matching prone to typos
- No compile-time validation

### 3. Migration Complexity

**Migration count:**
- 50+ MySQL migrations
- 50+ PostgreSQL migrations

**Migration utilities per version:**
- `v110/MigrationUtil.java`
- `v1100/MigrationUtil.java`
- `v1102/MigrationUtil.java`
- `v1115/MigrationUtil.java`
- `v1122/MigrationUtil.java`

**Risk:** Migration failures can corrupt database state.

### 4. Dual Database Support

MySQL (default) and PostgreSQL support requires:
- Conditional SQL in migrations
- Different index strategies (partial indexes vs. MySQL)
- Separate migration paths

## Missing or Incomplete Tests

### Integration Test Blocklist

**TableResourceIT.java** has 50+ TODO items for SDK migration:
- Column pagination tests
- Table joins tests
- Sample data tests
- Custom metrics tests
- Lineage column rename tests
- CSV import/export tests

**Test coverage gaps:**
- Authorization tests need rework (`BaseEntityIT.java:531`)
- ETag/patch tests incomplete (`BaseEntityIT.java:1784-1789`)
- Concurrent update tests skipped
- SDK-dependent feature tests blocked

### Python Ingestion Tests

- `test_deprecated_workflow_functions.py` - TODO to remove deprecated functions in Release 1.6
- `test_sql_lineage.py` - CTE parsing issues with sqlglot
- `test_query_masker_dialect_specific.py` - Multiple STRUCT masking issues
- Various E2E tests skipped due to credential issues

## Code Quality Issues

### 1. Wildcard Imports

Several files use wildcard imports against project standards:
- `EntityUtil.java:64`: `import org.openmetadata.schema.type.*;`
- Various test files with `import static ...*`

### 2. Test Infrastructure

**Antipatterns found:**
- Mocks for internal class wiring rather than integration tests
- Thread.sleep in tests (should use Awaitility)
- Overly mocked unit tests that verify wiring, not behavior

### 3. Frontend Concerns

**TypeScript `any` usage found in:**
- `ServiceConnectionDetails.component.tsx` - `useState<Record<string, any>>`
- Multiple test files with `as any` casts
- Legacy component patterns before strict typing

**Tree virtualization TODO:**
- `useTreeVirtualization.tsx` - "Implement virtualization using @tanstack/react-virtual or react-window"
- Performance concern for large tree data structures

## Build and Configuration Issues

### 1. Nox/Python Build Configuration

- `noxfile.py` has TODOs for Python 3.9 compatibility
- Platform-specific dependency issues (psycopg2-binary on Mac)
- spaCy dependency installation complications

### 2. Docker Hack for M1 Macs

`docker-compose-gcp.yml:288`:
```yaml
# HACK: This is hack for M1 mac or later to avoid aborting JVM by the Google Secret Manager library.
```

## Migration Path Recommendations

### Immediate Actions

1. **Deprecation cleanup**: Remove or finalize deprecated classes before 2.0 release
2. **Code splitting**: Break `EntityRepository` and `CollectionDAO` into focused classes
3. **ThreadResourceContext fix**: Complete the domain resolution implementation
4. **Test migration**: Prioritize SDK migration for TableResourceIT tests

### Medium-term Actions

1. **Search engine abstraction**: Consolidate ES/OS logic into unified interface
2. **Field system typing**: Add compile-time validation for field names
3. **Repository pattern refactor**: Extract common patterns into mixins/traits

### Long-term Actions

1. **Monolith decomposition**: Consider service boundaries for large modules
2. **Database migration consolidation**: Single database strategy
3. **Frontend component library migration**: Complete transition to `openmetadata-ui-core-components`

## Risk Summary

| Category | Risk Level | Items |
|----------|------------|-------|
| Code Size | CRITICAL | 3 monster classes |
| Security | HIGH | Authorization gaps, deprecated auth |
| Performance | MEDIUM | N+1 potential, search complexity |
| Technical Debt | HIGH | Deprecated code, complex inheritance |
| Testing | MEDIUM | Blocked tests, missing coverage |
| Maintainability | HIGH | Migration complexity, dual DB |

## Documentation References

- Architecture overview: `DEVELOPER.md`
- Frontend patterns: `CLAUDE.md`
- Code standards: `skills/standards/code_style.md`
- Performance guidelines: `skills/standards/performance.md`
