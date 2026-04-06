# OpenMetadata Code Conventions

This document captures the coding standards, style rules, and architectural patterns enforced across the OpenMetadata codebase. All contributors must adhere to these conventions. CI will reject non-conforming PRs.

---

## 1. General Principles

### 1.1 Three-Language Monorepo

OpenMetadata is a polyglot project with three primary language layers:

| Layer | Language | Version | Build/Test Tool |
|-------|----------|---------|----------------|
| Backend | Java | 21 | Maven (`mvn`) |
| Frontend | TypeScript/React | Node 22+ | Yarn |
| Ingestion | Python | 3.10–3.12 | pytest/nox |

### 1.2 Schema-First Architecture

All entity definitions originate as JSON Schema files in `openmetadata-spec/src/main/resources/json/schema/`. Code generation produces:
- **Java POJOs** via jsonschema2pojo
- **Python models** via datamodel-code-generator
- **TypeScript interfaces** via QuickType
- **UI form schemas** via `parseSchemas.js`

Never hand-write types that should be generated. Modify schemas first, then regenerate.

### 1.3 License Header

Every source file must carry the Apache 2.0 license header. Run `make license-header-fix` (UI) or `mvn spotless:apply` (Java) to add/verify headers.

---

## 2. Java Backend Conventions

**Reference:** `pom.xml`, `DEVELOPER.md` §Java Backend, `CLAUDE.md` §Java Code Requirements

### 2.1 Formatting & Style

- **Formatter:** Spotless with Google Java Format (AOSP style)
  - Run: `mvn spotless:apply`
  - Check: `mvn spotless:check` (CI gate)
- **Checkstyle:** Custom rules in `checkstyle/checkstyle.xml`
- **Java Version:** 21 (enforced by `maven-compiler-plugin`)
- **Imports:** No wildcard imports. No fully-qualified type names in code — use proper `import` statements.

### 2.2 Method Complexity (Kafka-Grade Standards)

These are enforced in code review and by the `/code-review` skill:

| Rule | Limit |
|------|-------|
| Method lines (excl. blank lines, braces) | **15 max** |
| Cyclomatic complexity | **10 max** |
| Nesting depth | **3 max** |
| Parameters | **5 max** |
| Class size | **500 lines** (1000 = design problem) |

Use early returns to flatten nesting:

```java
// BAD — deep nesting
if (entity != null) {
  if (entity.isActive()) {
    if (hasPermission(entity)) {
      process(entity);
    }
  }
}

// GOOD — early returns, flat
if (entity == null) return;
if (!entity.isActive()) return;
if (!hasPermission(entity)) return;
process(entity);
```

### 2.3 Naming Conventions

| Element | Convention | Example |
|---------|-----------|---------|
| Methods | Verb phrase, camelCase | `calculateScore()`, `findByName()` |
| Booleans | Question form | `isActive`, `hasPermission`, `canRetry` |
| Variables | Descriptive, no abbreviations | `entityReference` not `er` |
| Constants | `UPPER_SNAKE_CASE` | `MAX_RETRY_COUNT`, `DEFAULT_PAGE_SIZE` |
| Classes | PascalCase | `EntityRepository`, `TableResource` |
| Package | lowercase | `org.openmetadata.service.resources` |

### 2.4 Magic Strings — Define Constants

Never use raw string literals in `.equals()`, `.contains()`, or `switch` cases. Use `Entity` constants or define your own:

```java
// BAD
if (taskStatus.equals("Open")) { ... }
if (config.getResources().get(0).equals("all")) { ... }

// GOOD
if (taskStatus == TaskStatus.OPEN) { ... }
private static final String RESOURCE_ALL = "all";
```

### 2.5 Error Handling

- **No empty catch blocks** — at minimum, log the exception
- **No `catch (Exception e)`** — catch the specific expected type
- **No `e.printStackTrace()`** — use the logger
- **Error messages must include context:** `"Table '%s' not found in database '%s'"`, not `"Not found"`
- **No `throw` or `return` inside `finally` blocks** — masks the original exception
- **No exceptions for flow control** — use conditionals for expected cases

### 2.6 No Convoluted if/else Chains

More than 3 `else if` branches means the structure is wrong. Refactor using:
- `switch` expression with pattern matching (Java 21) for `instanceof`
- `switch` expression for enum values
- `Map` dispatch for `.equals("string")` lookups

```java
// BAD
if (type.equals("table")) { return new TableHandler(); }
else if (type.equals("dashboard")) { return new DashboardHandler(); }
else if (type.equals("pipeline")) { return new PipelineHandler(); }

// GOOD — Map dispatch
private static final Map<String, EntityHandler> HANDLERS =
    Map.of("table", new TableHandler(),
           "dashboard", new DashboardHandler(),
           "pipeline", new PipelineHandler());
```

