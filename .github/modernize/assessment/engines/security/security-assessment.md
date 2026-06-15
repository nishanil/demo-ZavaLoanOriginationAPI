# Security Assessment Report

**Generated:** 2026-06-15T19:55:00Z

## Summary

| Metric | Count |
|--------|-------|
| Total Findings | 9 |
| CVE Vulnerabilities | 3 |
| CWE Vulnerabilities | 6 |
| Total Rules Assessed | 59 |
| Rules Passed | 53 |

### By Severity

| Severity | Count |
|----------|-------|
| mandatory | 3 |
| optional | 3 |
| potential | 3 |

### By Category

| Category | Count |
|----------|-------|
| CVE | 3 |
| Code Quality | 3 |
| Credentials & Secrets | 3 |

---

## CVE Findings (Dependency Vulnerabilities)

### CVE-2025-59250: JDBC Driver for SQL Server has improper input validation issue
- **Severity:** mandatory
- **Story Points:** 1
- **Files:** build.gradle:16

[CVE-2025-59250](https://github.com/advisories/GHSA-m494-w24q-6f7w): JDBC Driver for SQL Server has improper input validation issue

Severity: HIGH (CVSS 8.1)

Improper input validation in the Microsoft JDBC Driver for SQL Server allows an unauthorized attacker to perform spoofing over a network.

Affected dependencies:
- com.microsoft.sqlserver:mssql-jdbc:12.6.3.jre11 (declared at build.gradle:16)
  Vulnerable range: >= 12.6.0.jre11, < 12.6.5.jre11

Recommended fix:
- Upgrade com.microsoft.sqlserver:mssql-jdbc to 12.6.5.jre11 or later

---

### CVE-2023-5072: Java: DoS Vulnerability in JSON-JAVA
- **Severity:** mandatory
- **Story Points:** 1
- **Files:** build.gradle:17

[CVE-2023-5072](https://github.com/advisories/GHSA-4jq9-2xhw-jpx7): Denial of Service vulnerability in JSON-Java

Severity: HIGH (CVSS 7.5)

A bug in the JSON-Java parser means that an input string of modest size can lead to indefinite amounts of memory being used, causing an OutOfMemoryError. Nested JSON objects can cause exponential memory consumption.

Affected dependencies:
- org.json:json:20140107 (declared at build.gradle:17)
  Vulnerable range: <= 20230618

Recommended fix:
- Upgrade org.json:json to 20231013 or later

---

### CVE-2022-45688: json stack overflow vulnerability
- **Severity:** mandatory
- **Story Points:** 1
- **Files:** build.gradle:17

[CVE-2022-45688](https://github.com/advisories/GHSA-3vqj-43w4-2q58): Stack overflow in XML.toJSONObject component

Severity: HIGH (CVSS 7.5)

A stack overflow in org.json:json before version 20230227 allows attackers to cause a Denial of Service (DoS) via crafted JSON or XML data.

Affected dependencies:
- org.json:json:20140107 (declared at build.gradle:17)
  Vulnerable range: < 20230227

Recommended fix:
- Upgrade org.json:json to 20230227 or later (20231013 or later is recommended to also fix CVE-2023-5072)

---

## CWE Findings (Code-Level Vulnerabilities)

### CWE-259: Use of Hard-coded Password
- **Category:** Credentials & Secrets
- **Severity:** optional
- **Story Points:** 5
- **Files:** src/main/resources/loan-origination.properties, Dockerfile

The database password 'YourStrong!Passw0rd' is hard-coded in two places: (1) loan-origination.properties line 5 as 'db.password', and (2) Dockerfile line 11 as 'ENV DB_PASSWORD=YourStrong!Passw0rd', embedding the credential directly into the container image at build time.

---

### CWE-477: Use of Obsolete Function
- **Category:** Code Quality
- **Severity:** optional
- **Story Points:** 1
- **Files:** src/main/java/com/zavabank/loanorigination/LoanConnectionFactory.java

LoanConnectionFactory uses `Class.forName("com.microsoft.sqlserver.jdbc.SQLServerDriver")` at line 10 to manually register the JDBC driver. Since JDBC 4.0 (Java 6+), drivers are automatically loaded via the ServiceLoader mechanism, making explicit Class.forName() driver registration obsolete.

---

### CWE-681: Incorrect Conversion between Numeric Types
- **Category:** Code Quality
- **Severity:** potential
- **Story Points:** 3
- **Files:** src/main/java/com/zavabank/loanorigination/LoanApplyServlet.java

In `LoanApplyServlet.doPost()` at line 37, a BigDecimal is created directly from a double: `new BigDecimal(payload.optDouble("requestedAmount", 0d))`. The BigDecimal(double) constructor preserves the exact binary floating-point representation, producing unexpected precision for financial amounts.

---

### CWE-778: Insufficient Logging
- **Category:** Credentials & Secrets
- **Severity:** potential
- **Story Points:** 3
- **Files:** src/main/java/com/zavabank/loanorigination/LoanApplyServlet.java, src/main/java/com/zavabank/loanorigination/LoanStatusServlet.java

The application contains no logging framework (no SLF4J, Log4j, java.util.logging, or System.out statements). Security-critical events such as loan application submissions, loan decisions, service call failures (caught and silently swallowed in `callJsonApi`), and database errors are not logged, preventing audit trails and incident detection.

---

### CWE-798: Use of Hard-coded Credentials
- **Category:** Credentials & Secrets
- **Severity:** optional
- **Story Points:** 5
- **Files:** src/main/resources/loan-origination.properties, Dockerfile

Hard-coded credentials appear in multiple locations: (1) loan-origination.properties line 5 ('db.password') and line 4 ('db.user=sa' — the SQL Server system administrator account), (2) Dockerfile lines 10-11 ('ENV DB_USER=sa' and 'ENV DB_PASSWORD=YourStrong!Passw0rd') which bake the credentials into the container image. Using the SA account with a hard-coded password presents a critical security risk.

---

### CWE-1057: Data Access Operations Outside of Expected Data Manager Component
- **Category:** Code Quality
- **Severity:** potential
- **Story Points:** 5
- **Files:** src/main/java/com/zavabank/loanorigination/LoanApplyServlet.java, src/main/java/com/zavabank/loanorigination/LoanStatusServlet.java, src/main/java/com/zavabank/loanorigination/LoanBootstrapServlet.java

JDBC data-access operations (INSERT, UPDATE, SELECT, CREATE TABLE) are performed directly inside servlet classes (LoanApplyServlet.insertApplication/updateApplication/insertDecision, LoanStatusServlet.doGet, LoanBootstrapServlet.init) rather than in a dedicated data manager or repository component. This tightly couples controller logic with persistence logic.
