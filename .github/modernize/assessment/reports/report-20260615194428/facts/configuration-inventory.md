# Configuration & Externalized Settings Inventory

This project uses a compact configuration model centered on one properties file with optional environment-variable overrides and servlet deployment descriptors.

## Configuration Sources

| Source | Type | Path/Location | Notes |
|---|---|---|---|
| loan-origination.properties | Application properties | src/main/resources/loan-origination.properties | Primary runtime configuration for DB and downstream URLs |
| web.xml | Servlet deployment descriptor | src/main/webapp/WEB-INF/web.xml | Declares servlet classes and URL mappings |
| Dockerfile | Container build/runtime metadata | Dockerfile | Exposes port 8080 and packages WAR |
| Environment variables | External override source | Process environment (`DB_*`, `*_SERVICE_BASE_URL`) | Overrides properties when present |
| build.gradle | Build configuration | build.gradle | Declares dependencies and Java compatibility |

## Build Profiles

| Profile | Activation | Purpose | Key Dependencies/Plugins |
|---|---|---|---|
| Default Gradle build | `gradle build` | Builds WAR artifact | `java`, `war` plugins |

## Runtime Profiles

| Profile | Activation Method | Config Files | Key Overrides |
|---|---|---|---|
| Default | implicit | loan-origination.properties | db.*, kyc/risk/ledger URLs, default interest rate |
| Env override | environment variables | env + loan-origination.properties fallback | `DB_HOST`, `DB_PORT`, `DB_NAME`, `DB_USER`, `DB_PASSWORD`, `KYC_SERVICE_BASE_URL`, `RISK_SERVICE_BASE_URL`, `LEDGER_SERVICE_BASE_URL`, `LOAN_DEFAULT_INTEREST_RATE` |

## Properties Inventory

| Property Key | Default | Profiles | Source |
|---|---|---|---|
| db.host | sqlserver | default | properties / `DB_HOST` |
| db.port | 1433 | default | properties / `DB_PORT` |
| db.name | ZavaBankDB | default | properties / `DB_NAME` |
| db.user | sa | default | properties / `DB_USER` |
| db.password | [MASKED] | default | properties / `DB_PASSWORD` |
| kyc.service.baseUrl | http://zava-kyc-service:8080 | default | properties / `KYC_SERVICE_BASE_URL` |
| risk.service.baseUrl | http://zava-risk-engine:8080 | default | properties / `RISK_SERVICE_BASE_URL` |
| ledger.service.baseUrl | http://zava-ledger:8080 | default | properties / `LEDGER_SERVICE_BASE_URL` |
| loan.default.interestRate | 0.0725 | default | properties / `LOAN_DEFAULT_INTEREST_RATE` |

## Startup Parameters & Resource Requirements

| Service | JVM/Runtime Options | Memory | Instance Count |
|---|---|---|---|
| ZavaLoanOriginationAPI | None explicitly configured in repository | Not specified | Not specified |

## Startup Dependency Chain

1. Servlet container starts application and loads `LoanBootstrapServlet` (`load-on-startup=1`).
2. `LoanBootstrapServlet` connects to SQL Server and ensures required tables exist.
3. API servlets become available for `/api/loans/*` and `/health`.
4. Runtime calls depend on availability of external KYC, risk, and ledger endpoints.

## Secrets & Sensitive Configuration

| Secret Reference | Type | Storage (masked) |
|---|---|---|
| db.password / `DB_PASSWORD` | Database credential | properties/env (masked) |
| db.user / `DB_USER` | Database username | properties/env |

### Secrets Provisioning Workflow

Secrets are expected to be provided through environment variables at deployment time, with file-based values as fallback defaults. Application startup reads environment variables first, then properties. Database secrets are consumed by the JDBC connection factory; downstream service URLs are consumed by loan orchestration logic.

## Feature Flags

No dedicated feature flag framework or conditional feature toggles were detected.

## Framework & Runtime Versions

| Component | Version | Source |
|---|---|---|
| Java target compatibility | 11 | build.gradle |
| Servlet API | 4.0.1 | build.gradle |
| SQL Server JDBC driver | 12.6.3.jre11 | build.gradle |
| org.json | 20140107 | build.gradle |
| Gradle | 9.5.1 (runtime used in session) | local execution output |
