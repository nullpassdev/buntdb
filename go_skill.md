---
skill:
  name: "golang-expert"
  version: "1.0.1"
  description: >
    Expert-level Go (Golang) software engineering skill. Guides the agent to
    produce idiomatic, production-grade Go code following community standards,
    the official Go proverbs, and battle-tested architectural patterns.
  tags:
    - golang
    - backend
    - systems
    - microservices
    - cloud-native
pandorian-metadata:
  id: golang-expert
  language: go
  title: Golang Expert
  category: Code Quality & Maintainability
  is_active: true
  tags: [golang, backend, systems, microservices, cloud-native, best-practice, testing]
---

# ─────────────────────────────────────────────────────────────────────────────
# 1. CORE PHILOSOPHY
# ─────────────────────────────────────────────────────────────────────────────
philosophy:
  principles:
    - "Clear is better than clever."
    - "A little copying is far better than a little dependency."
    - "The bigger the interface, the weaker the abstraction."
    - "Make the zero value useful."
    - "Don't panic."
    - "Errors are values — handle them, don't hide them."
    - "Accept interfaces, return structs."
    - "Write code for the next reader, not the compiler."
    - "Composition over inheritance — always."
    - "Design for testability from day one."

  anti_patterns:
    - "Never use init() for business logic; reserve it only for registering drivers or codecs."
    - "Never use package-level mutable state."
    - "Never swallow errors with _ unless explicitly justified in a comment."
    - "Avoid god structs — if a struct has >7 fields, reconsider its responsibility."
    - "Avoid premature abstraction — wait until you have 3 concrete cases before extracting an interface."
    - "Never use reflect or unsafe in application code without extraordinary justification."
    - "Avoid global singletons — pass dependencies explicitly."

# ─────────────────────────────────────────────────────────────────────────────
# 2. PROJECT STRUCTURE
# ─────────────────────────────────────────────────────────────────────────────
project_structure:
  description: >
    Follow the community-standard Go project layout. Avoid the /pkg directory
    unless the project is a large shared library. Keep the top-level clean.

  layout: |
    repo-root/
    ├── cmd/                        # Entry points — one sub-dir per binary
    │   └── <app-name>/
    │       └── main.go             # Minimal: parse flags, wire deps, call run()
    │
    ├── internal/                   # Private application code (import-restricted)
    │   ├── domain/                 # Core business types, interfaces, and value objects
    │   │   ├── model.go            # Domain entities and aggregates
    │   │   ├── errors.go           # Domain-specific sentinel errors
    │   │   └── repository.go       # Repository interface definitions
    │   │
    │   ├── service/                # Business logic / use-case orchestration
    │   │   ├── order.go
    │   │   └── order_test.go
    │   │
    │   ├── adapter/                # Driven adapters (implementations of domain interfaces)
    │   │   ├── postgres/           # e.g. PostgreSQL repository implementation
    │   │   │   ├── order_repo.go
    │   │   │   └── order_repo_test.go
    │   │   ├── dynamo/             # e.g. DynamoDB implementation
    │   │   └── sqs/               # e.g. SQS publisher
    │   │
    │   ├── handler/                # Driving adapters (HTTP handlers, gRPC servers, CLI)
    │   │   ├── http/
    │   │   │   ├── router.go
    │   │   │   ├── order_handler.go
    │   │   │   └── middleware.go
    │   │   └── grpc/
    │   │
    │   ├── platform/               # Cross-cutting infra: logging, tracing, config
    │   │   ├── config/
    │   │   │   └── config.go       # Structured config loading (env / file)
    │   │   ├── logger/
    │   │   ├── observability/
    │   │   └── shutdown/           # Graceful shutdown orchestration
    │   │
    │   └── wire/                   # Dependency injection / wiring (manual or google/wire)
    │       └── wire.go
    │
    ├── api/                        # API contracts: OpenAPI specs, proto files, JSON schemas
    │   ├── openapi/
    │   └── proto/
    │
    ├── migrations/                 # Database migration files (SQL, Liquibase, etc.)
    │
    ├── scripts/                    # Dev and CI helper scripts
    │
    ├── testdata/                   # Golden files and fixtures (go test ignores this name)
    │
    ├── .golangci.yml               # Linter config
    ├── Dockerfile
    ├── Makefile
    ├── go.mod
    ├── go.sum
    └── README.md

  rules:
    - "main.go must be tiny: parse config, build the dependency graph, start the server, and block on shutdown. No business logic."
    - "All application code lives under internal/ to enforce encapsulation at the module level."
    - "Domain types (interfaces, entities, value objects, errors) must have zero external dependencies."
    - "Adapter packages implement domain interfaces; they may import domain — never the reverse."
    - "Handler packages translate HTTP/gRPC into service calls; they must not contain business logic."
    - "Never import a child package from a parent package — dependencies flow inward."

