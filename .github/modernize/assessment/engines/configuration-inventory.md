# Configuration & Externalized Settings Inventory

ZavaLoanOriginationAPI uses **two configuration sources** — a bundled properties file and container environment variables — with no external config server, secret store, or feature-flag framework.

## Configuration Sources

| Source | Type | Path/Location | Notes |
|---|---|---|---|
| `loan-origination.properties` | Java properties file | `src/main/resources/loan-origination.properties` | Bundled into the WAR; provides defaults for all runtime properties |
| Environment variables | OS/container env | Set in `Dockerfile` and overridable at container runtime | Take precedence over properties file values in `LoanOriginationConfig.read()` |
| `web.xml` | Servlet deployment descriptor | `src/main/webapp/WEB-INF/web.xml` | Declares servlet mappings and startup order; no externalized properties |
| `build.gradle` | Gradle build script | `build.gradle` | Declares dependencies, Java 11 source/target compatibility, and WAR artifact name |
| `Dockerfile` | Container image definition | `Dockerfile` | Multi-stage build (Gradle 7.6 + JDK 11 → Tomcat 9 + JDK 11); bakes default env vars into the image |

No Spring Cloud Config server, Azure App Configuration, Consul KV, HashiCorp Vault, or AWS AppConfig integration is present.

## Build Profiles

| Profile | Activation | Purpose | Key Dependencies/Plugins |
|---|---|---|---|
| (default, single) | Always | Compile and package the application as a WAR | `java` plugin, `war` plugin; `javax.servlet-api:4.0.1` (compileOnly), `mssql-jdbc:12.6.3.jre11`, `org.json:20140107` |

No Maven/Gradle profiles, build types, or conditional compilation are defined. The project has a single build configuration.

## Runtime Profiles

| Profile | Activation Method | Config Files | Key Overrides |
|---|---|---|---|
| (none — single environment) | N/A | `loan-origination.properties` (bundled), environment variables | Environment variables (e.g., `DB_HOST`, `DB_PASSWORD`) override bundled defaults at runtime |

No Spring profiles, `.env` variants, or environment-specific configuration files exist. The application reads a single properties file with environment variable fallback logic in `LoanOriginationConfig`.

## Properties Inventory

### ZavaLoanOriginationAPI

| Property Key | Default Value | Environment Variable Override | Source |
|---|---|---|---|
| `db.host` | `sqlserver` | `DB_HOST` | `loan-origination.properties` / env |
| `db.port` | `1433` | `DB_PORT` | `loan-origination.properties` / env |
| `db.name` | `ZavaBankDB` | `DB_NAME` | `loan-origination.properties` / env |
| `db.user` | `sa` | `DB_USER` | `loan-origination.properties` / env |
| `db.password` | [MASKED] | `DB_PASSWORD` | `loan-origination.properties` / env |
| `kyc.service.baseUrl` | `http://zava-kyc-service:8080` | `KYC_SERVICE_BASE_URL` | `loan-origination.properties` / env |
| `risk.service.baseUrl` | `http://zava-risk-engine:8080` | `RISK_SERVICE_BASE_URL` | `loan-origination.properties` / env |
| `ledger.service.baseUrl` | `http://zava-ledger:8080` | `LEDGER_SERVICE_BASE_URL` | `loan-origination.properties` / env |
| `loan.default.interestRate` | `0.0725` | `LOAN_DEFAULT_INTEREST_RATE` | `loan-origination.properties` / env |

Resolution order in `LoanOriginationConfig.read()`: environment variable → properties file value. JDBC URL is assembled as `jdbc:sqlserver://{host}:{port};databaseName={name};encrypt=false;trustServerCertificate=true`.

## Startup Parameters & Resource Requirements

| Service | JVM/Runtime Options | Memory | Instance Count |
|---|---|---|---|
| ZavaLoanOriginationAPI | Tomcat 9 defaults; no explicit `-Xms`/`-Xmx` or system properties set | Not specified (container default) | 1 (no scaling config) |

No JVM heap tuning, `-D` system properties, Kubernetes resource requests/limits, or Docker memory constraints are configured. The Dockerfile uses `CMD ["catalina.sh", "run"]` with Tomcat's default JVM options.

## Startup Dependency Chain

1. **SQL Server** must be reachable — `LoanBootstrapServlet` (load-on-startup=1) connects immediately at container startup to create schema tables. If SQL Server is unavailable, the servlet throws `ServletException` and the WAR fails to deploy.
2. **ZavaLoanOriginationAPI** starts after its own container initializes and `LoanBootstrapServlet` completes successfully.
3. **zava-kyc-service**, **zava-risk-engine**, **zava-ledger** — referenced only at request time, not at startup. Their unavailability causes runtime 500 errors but does not prevent the WAR from deploying.

There is **no readiness probe, no dockerize wait-for-TCP mechanism, no Kubernetes `depends_on`, and no retry on startup failure**. SQL Server must be available before the application container starts.

## Secrets & Sensitive Configuration

| Secret Reference | Type | Storage |
|---|---|---|
| `db.password` / `DB_PASSWORD` | Database password | Plaintext in `loan-origination.properties`; also baked as default ENV in `Dockerfile` as `ENV DB_PASSWORD=YourStrong!Passw0rd` [MASKED] |
| `db.user` / `DB_USER` | Database username | Plaintext in `loan-origination.properties` and `Dockerfile` ENV |
| `DB_HOST`, `DB_PORT`, `DB_NAME` | DB connection details | Plaintext in `loan-origination.properties` and `Dockerfile` ENV |

> ⚠️ The database password is committed to source control in both `loan-origination.properties` and the `Dockerfile`. No encryption (Jasypt, sealed secrets, DPAPI) is applied.

### Secrets Provisioning Workflow

There is **no formal secrets provisioning workflow**. Credentials are resolved in this order:

1. Container runtime environment variables (e.g., set via `docker run -e DB_PASSWORD=...` or a Kubernetes `Secret` → `envFrom`).
2. Fall back to the bundled `loan-origination.properties` defaults if no environment variable is present.

No managed identity, Azure Key Vault, HashiCorp Vault, AWS Secrets Manager, GitHub Actions secret injection, or RBAC-controlled secret access is configured. The default credentials baked into the `Dockerfile` are identical to the properties file defaults, making the fallback effectively the same hardcoded value in both paths.

## Feature Flags

No feature flag framework is present. There are no `@ConditionalOnProperty` annotations, LaunchDarkly/Unleash integrations, custom toggle beans, or A/B testing configurations. All application behavior is statically compiled.

## Framework & Runtime Versions

| Component | Version | Source |
|---|---|---|
| Java (source/target compatibility) | 11 | `build.gradle` (`sourceCompatibility = JavaVersion.VERSION_11`) |
| Gradle (build tool) | 7.6 (Docker build stage) | `Dockerfile` (`FROM gradle:7.6-jdk11`) |
| Servlet API | javax.servlet-api 4.0.1 | `build.gradle` (compileOnly) |
| Microsoft JDBC Driver for SQL Server | 12.6.3.jre11 | `build.gradle` |
| org.json | 20140107 | `build.gradle` |
| Tomcat (servlet container) | 9 | `Dockerfile` (`FROM tomcat:9-jdk11`) |
| JDK (runtime, Docker) | 11 | `Dockerfile` base images (`gradle:7.6-jdk11`, `tomcat:9-jdk11`) |
| WAR artifact name | ROOT.war | `build.gradle` (`archiveBaseName = 'ROOT'`) |
