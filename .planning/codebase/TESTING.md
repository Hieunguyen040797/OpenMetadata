# OpenMetadata Testing Guide

This document describes the testing infrastructure, frameworks, patterns, and conventions for each language layer in OpenMetadata. All new code must include appropriate tests. CI will block PRs that do not meet coverage requirements.

---

## 1. Testing Philosophy

The project follows a strict philosophy:

- **Test real behavior, not mock wiring.** If a test requires mocking 3+ classes just to verify a method call, it's testing the wrong thing.
- **Prefer integration tests** over heavily-mocked unit tests. The project has full integration test infrastructure (Docker containers, real Elasticsearch, test databases).
- **Mocks are for boundaries, not internals.** Mock external services, not your own classes.
- **A test that mocks everything proves nothing.** Only verifies that mocks are wired correctly.
- **Test the outcome, not the implementation.** Assert on observable results (API responses, database state, stats values) rather than verifying internal method calls.
- **No `Thread.sleep()` in tests.** Use condition-based waiting or `Awaitility`.

---

## 2. Java Testing

**Reference:** `pom.xml`, `openmetadata-service/pom.xml`, `openmetadata-integration-tests/pom.xml`

### 2.1 Frameworks & Tools

| Purpose | Tool | Version |
|---------|------|---------|
| Unit testing | JUnit 5 (Jupiter) | Managed in `pom.xml` |
| Assertions | AssertJ | Via JUnit 5 bundled |
| Mocking | Mockito | Via JUnit 5 bundled |
| Coverage | Maven Surefire + JaCoCo | |
| Integration tests | JUnit 5 + Dropwizard TestKit | `openmetadata-integration-tests/` |
| Waiting | Awaitility | |
| Property-based testing | jqwik | |

### 2.2 Test Directory Structure

```
openmetadata-service/src/test/java/org/openmetadata/
├── csv/                    # CSV parsing tests
├── service/
│   ├── apps/              # App/bundle tests
│   ├── audit/             # Audit log tests
│   ├── clients/           # REST client tests
│   ├── events/            # Event system tests
│   ├── exception/          # Exception message tests
│   └── formatter/         # Message formatter tests
└── (others by package)

openmetadata-integration-tests/src/test/java/org/openmetadata/it/
├── k8s/                   # Kubernetes integration tests
└── tests/
    ├── AgentExecutionResourceIT.java
    ├── AIApplicationResourceIT.java
    ├── AlertsRuleEvaluatorResourceIT.java
    ├── BaseEntityIT.java              # Base class for all entity ITs
    ├── BaseServiceIT.java            # Base class for service ITs
    ├── ChartResourceIT.java
    ├── DashboardResourceIT.java
    ├── DatabaseResourceIT.java
    ├── GlossaryResourceIT.java
    └── ... (one *IT.java per entity)
```

### 2.3 Unit Test Patterns

Unit tests live in `openmetadata-service/src/test/java/` and use JUnit 5:

```java
import static org.junit.jupiter.api.Assertions.*;
import org.junit.jupiter.api.Test;

class MyServiceTest {
  @Test
  void testCalculateScore_returnsExpectedResult() {
    MyService svc = new MyService();
    double score = svc.calculateScore(10.0, 2.0);
    assertEquals(5.0, score, 0.001);
  }

  @Test
  void testFindByName_returnsNullForMissingEntity() {
    MyService svc = new MyService();
    Entity result = svc.findByName("doesNotExist");
    assertNull(result);
  }
}
```

Key patterns:
- **`assertEquals(actual, expected, delta)`** for floating-point comparisons
- **`assertThrows`** for expected exceptions
- **`assertThat(entity).isNull()`** for AssertJ fluent assertions
- **Never use `Thread.sleep()`** — use Awaitility:
  ```java
  await().atMost(5, TimeUnit.SECONDS).untilAsserted(() ->
      assertThat(processedCount.get()).isGreaterThan(0));
  ```

### 2.4 Integration Test Pattern (*IT.java)