# ─────────────────────────────────────────────────────────────────────────────
# 3. CODING STANDARDS
# ─────────────────────────────────────────────────────────────────────────────
coding_standards:

  naming:
    - "Use MixedCaps / mixedCaps. Never use snake_case for Go identifiers (except in test names: Test_parseToken_expired)."
    - "Acronyms are all-caps: HTTPClient, userID, xmlParser."
    - "Single-method interfaces use the -er suffix: Reader, Notifier, Storer."
    - "Avoid stutter: use queue.Item, not queue.QueueItem."
    - "Receivers: use 1-2 letter abbreviations consistent within a type (s for Service, r for Repo). Never use 'this' or 'self'."
    - "Unexported helpers start lowercase and stay in the same file as their caller when small."
    - "Package names are singular, lowercase, one word when possible: order, not orders or orderPkg."

  error_handling:
    description: "Errors are first-class citizens. Treat them as carefully as the happy path."
    rules:
      - "Always return error as the last return value."
      - "Wrap errors with context using fmt.Errorf(\"verb noun: %w\", err) to build an error chain."
      - "Define sentinel errors as package-level vars: var ErrNotFound = errors.New(\"not found\")."
      - "Use errors.Is() and errors.As() for comparison — never == except against nil."
      - "Custom error types must implement Error() string and optionally Unwrap() error."
      - "In HTTP handlers, map domain errors to status codes at the handler boundary — never inside services."
      - "Log the error once at the point of handling; do not log-and-return (this produces duplicate logs)."
      - |
        Use structured error types for rich context when needed:
          type ValidationError struct {
              Field   string
              Message string
          }
          func (e *ValidationError) Error() string {
              return fmt.Sprintf("validation: %s — %s", e.Field, e.Message)
          }

  context_usage:
    - "Pass context.Context as the first parameter of every function that does I/O or may be cancelled."
    - "Never store context in a struct field."
    - "Use context for cancellation, deadlines, and request-scoped values only."
    - "Create context values with unexported key types to avoid collisions: type ctxKey struct{}."
    - "Always respect ctx.Done() in long-running loops and goroutines."

  concurrency:
    - "Start goroutines only when you have a clear ownership and shutdown plan."
    - "Prefer errgroup.Group for fan-out/fan-in with error propagation."
    - "Always use sync.WaitGroup or errgroup to join goroutines before returning from the owning function."
    - "Channels are for communication; mutexes are for protecting state. Choose based on intent."
    - "Buffered channels must have a justified, documented buffer size."
    - "Never leak goroutines — every goroutine must have a termination signal (context, done channel, or bounded work)."
    - "Use sync.Once for lazy, thread-safe initialization."

  dependency_injection:
    description: "Explicit dependency injection via constructors. No service locators, no globals."
    rules:
      - |
        Constructor pattern:
          func NewOrderService(repo domain.OrderRepository, pub domain.EventPublisher, log *slog.Logger) *OrderService {
              return &OrderService{repo: repo, pub: pub, log: log}
          }
      - "Accept interfaces in constructors, store as interface fields, return concrete pointer."
      - "Use functional options (func(*Config)) for optional/advanced configuration."
      - "For large applications, consider google/wire for compile-time DI — avoid runtime reflection-based DI."
      - "Test doubles are passed through the same constructor — no special test hooks."

  functional_options:
    description: "Use the functional options pattern for flexible, backwards-compatible APIs."
    example: |
      type Option func(*Server)

      func WithPort(port int) Option {
          return func(s *Server) { s.port = port }
      }

      func WithLogger(log *slog.Logger) Option {
          return func(s *Server) { s.log = log }
      }

      func NewServer(handler http.Handler, opts ...Option) *Server {
          s := &Server{
              handler: handler,
              port:    8080,
              log:     slog.Default(),
          }
          for _, opt := range opts {
              opt(s)
          }
          return s
      }

  logging:
    - "Use log/slog (standard library, Go 1.21+). Avoid third-party loggers unless the project predates slog."
    - "Always use structured logging: slog.Info(\"order created\", \"order_id\", id, \"user_id\", uid)."
    - "Log levels: Debug for dev-only detail, Info for business events, Warn for recoverable anomalies, Error for failures requiring attention."
    - "Never log sensitive data: passwords, tokens, PII."
    - "Inject *slog.Logger into structs — never use the global default in library or service code."

  configuration:
    - "Load config once at startup into a strongly-typed struct."
    - "Support environment variables as the primary source (12-factor)."
    - "Use struct tags for env mapping: `env:\"DATABASE_URL,required\"`."
    - "Validate config at startup — fail fast with clear error messages."
    - "Never read os.Getenv() deep inside business logic."

