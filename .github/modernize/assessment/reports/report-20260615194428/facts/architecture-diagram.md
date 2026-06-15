# Architecture Diagram

This diagram summarizes the servlet-based loan origination API and its key runtime components.

## Application Architecture

```mermaid
flowchart TD
    subgraph Client["Client Layer"]
        Consumer["Internal Banking Client"]
    end
    subgraph App["Application Layer - Java Servlet WAR"]
        Health["HealthServlet"]
        Apply["LoanApplyServlet"]
        Status["LoanStatusServlet"]
        Bootstrap["LoanBootstrapServlet"]
        Config["LoanOriginationConfig"]
        ConnFactory["LoanConnectionFactory"]
    end
    subgraph Data["Data Layer"]
        JDBC["JDBC + PreparedStatement"]
        SQL[("SQL Server")]
    end
    subgraph External["External Services"]
        KYC["KYC Service"]
        Risk["Risk Engine Service"]
        Ledger["Ledger Service"]
    end

    Consumer -->|"HTTP requests"| Apply
    Consumer -->|"HTTP requests"| Status
    Consumer -->|"health check"| Health
    Bootstrap -->|"startup DDL"| ConnFactory
    Apply -->|"reads config"| Config
    Status -->|"reads config"| Config
    Apply -->|"db writes"| JDBC
    Status -->|"db reads"| JDBC
    JDBC -->|"SQL"| SQL
    Apply -->|"POST /api/kyc/verify"| KYC
    Apply -->|"POST /api/risk/score"| Risk
    Apply -->|"POST /api/accounts/create"| Ledger
```

### Technology Stack Summary

| Layer | Technology | Version | Purpose |
|---|---|---|---|
| Presentation | Javax Servlet API | 4.0.1 | HTTP endpoint handling |
| Business | Java application logic | Java 11 target | Loan orchestration and decision flow |
| Data Access | JDBC + SQL Server driver | mssql-jdbc 12.6.3.jre11 | Persist and query loan records |
| Serialization | org.json | 20140107 | Parse request payloads and produce JSON responses |

### Data Storage & External Services

The service stores loan applications and decision records in SQL Server tables (`LoanApplications`, `LoanDecisions`) and calls three downstream HTTP services for KYC verification, risk scoring, and ledger account creation.

### Key Architectural Decisions

- Uses synchronous orchestration in `LoanApplyServlet` to chain KYC, risk, and ledger calls in one request lifecycle.
- Uses JDBC prepared statements for CRUD-style database access without an ORM layer.
- Bootstraps schema at startup using `LoanBootstrapServlet`.

## Component Relationships

```mermaid
flowchart LR
    subgraph Presentation
        HS["HealthServlet"]
        LAS["LoanApplyServlet"]
        LSS["LoanStatusServlet"]
        LBS["LoanBootstrapServlet"]
    end
    subgraph Business["Business Logic"]
        Orch["OrchestrationResult Logic"]
        APICall["callJsonApi"]
        Conf["LoanOriginationConfig"]
    end
    subgraph DataAccess["Data Access"]
        CF["LoanConnectionFactory"]
        SQLA["LoanApplications SQL"]
        SQLD["LoanDecisions SQL"]
    end
    subgraph Infra["Infrastructure"]
        Props["loan-origination.properties"]
    end

    LAS -->|"orchestrates"| Orch
    LAS -->|"calls"| APICall
    LAS -->|"opens DB connection"| CF
    LSS -->|"opens DB connection"| CF
    LBS -->|"creates tables"| CF
    CF -->|"uses"| Conf
    Conf -->|"loads"| Props
    LAS -->|"insert/update"| SQLA
    LAS -->|"insert"| SQLD
    LSS -->|"select"| SQLA
```

### Component Inventory

| Component | Layer | Type | Responsibility |
|---|---|---|---|
| LoanApplyServlet | Presentation | Servlet | Accepts loan applications and orchestrates downstream decisions |
| LoanStatusServlet | Presentation | Servlet | Returns loan status and decision details by application id |
| LoanBootstrapServlet | Presentation | Servlet (startup) | Creates required database tables during initialization |
| HealthServlet | Presentation | Servlet | Returns basic health response |
| LoanOriginationConfig | Business Logic | Configuration utility | Resolves config values from env vars and properties |
| LoanConnectionFactory | Data Access | Connection factory | Creates SQL Server JDBC connections |