Every new REST endpoint requires a corresponding `*IT.java` in `openmetadata-integration-tests/`. These tests:
- Boot a real Dropwizard application via `IntegrationTestSuite`
- Use real MySQL/PostgreSQL and Elasticsearch containers (via Docker Compose)
- Test the full HTTP request/response cycle
- Verify database state mutations
- Extend `BaseEntityIT<E, C>` for entities or `BaseServiceIT` for services

```java
class TableResourceIT extends BaseEntityIT<Table, CreateTableRequest> {
  // COLLECTION_PATH, FIELDS, createEntity, getEntity, updateEntity implemented
}
```

CI runs integration tests in `.github/workflows/integration-tests-mysql-elasticsearch.yml` and `.github/workflows/integration-tests-postgres-opensearch.yml`.

### 2.5 Enum Backward Compatibility Tests

A dedicated test class validates enum ordinal stability:

```java
class EnumBackwardCompatibilityTest {
  @Test
  void testRelationshipEnumBackwardCompatible() {
    assertEquals(25, Relationship.values().length);
    assertEquals(24, Relationship.OUTPUT_PORT.ordinal());
  }
}
```

New enum values must be added at the **end** of an enum to preserve database compatibility. This test will fail if a value is inserted in the middle.

### 2.6 Test Coverage Target

- **90% line coverage** on changed classes (enforced in `/test-enforcement` skill)
- Coverage is collected by JaCoCo during `mvn test` and reported to SonarCloud
- Run coverage locally: `mvn test -Pstatic-code-analysis`

### 2.7 Test Naming & Structure

| Pattern | Description |
|---------|-------------|
| `*Test.java` | Unit tests (no external dependencies) |
| `*IT.java` | Integration tests (real infrastructure) |
| `*E2ETest.java` | End-to-end tests (full system) |
| `*BenchTest.java` | Benchmark/performance tests |

---

## 3. Python Ingestion Testing

**Reference:** `ingestion/pyproject.toml`, `ingestion/Makefile`, `ingestion/noxfile.py`

### 3.1 Frameworks & Tools

| Purpose | Tool |
|---------|------|
| Unit tests | pytest |
| Parallel test execution | pytest-xdist |
| Coverage | coverage.py + pytest-cov |
| Type checking | basedpyright |
| Linting | pylint, black, isort, pycln |
| Mocking | `unittest.mock` (MagicMock, patch) |
| Property-based testing | hypothesis |
| E2E testing | Playwright (via pytest-playwright) |

### 3.2 Test Directory Structure

```
ingestion/tests/
├── conftest.py                    # Session fixtures (auto-discovery)
├── unit/
│   ├── airflow/
│   ├── bulksink/
│   ├── clients/
│   ├── connections/
│   ├── great_expectations/
│   ├── lineage/
│   │   └── masker/
│   ├── metadata/
│   │   ├── cli/
│   │   ├── common/
│   │   ├── data_quality/
│   │   ├── ingestion/
│   │   │   ├── connections/
│   │   │   ├── models/
│   │   │   └── ometa/
│   │   ├── pii/
│   │   ├── profiler/
│   │   └── utils/
│   ├── models/
│   ├── observability/
│   │   ├── data_quality/
│   │   └── profiler/
│   └── topology/                   # (currently ignored)
├── integration/
│   ├── ometa/
│   ├── postgres/
│   ├── mysql/
│   ├── profiler/
│   └── data_quality/
└── e2e/
    ├── artifacts/                 # Screenshots on failure
    └── test_*.py
```

### 3.3 conftest.py — Required Fixtures

`ingestion/tests/unit/conftest.py` is loaded automatically and provides:

- **`worker_id` fixture** — supports pytest-xdist worker identification
- **`register_sqlite_math_functions`** — registers `math.sqrt` for SQLite tests (session-scoped, autouse)
- **`_mock_validate`** — patches `OpenMetadata.validate_versions` (auto)
- **`_mock_health`** — patches `OpenMetadata.health_check` (auto)

### 3.4 Pytest Configuration

Configured in `ingestion/pyproject.toml`:

```toml
[tool.pytest.ini_options]
markers = [
    "slow: marks tests as slow (deselect with '-m \"not slow\"')",
    "xdist_group(name): group tests to run on the same xdist worker"
]
addopts = "--ignore=ingestion/tests/unit/topology/database/test_deltalake.py --ignore=ingestion/tests/integration/sources/mlmodels/mlflow/test_mlflow.py"
```