# ─────────────────────────────────────────────────────────────────────────────
# 4. INTERFACES & API DESIGN
# ─────────────────────────────────────────────────────────────────────────────
api_design:

  interfaces:
    - "Define interfaces where they are consumed, not where they are implemented."
    - "Keep interfaces small: 1-3 methods is ideal."
    - |
      Standard domain interface example:
        // In internal/domain/repository.go
        type OrderRepository interface {
            Get(ctx context.Context, id string) (*Order, error)
            Save(ctx context.Context, order *Order) error
            List(ctx context.Context, filter OrderFilter) ([]Order, error)
        }
    - "Never export an interface solely for mocking — consumer-side interfaces solve this automatically."

  http_apis:
    - "Use a thin router (stdlib http.ServeMux for Go 1.22+, or chi). Avoid heavy frameworks."
    - |
      Handler signature pattern — always return an error for central handling:
        func (h *OrderHandler) Create(w http.ResponseWriter, r *http.Request) {
            var req CreateOrderRequest
            if err := json.NewDecoder(r.Body).Decode(&req); err != nil {
                h.respondError(w, r, fmt.Errorf("decode request: %w", err), http.StatusBadRequest)
                return
            }
            order, err := h.service.Create(r.Context(), req.toDomain())
            if err != nil {
                h.respondError(w, r, err, h.mapError(err))
                return
            }
            h.respondJSON(w, r, http.StatusCreated, toOrderResponse(order))
        }
    - "Use middleware for cross-cutting: auth, request ID injection, panic recovery, structured access logging."
    - "Respond with consistent JSON envelope: { \"data\": ..., \"error\": ... }."
    - "Always set Content-Type before writing the status code."

  grpc:
    - "Define service contracts in .proto files under api/proto/."
    - "Generate Go code into a dedicated gen/ or api/ package — never edit generated code."
    - "Implement service interfaces in internal/handler/grpc/."
    - "Use interceptors for logging, auth, and recovery — mirror the HTTP middleware approach."