### 2.7 Immutability

- Use `final` on local variables and parameters
- Use `final` on fields set in the constructor
- Return `Collections.unmodifiableList()` / `List.copyOf()` from public methods
- Utility classes must be `final` with a private constructor
- Prefer `record` for immutable data carriers

### 2.8 Modern Java (21) Idioms

- Try-with-resources for all `AutoCloseable` objects
- Diamond operator `<>` — `new ArrayList<>()` not `new ArrayList<String>()`
- Pattern matching: `if (obj instanceof String s)` instead of cast
- Switch expressions instead of `if/else if` chains on enums
- `List.of()`, `Map.of()`, `Set.of()` for immutable collection literals
- Text blocks `"""` for multi-line strings

### 2.9 REST Resource Pattern

All entity REST resources extend `EntityResource<E, R extends EntityRepository<E>>`. See `DEVELOPER.md` §REST Resource Pattern for the full template.

Key conventions:
- `COLLECTION_PATH` constant for the URL path
- `FIELDS` constant for supported field queries
- Inner `*List extends ResultList<E>` class for list serialization
- All methods receive `@Context SecurityContext` and `@Context UriInfo`

### 2.10 Common Bug Patterns to Avoid

- `equals()` without `hashCode()` (or vice versa)
- `equals()` on arrays — use `Arrays.equals()`
- Ignoring return values of `String.replace()`, `File.delete()`
- `collection.size() == 0` — use `collection.isEmpty()`
- String concatenation inside loops — use `StringBuilder`
- `synchronized` on non-final fields
- `toLowerCase()` without `Locale` — always use `toLowerCase(Locale.ROOT)`
- Double map lookups — use `computeIfAbsent()` or `getOrDefault()`

---

## 3. Python Ingestion Conventions

**Reference:** `ingestion/pyproject.toml`, `ingestion/Makefile`, `CLAUDE.md` §Python Ingestion Development, `DEVELOPER.md` §Python Ingestion

### 3.1 Formatting & Style

| Tool | Purpose | Config |
|------|---------|--------|
| **Black** | Code formatting (max-line-length: 120) | `pyproject.toml` |
| **isort** | Import sorting (profile: black) | `pyproject.toml` |
| **pycln** | Remove unused imports | `pyproject.toml` |
| **PyLint** | Code quality (fail-under: 6.0) | `.pylintrc` + `pyproject.toml` |
| **basedpyright** | Type checking | `pyproject.toml` |

Commands:
```bash
make py_format        # Format code (black + isort + pycln)
make py_format_check  # Check formatting (CI gate)
make lint             # Run pylint
make static-checks    # Run basedpyright type checking
```

### 3.2 PyLint Disabled Rules

The following rules are intentionally disabled (see `.pylintrc`):
- `W1203` / `W1202` — f-string/lazy formatting (use f-strings for readability)
- `R0903` — too-few-public-methods (false negatives with Pydantic)
- `W0707` — raise-missing-from (exception encapsulation)
- `R0901` — too-many-ancestors (SQA class inheritance)
- `W0703` — broad-except (external source systems)
- `W0511` — fixme (internal notes)
- `W1518` — method-cache-max-size-none (LRU with maxsize None)

### 3.3 Naming & Structure

- **Module naming:** `(([a-z_][a-z0-9_]*)|([a-zA-Z0-9]+))$` (configured in pylint)
- **Max args:** 7 | **Max attributes:** 12 | **Max public methods:** 25
- **Type variable names:** `T, C, fn, db, df, i`
- **Generated code** (`src/metadata/generated/`) is excluded from linting and type checking

### 3.4 Connector Organization

Keep connector-specific logic in the connector's directory. Never add connector-specific edge cases to shared utility files:

```
# CORRECT
ingestion/src/metadata/ingestion/source/database/redshift/connection.py  # Redshift-specific IAM auth

# WRONG — clutters shared builder
ingestion/src/metadata/ingestion/connections/builders.py  # Generic/shared
```

### 3.5 Topology Pattern

All connectors use the declarative topology pattern (`ServiceSpec` + `TopologyNode`). See `DEVELOPER.md` §Python Ingestion Topology for the full pattern.

### 3.6 Python Testing Style

- **Framework:** pytest — plain `assert` statements, no `unittest.TestCase`
- **Fixtures:** Use pytest fixtures, not `setUp`/`tearDown`
- **Mocking:** `unittest.mock` (MagicMock, patch) — compatible with pytest
- **No `Thread.sleep()`** — use condition-based waiting or `Awaitility`

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

---

## 4. TypeScript/React Frontend Conventions

**Reference:** `openmetadata-ui/src/main/resources/ui/eslint.config.mjs`, `package.json`, `CLAUDE.md` §Frontend Architecture Patterns

### 4.1 Formatting & Style