### 3.5 Running Tests

```bash
# Unit tests (fast)
cd ingestion
make unit_ingestion

# With nox (parallel, coverage)
nox -s unit-tests

# Static checks
make static-checks    # basedpyright
make lint             # pylint

# Integration tests
nox -s integration-tests -- --standalone

# All tests with coverage
nox -s unit-tests && nox -s integration-tests -- --standalone
nox -s combine-coverage
```

### 3.6 Python Test Style

Follow pytest style — plain `assert` statements, no `unittest.TestCase`:

```python
# CORRECT — pytest style
class TestEntityLink:
    def test_parse_simple(self):
        result = EntityLink.parse("..")
        assert result.entity_type == "table"

# WRONG — unittest style
class TestEntityLink(TestCase):
    def test_parse_simple(self):
        self.assertEqual(result.entity_type, "table")
```

Test classes should not inherit from `TestCase`. Use plain classes prefixed with `Test`:
- `class TestCustomBasemodelValidation`
- `class TestConnectionBuilder`

### 3.7 Mocking Patterns

```python
from unittest.mock import patch, MagicMock

def test_with_mock():
    with patch("metadata.ingestion.ometa.ometa_api.OpenMetadata.health_check"):
        client = create_client()
        result = client.get_entity("table", "uuid")
        assert result is not None
```

### 3.8 Coverage Configuration

```toml
[tool.coverage.run]
source = ["metadata"]
relative_files = true
branch = true

[tool.coverage.report]
omit = [
    "*__init__*",
    "*/generated/*",
    "tests/*",
    "*/src/metadata/ingestion/source/database/sample_*"
]
```

Target: **90% line coverage** on changed modules.

---

## 4. TypeScript/React Testing

**Reference:** `openmetadata-ui/src/main/resources/ui/package.json`, `jest.config.js`

### 4.1 Frameworks & Tools

| Purpose | Tool | Version |
|---------|------|---------|
| Unit testing | Jest | 29+ |
| React Testing | @testing-library/react | 14+ |
| User Event | @testing-library/user-event | |
| Test environment | jsdom | |
| E2E testing | Playwright | 1.57 |
| Coverage | Jest built-in + jest-sonar-reporter | |

### 4.2 Test Directory Structure

```
openmetadata-ui/src/main/resources/ui/
├── src/
│   ├── test/
│   │   └── unit/
│   │       ├── coverage/              # Generated coverage reports
│   │       └── mocks/                 # Jest mocks (SVG, JSON, etc.)
│   ├── setupTests.js                  # Jest setup (auto-mocked modules)
│   └── (source files)
└── playwright/
    ├── test-data/
    ├── e2e/
    ├── doc-generator/
    └── output/
```

Jest test files are auto-discovered: `src/**/*.test.{ts,tsx,js,jsx}` and `src/**/*.spec.{ts,tsx}`.

### 4.3 Jest Configuration

Key settings in `jest.config.js`:
- **`testEnvironment: jsdom`** — browser-like DOM for React testing
- **`preset: ts-jest`** — TypeScript support
- **`transformIgnorePatterns`** — modules to transpile from node_modules
- **`clearMocks: true`** — auto-clear mocks between tests
- **`fakeTimers: { enableGlobally: true }`** — fake timers enabled globally
- **`testResultsProcessor: 'jest-sonar-reporter'`** — SonarCloud integration

### 4.4 React Testing Library Patterns

```tsx
import { render, screen } from '@testing-library/react';
import userEvent from '@testing-library/user-event';
import { MyComponent } from './MyComponent';

describe('MyComponent', () => {
  it('should render the submit button', () => {
    render(<MyComponent />);
    expect(screen.getByRole('button', { name: /submit/i })).toBeInTheDocument();
  });

  it('should call onSubmit when clicked', async () => {
    const user = userEvent.setup();
    const onSubmit = vi.fn();
    render(<MyComponent onSubmit={onSubmit} />);
    await user.click(screen.getByRole('button', { name: /submit/i }));
    expect(onSubmit).toHaveBeenCalled();
  });
});
```