# ─────────────────────────────────────────────────────────────────────────────
# 5. TESTING
# ─────────────────────────────────────────────────────────────────────────────
testing:

  philosophy:
    - "Tests are first-class production code. Apply the same quality standards."
    - "Test behavior, not implementation. If you refactor internals and tests break, the tests were wrong."
    - "The test pyramid: many unit tests, some integration tests, few end-to-end tests."
    - "Every exported function in service/ and adapter/ must have test coverage."

  unit_tests:
    - "Place tests in the same package for white-box access (order_test.go alongside order.go)."
    - "Use _test package suffix (order_test) only when you want to test the public API as a consumer would."
    - |
      Table-driven tests are the default pattern:
        func TestParseAmount(t *testing.T) {
            tests := []struct {
                name    string
                input   string
                want    int64
                wantErr bool
            }{
                {name: "valid cents",    input: "12.34", want: 1234, wantErr: false},
                {name: "negative",       input: "-5.00", want: -500, wantErr: false},
                {name: "empty string",   input: "",      want: 0,    wantErr: true},
                {name: "invalid format", input: "abc",   want: 0,    wantErr: true},
            }
            for _, tt := range tests {
                t.Run(tt.name, func(t *testing.T) {
                    got, err := ParseAmount(tt.input)
                    if (err != nil) != tt.wantErr {
                        t.Fatalf("ParseAmount(%q) error = %v, wantErr %v", tt.input, err, tt.wantErr)
                    }
                    if got != tt.want {
                        t.Errorf("ParseAmount(%q) = %d, want %d", tt.input, got, tt.want)
                    }
                })
            }
        }
    - "Use t.Helper() in every test helper function to get correct line numbers on failure."
    - "Use t.Cleanup() for teardown — not defer — so cleanup runs even if t.Fatal is called."
    - "Prefer stdlib testing + testify/assert for assertions. Avoid heavy test frameworks."
    - "Use t.Parallel() for independent tests to surface race conditions."

  mocks_and_fakes:
    - "Define mock/fake implementations in a _test.go file or a dedicated internal/mock/ package."
    - |
      Prefer hand-written fakes over generated mocks for domain interfaces:
        type fakeOrderRepo struct {
            orders map[string]*domain.Order
            err    error
        }
        func (f *fakeOrderRepo) Get(ctx context.Context, id string) (*domain.Order, error) {
            if f.err != nil {
                return nil, f.err
            }
            o, ok := f.orders[id]
            if !ok {
                return nil, domain.ErrNotFound
            }
            return o, nil
        }
    - "When fakes grow large or interfaces change often, use mockery or moq for generation."
    - "Never mock what you don't own — wrap third-party clients behind your own interface, then mock that."

  integration_tests:
    - "Use build tags or TestMain to gate integration tests: //go:build integration."
    - "Use testcontainers-go to spin up real Postgres, Redis, DynamoDB-local, SQS (via LocalStack) in tests."
    - |
      Pattern for testcontainers setup:
        func setupPostgres(t *testing.T) *pgxpool.Pool {
            t.Helper()
            ctx := context.Background()
            container, err := postgres.Run(ctx, "postgres:16-alpine",
                postgres.WithDatabase("testdb"),
            )
            require.NoError(t, err)
            t.Cleanup(func() { _ = container.Terminate(ctx) })
            connStr, err := container.ConnectionString(ctx, "sslmode=disable")
            require.NoError(t, err)
            pool, err := pgxpool.New(ctx, connStr)
            require.NoError(t, err)
            t.Cleanup(func() { pool.Close() })
            return pool
        }
    - "Run migrations in TestMain or a shared setup helper — never rely on pre-seeded databases."
    - "Each test gets its own isolated data context (separate schema, truncation, or transaction rollback)."

  golden_files:
    - "Store expected outputs in testdata/ and compare with golden file pattern."
    - "Use -update flag to regenerate: if *update { os.WriteFile(...) }."
    - "Golden files are committed to version control."

  test_organization:
    - "Name tests: Test<Function>_<scenario> or Test<Type>_<Method>_<scenario>."
    - "Group related subtests with t.Run() for clear failure output."
    - "Keep test files in the same directory as the code they test."
    - "Shared test fixtures go in testdata/ or internal/testutil/."

# ─────────────────────────────────────────────────────────────────────────────
# 6. DATABASE & PERSISTENCE
# ─────────────────────────────────────────────────────────────────────────────
database:

  general:
    - "Use pgx (jackc/pgx) for PostgreSQL — not database/sql directly."
    - "Use connection pooling (pgxpool.Pool) and pass the pool to repositories."
    - "Always use parameterized queries — never concatenate user input into SQL."
    - "Use context with all DB operations for cancellation support."

  transactions:
    - |
      Transaction helper pattern:
        func withTx(ctx context.Context, pool *pgxpool.Pool, fn func(tx pgx.Tx) error) error {
            tx, err := pool.Begin(ctx)
            if err != nil {
                return fmt.Errorf("begin tx: %w", err)
            }
            defer func() { _ = tx.Rollback(ctx) }()
            if err := fn(tx); err != nil {
                return err
            }
            return tx.Commit(ctx)
        }
    - "Keep transactions short — acquire, do work, commit. Never hold a tx across I/O to external services."
    - "For DynamoDB, use TransactWriteItems for multi-item atomicity and optimistic locking via version attributes."

  migrations:
    - "Use a dedicated migration tool (golang-migrate, goose, or Liquibase)."
    - "Migrations are numbered, idempotent, and checked into version control."
    - "Every migration must have a corresponding rollback (down) migration."
    - "Never modify a migration that has been applied to any shared environment."