| Tool | Purpose | Config |
|------|---------|--------|
| **ESLint 9** (flat config) | Linting | `eslint.config.mjs` |
| **Prettier 2.8** | Formatting (2-space, single quotes) | `.prettierrc.yaml` |
| **organize-imports-cli** | Import sorting | package.json script |
| **TypeScript 5** | Type checking | `tsconfig.json` |

Commands:
```bash
yarn lint:fix            # ESLint auto-fix
yarn pretty             # Prettier format all
yarn organize-imports   # Sort imports
yarn tsc:check          # TypeScript type check (noEmit)
make ui-checkstyle      # All checks: imports + lint + pretty + license + i18n + docs
```

### 4.2 ESLint Rules (Key Enforced Rules)

```javascript
eqeqeq: ['error', 'smart']           // Use === not == (smart allows null checks)
no-console: 'error'                   // No console.log/warn/error
max-len: [error, { code: 200 }]       // 200 chars per line
react/jsx-boolean-value: ['error', 'never']  // <Checkbox checked /> not checked={true}
react/self-closing-comp: 'error'      // <Div /> not <Div></Div>
jest/consistent-test-it: 'error'      // Use it() consistently in tests
```

### 4.3 TypeScript Rules

- **Never use `any`** — use proper types or `unknown` with type guards
- **`@typescript-eslint/no-explicit-any`: warn** (not error — exists to track)
- **`@typescript-eslint/no-unused-vars`**: error with `argsIgnorePattern: '^_'` and `varsIgnorePattern: '^_'`
- Import types from `generated/` for API response types — never hand-write interfaces for schema-defined types

### 4.4 React Component Patterns

- **File naming:** `ComponentName.component.tsx`, interfaces `ComponentName.interface.ts`
- **State:** `useState` with proper typing, avoid `any`
- **Side effects:** `useEffect` with proper dependency arrays
- **Performance:** `useCallback` for event handlers, `useMemo` for expensive computations
- **Custom hooks:** Prefix with `use`, place in `src/hooks/`, return typed objects
- **Loading states:** `useState<Record<string, boolean>>({})` for multiple loading states
- **Error handling:** `showErrorToast` / `showSuccessToast` from ToastUtils

### 4.5 i18n — No String Literals in JSX

All user-facing strings must use `useTranslation()`:

```tsx
// WRONG
<Button>Run</Button>

// CORRECT
const { t } = useTranslation();
<Button>{t('label.run')}</Button>
```

Keys live in `src/locale/languages/en-us.json` with kebab-case names:
```json
{
  "label": {
    "run": "Run",
    "add-entity": "Add {{entity}}"
  }
}
```

Locale files must have keys in **alphabetical order** — enforced by `jsonc/sort-keys` ESLint rule on `src/locale/**/*.json`.

### 4.6 Styling — Component Library & Tailwind

- **Use `openmetadata-ui-core-components`** for all new UI work. Do not use MUI.
- **Tailwind classes:** Must use `tw:` prefix to avoid Ant Design/Less conflicts (e.g., `tw:flex`, `tw:text-sm`)
- **Design tokens:** CSS custom properties — `--color-text-primary`, `--color-bg-secondary`, etc. Never hardcoded colors.
- **No new MUI imports** — actively removing MUI from codebase.

### 4.7 Import Organization

Imports must be ordered (enforced by `organize-imports-cli`):
1. External libraries (React, etc.)
2. Internal absolute imports (`generated/`, `constants/`, `hooks/`, etc.)
3. Relative imports for utilities and components
4. Asset imports (SVGs, styles)
5. Type imports grouped separately when needed

### 4.8 Test File Naming

Jest test files follow: `src/**/*.test.{ts,tsx,js,jsx}` (also `*.spec.{ts,tsx}`)

---

## 5. Documentation Standards

- **No unnecessary comments** — write self-documenting code
- **Comments only for:** Complex business logic, non-obvious algorithms/workarounds, public API JavaDoc, TODO/FIXME with ticket references
- **BAD examples:**
  - `// Create user` before `createUser()`
  - `// Verify domain is set` before `assertNotNull(entity.getDomain())`
- If the code needs a comment to be understood, refactor the code to be clearer instead

---

## 6. CI Checkstyle Enforcement

All checkstyle rules are enforced in CI on every PR. PRs that fail checks will not merge.

| Language | CI Workflow | Check Command |
|----------|-------------|---------------|
| Java | `java-checkstyle.yml` | `mvn spotless:apply` |
| Python | `py-checkstyle.yml` | `make py_format` + `make py_format_check` |
| TypeScript/UI | `ui-checkstyle.yml` | `make ui-checkstyle` |
| Jest | `yarn-coverage.yml` | `yarn test:cov-summary` |

All checkstyle commands are available as Makefile targets. Run the relevant `make` command before submitting your PR to match CI results locally.
