# ZavaLoanOriginationAPI

Backend API service for Zava Bank's loan origination workflow — handles loan application submission, underwriting orchestration, and status tracking.

## Tech Stack

- **Language:** Java
- **Build Tool:** Gradle
- **Runtime:** Servlet-based (WAR deployment)
- **Containerization:** Docker

## Features

- Loan application submission endpoint
- Underwriting orchestration with external service integration
- Loan status retrieval
- Health check endpoint
- Configurable connection settings for downstream services

## Getting Started

### Prerequisites

- Java 17+
- Gradle 8.9+
- Docker (for containerized deployment)

### Build

```bash
./gradlew build
```

### Run with Docker

```bash
docker build -t zava-loan-origination-api .
docker run -p 8080:8080 zava-loan-origination-api
```

## Project Structure

```
├── src/main/java/com/zavabank/loanorigination/
│   ├── LoanApplyServlet.java        # Loan application endpoint
│   ├── LoanStatusServlet.java       # Status query endpoint
│   ├── LoanBootstrapServlet.java    # Service initialization
│   ├── LoanConnectionFactory.java   # Connection management
│   ├── LoanOriginationConfig.java   # Configuration
│   └── HealthServlet.java           # Health check
├── src/main/resources/
│   └── loan-origination.properties  # Service configuration
├── src/main/webapp/
│   └── WEB-INF/web.xml              # Servlet mappings
├── build.gradle                     # Build configuration
├── settings.gradle                  # Gradle settings
└── Dockerfile                       # Container definition
```