# ─────────────────────────────────────────────────────────────────────────────
# 7. OBSERVABILITY
# ─────────────────────────────────────────────────────────────────────────────
observability:

  logging:
    - "See coding_standards.logging above."

  metrics:
    - "Use OpenTelemetry SDK for metrics (otel.Meter)."
    - "Instrument: request count, latency histograms, error rates, queue depth."
    - "Expose /metrics for Prometheus scraping if applicable."

  tracing:
    - "Use OpenTelemetry for distributed tracing."
    - "Create spans at service boundaries: HTTP middleware, DB calls, external API calls."
    - "Propagate trace context via context.Context — never via struct fields or globals."
    - "Add attributes to spans: order_id, user_id, operation name."

  health_checks:
    - "Expose /healthz (liveness) and /readyz (readiness) endpoints."
    - "/healthz returns 200 if the process is alive."
    - "/readyz checks downstream dependencies (DB, cache, message broker) and returns 503 if any are unhealthy."

# ─────────────────────────────────────────────────────────────────────────────
# 8. GRACEFUL SHUTDOWN
# ─────────────────────────────────────────────────────────────────────────────
graceful_shutdown:
  pattern: |
    func run(ctx context.Context, cfg Config) error {
        ctx, cancel := signal.NotifyContext(ctx, syscall.SIGINT, syscall.SIGTERM)
        defer cancel()

        // Build dependencies ...
        srv := &http.Server{Addr: cfg.Addr, Handler: router}

        g, gCtx := errgroup.WithContext(ctx)

        g.Go(func() error {
            log.Info("starting server", "addr", cfg.Addr)
            if err := srv.ListenAndServe(); err != nil && !errors.Is(err, http.ErrServerClosed) {
                return err
            }
            return nil
        })

        g.Go(func() error {
            <-gCtx.Done()
            log.Info("shutting down server")
            shutdownCtx, shutdownCancel := context.WithTimeout(context.Background(), 10*time.Second)
            defer shutdownCancel()
            return srv.Shutdown(shutdownCtx)
        })

        return g.Wait()
    }
  rules:
    - "Trap SIGINT and SIGTERM via signal.NotifyContext."
    - "Drain in-flight requests before exiting."
    - "Close database pools, flush telemetry, and ack pending messages in order."
    - "Set a hard shutdown timeout (e.g. 10s) to prevent hanging."

# ─────────────────────────────────────────────────────────────────────────────
# 9. BUILD, CI/CD & TOOLING
# ─────────────────────────────────────────────────────────────────────────────
tooling:

  linting:
    - "Use golangci-lint with a checked-in .golangci.yml config."
    - |
      Recommended linters to enable:
        errcheck, govet, staticcheck, unused, gosimple, ineffassign,
        revive, gofumpt, gocritic, misspell, prealloc, nolintlint,
        errorlint, exhaustive, noctx, bodyclose, sqlclosecheck
    - "Run linting in CI as a blocking step."

  formatting:
    - "Use gofumpt (strict superset of gofmt). Enforce in pre-commit and CI."

  build:
    - "Use a Makefile with standard targets: build, test, lint, run, migrate, docker-build."
    - "Inject version info via ldflags: -ldflags \"-X main.version=$(git describe --tags)\"."
    - "Use multi-stage Docker builds: build in golang:1.23-alpine, run in gcr.io/distroless/static-debian12."

  ci_pipeline:
    - "Steps: checkout → go mod download → lint → test (with race detector) → build → push image."
    - "Always run tests with -race flag: go test -race ./..."
    - "Cache go module downloads and build cache between CI runs."
    - "For private modules, use GOPRIVATE and GONOSUMDB with a GitHub PAT or SSH key."

  module_management:
    - "go.mod must list the minimum Go version the project supports."
    - "Run go mod tidy in CI and fail if it produces a diff."
    - "Pin major dependency versions. Review minor/patch bumps via Dependabot or Renovate."
    - "For monorepos with multiple modules, use Go workspace (go.work) for local development."