### 4.5 Running Tests

```bash
cd openmetadata-ui/src/main/resources/ui

yarn test                        # Run all tests (passWithNoTests)
yarn test:watch                 # Watch mode
yarn test:cov-summary           # Coverage with JSON summary (for SonarCloud)

# E2E / Playwright
yarn playwright:run             # Run E2E tests
yarn playwright:open            # Open in UI mode
yarn playwright:codegen         # Generate test code
```

### 4.6 Coverage

Coverage is collected by Jest in `src/test/unit/coverage/` and submitted to SonarCloud via `yarn-coverage.yml` CI workflow.

Excluded from coverage:
- `src/generated/*` — generated code
- `src/@types/*` — type definitions
- `src/enums/*` — enum files
- `src/interface/*` — interface stubs

### 4.7 Playwright E2E Rules

Playwright tests follow strict rules enforced by `eslint-plugin-playwright`:

```javascript
// Blocking (error) — zero existing violations, prevent regressions
'playwright/no-networkidle': 'error',       // Flaky — use web-first assertions
'playwright/no-page-pause': 'error',        // Remove page.pause() before commit
'playwright/no-focused-test': 'error',      // No .only on tests (blocks CI)

// Aspirational (warn) — existing violations to fix over time
'playwright/missing-playwright-await': 'warn',
'playwright/valid-expect': 'warn',
'playwright/no-wait-for-timeout': 'warn',
'playwright/no-force-option': 'warn',
'playwright/no-element-handle': 'warn',
'playwright/prefer-web-first-assertions': 'warn',
```

---

## 5. CI Test Infrastructure

### 5.1 GitHub Actions Workflows

| Workflow | Trigger | Purpose |
|----------|---------|---------|
| `openmetadata-service-unit-tests.yml` | push/PR | Java unit tests + Surefire |
| `integration-tests-mysql-elasticsearch.yml` | nightly/schedule | Full Java IT suite |
| `py-tests.yml` | PR | Python unit + integration tests |
| `yarn-coverage.yml` | PR | Jest coverage → SonarCloud |
| `playwright-mysql-e2e.yml` | nightly | Full E2E suite (MySQL) |
| `playwright-postgresql-e2e.yml` | nightly | Full E2E suite (Postgres) |
| `java-checkstyle.yml` | push/PR | `mvn spotless:apply` |
| `py-checkstyle.yml` | PR | `make py_format_check` |
| `ui-checkstyle.yml` | PR | ESLint + Prettier + i18n sync |
| `maven-sonar-build.yml` | schedule | Full SonarCloud scan |

### 5.2 Test Database Containers

Integration tests use Docker containers via Testcontainers or Docker Compose:
- **MySQL** (default) or **PostgreSQL** — database
- **Elasticsearch 7.x** or **OpenSearch 2.x** — search
- **Docker services:** `docker/development/docker-compose.yml`

### 5.3 SonarCloud Integration

- **Java:** Coverage from Maven Surefire → SonarCloud via `maven-sonar-build.yml`
- **Python:** Coverage combined from unit + integration → SonarCloud via `py-tests.yml`
- **TypeScript:** Jest coverage via `yarn-coverage.yml`

---

## 6. Coverage Targets

| Layer | Target | Tool |
|-------|--------|------|
| Java backend | **90% line coverage** on changed classes | JaCoCo + SonarCloud |
| Python ingestion | **90% line coverage** on changed modules | coverage.py + SonarCloud |
| TypeScript/React | **80% line coverage** (default) | Jest + SonarCloud |

Coverage enforcement is part of the `/test-enforcement` skill gate.

---

## 7. Test Directory Conventions by Language

```
.
├── openmetadata-service/src/test/java/   # Java unit tests
├── openmetadata-integration-tests/      # Java integration tests
├── ingestion/tests/
│   ├── unit/                            # Python unit tests (pytest)
│   ├── integration/                     # Python integration tests
│   └── e2e/                             # Python E2E tests
└── openmetadata-ui/src/main/resources/ui/
    ├── src/test/unit/                   # Jest unit tests
    └── playwright/                      # Playwright E2E tests
```

All test files must be colocated with or near their source modules. Use the same package/directory structure as the source code.