# ─────────────────────────────────────────────────────────────────────────────
# 10. DOCKER & DEPLOYMENT
# ─────────────────────────────────────────────────────────────────────────────
docker:
  dockerfile_pattern: |
    # Build stage
    FROM golang:1.23-alpine AS build
    WORKDIR /src
    COPY go.mod go.sum ./
    RUN go mod download
    COPY . .
    RUN CGO_ENABLED=0 GOOS=linux go build -ldflags="-s -w" -o /app ./cmd/<app-name>

    # Runtime stage
    FROM gcr.io/distroless/static-debian12
    COPY --from=build /app /app
    COPY --from=build /src/migrations /migrations
    USER nonroot:nonroot
    ENTRYPOINT ["/app"]
  rules:
    - "Always use multi-stage builds to minimize image size."
    - "Set CGO_ENABLED=0 for static binaries when not using CGO."
    - "Run as non-root user in production."
    - "Copy only the binary and necessary runtime assets (migrations, configs) into the final image."
    - "Use .dockerignore to exclude .git, vendor, testdata, docs."

# ─────────────────────────────────────────────────────────────────────────────
# 11. AWS & CLOUD-NATIVE PATTERNS
# ─────────────────────────────────────────────────────────────────────────────
aws_patterns:

  general:
    - "Use the official aws-sdk-go-v2. Never use v1 in new code."
    - "Load AWS config once at startup: cfg, err := config.LoadDefaultConfig(ctx)."
    - "Pass service clients (sqs.Client, dynamodb.Client) via dependency injection."
    - "Wrap AWS clients behind domain interfaces for testability."

  dynamodb:
    - "Use attributevalue.MarshalMap / UnmarshalMap with struct tags: `dynamodbav:\"pk\"`."
    - "Implement optimistic locking via a version attribute and ConditionExpression."
    - "Use TransactWriteItems for cross-item atomicity."
    - "Design single-table schemas with PK/SK patterns; document access patterns before coding."

  sqs:
    - "Use long polling (WaitTimeSeconds: 20) to reduce empty receives."
    - "Always delete messages after successful processing."
    - "Implement idempotent consumers — SQS guarantees at-least-once delivery."
    - "Use a dead-letter queue (DLQ) with maxReceiveCount for poison messages."

  localstack:
    - "Use LocalStack for local development and integration testing of SQS, SNS, DynamoDB, S3."
    - "Configure endpoint overrides via environment variables, not code-level if-else."

# ─────────────────────────────────────────────────────────────────────────────
# 12. CODE REVIEW CHECKLIST (for self-audit)
# ─────────────────────────────────────────────────────────────────────────────
code_review_checklist:
  - "Does every error get handled or explicitly ignored with a // comment?"
  - "Does every goroutine have a clear shutdown path?"
  - "Are all interfaces defined at the consumer site, not the implementation site?"
  - "Is context.Context the first param of all I/O functions?"
  - "Are there tests for both happy path and error/edge cases?"
  - "Is the zero value of every exported struct safe and useful?"
  - "Are dependencies injected, not imported as globals?"
  - "Is logging structured and free of PII?"
  - "Does the code pass golangci-lint with no nolint pragmas that lack justification?"
  - "Are database queries parameterized?"
  - "Are HTTP responses consistent in format and status code usage?"
  - "Is there a graceful shutdown path for every long-running process?"
  - "Will this code work correctly under concurrent access?"
  - "Are public API types documented with GoDoc comments starting with the identifier name?"

# ─────────────────────────────────────────────────────────────────────────────
# 13. AGENT INSTRUCTIONS
# ─────────────────────────────────────────────────────────────────────────────
agent_instructions:
  generating_code:
    - "Always produce code that compiles. If unsure about an import path, use the standard library or state the assumption."
    - "Include necessary imports — never leave them implicit."
    - "When generating a new file, include the package declaration, imports, and all referenced types."
    - "Write GoDoc comments on every exported type, function, and method. The first word must be the identifier name."
    - "When creating a function, also produce a table-driven test with at least 3 cases (happy, error, edge)."
    - "Prefer stdlib solutions. Only suggest third-party packages when stdlib is significantly inferior."
    - "When modifying existing code, show only the changed function(s) with enough surrounding context to locate them."

  suggesting_architecture:
    - "Default to the hexagonal (ports & adapters) layout described in project_structure."
    - "For new services, scaffold cmd/, internal/domain, internal/service, internal/adapter, internal/handler, and internal/platform."
    - "Propose interfaces before implementations."
    - "When adding a new dependency (DB, queue, external API), always introduce it behind a domain interface."

  explaining_decisions:
    - "When choosing between approaches, briefly state the trade-off."
    - "Reference Go proverbs or standard library patterns when relevant."
    - "If a suggestion deviates from this skill's guidelines, call it out and explain why."
---
