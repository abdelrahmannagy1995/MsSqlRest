# Business Requirements Document (BRD)
## Project: MSSQL REST Extension (PostgREST-compatible + Auth & RLS)

### Revision
- Author: abdelrahmannagy1995
- Date: 2025-11-13
- Version: 3.0 (.NET 9)

---

## 1. Purpose / Executive Summary
Build a standalone, easily deployable extension/service for Microsoft SQL Server 2019+ that runs on Windows or Linux using .NET 9 and exposes a full-featured REST API for database tables with integrated authentication, authorization, and Row-Level Security (RLS). The extension provides feature parity with PostgREST for querying (complex joins, OR/AND filters, computed columns, aggregations, RPC) plus Supabase-like authentication and RLS enforcement using SQL Server SESSION_CONTEXT and security policies.

Core goals:
- **Easy deployment**: Single executable/installer for Windows (Windows Service) and Linux (systemd service), or Docker container, built with .NET 9.
- **Full PostgREST feature parity**: Support all PostgREST query features including complex filters, ordering, embedding/joins, aggregations, stored procedure calls (RPC), bulk operations, prefer headers, and computed columns.
- **Integrated authentication**: Email/password signup/login, JWT access + refresh tokens, password reset flows, email verification scaffolding.
- **Database-level RLS**: Enforce row-level security using SQL Server inline TVFs reading SESSION_CONTEXT, mimicking Supabase RLS patterns.
- **Production-ready**: Audit logging, configurable role/claim mappings, connection pool management with session context isolation, HTTPS, secure defaults.
- **Native SQL Server integration**: Leverage .NET 9's first-class SQL Server support via Microsoft.Data.SqlClient.

## 2. Stakeholders
- Product Owner / Requester (abdelrahmannagy1995)
- Backend Engineers (API, Database)
- DevOps / SRE / IT Operations
- Security / Compliance
- Application Developers (API consumers)
- QA / Testers
- End-users (indirect)

## 3. Business Objectives & Success Metrics
- **Deployment simplicity**: Install and configure on Windows or Linux in < 15 minutes (single executable or installer).
- **Feature completeness**: 100% coverage of PostgREST query operators and embedding strategies by GA.
- **Security**: Zero critical vulnerabilities; enforce HTTPS, JWT validation, and RLS policies by default.
- **Performance**: 95th-percentile latency < 100ms for simple SELECTs, < 300ms for complex embedded queries (baseline load, warm cache).
- **Reliability**: 99.9% uptime SLA when deployed with HA SQL Server.
- **Developer adoption**: Reduce backend integration time by 60% compared to hand-written REST APIs.
- **.NET ecosystem integration**: Full compatibility with Azure services, Visual Studio tooling, and Microsoft developer ecosystem.

## 4. Scope

### In scope for MVP (comprehensive):
- **REST endpoints** for SELECT/INSERT/UPDATE/DELETE on arbitrary tables (configurable whitelist).
- **Full PostgREST feature parity**:
  - **Complex filters**: eq, neq, gt, gte, lt, lte, like, ilike, in, is, not, or, and, fts (full-text search if supported).
  - **Ordering**: Multi-column sorting, nulls first/last.
  - **Limit/offset** and **Range header** pagination with 206 Partial Content responses.
  - **Embedding (joins)**: Foreign key expansion, many-to-one, one-to-many, many-to-many via join tables.
  - **Aggregations**: count, sum, avg, min, max via select=aggregate.
  - **Computed columns**: Virtual/computed column exposure.
  - **Stored procedure calls (RPC)**: POST /rpc/function_name with parameters.
  - **Bulk operations**: Batch inserts, upserts (ON CONFLICT equivalent via MERGE).
  - **Prefer headers**: return=representation|minimal|headers-only, resolution=ignore-duplicates|merge-duplicates, count=exact|planned|estimated.
  - **Vertical filtering (select)**: Column selection, renaming, casting.
  - **Resource embedding**: Nested resource expansion.
- **Authentication module**:
  - Signup, login, token refresh, logout.
  - Password hashing (Argon2id using Konscious.Security.Cryptography or built-in PBKDF2).
  - Password reset token generation and validation.
  - Email verification token scaffolding (integration with email provider via config).
  - JWT access tokens (short-lived) + refresh tokens (long-lived, rotatable).
- **DB-level RLS**: SESSION_CONTEXT + inline TVFs + SECURITY POLICY for filter and block predicates.
- **Connection pool handling**: Set session context per-request, reset on connection return.
- **Audit logging**: Auth events (login, signup, token refresh, failed auth), SQL errors, policy violations.
- **Configurable role/claim mapping**: Map JWT claims to SESSION_CONTEXT keys (e.g., role, user_id, org_id, custom claims).
- **Deployment packaging**:
  - Windows: Windows Service installer (MSI or self-contained executable with service registration).
  - Linux: systemd service unit, DEB/RPM packages or single executable with install script.
  - Docker: Official Docker image with docker-compose example.
  - .NET 9 deployment options: Framework-dependent, self-contained, single-file, and Native AOT (optional for maximum performance).
- **Configuration**: JSON (appsettings.json), environment variables, or Azure App Configuration.

### Out of scope for MVP:
- GraphQL support.
- Real-time subscriptions / WebSockets (future).
- Built-in admin UI (CLI-based policy management is in scope).
- Multi-database backend support (Postgres, MySQL); this is SQL Server only.
- SaaS multi-tenancy packaging (deployable by single organization initially).

## 5. Requirements

### 5.1 Functional Requirements
**FR-01**: The extension must expose REST endpoints for whitelisted tables supporting full CRUD (GET, POST, PATCH, DELETE).  
**FR-02**: The extension must support all PostgREST query operators: eq, neq, gt, gte, lt, lte, like, ilike, in, is, not, or, and.  
**FR-03**: The extension must support multi-column ordering, nulls first/last, limit/offset, and Range header pagination.  
**FR-04**: The extension must support resource embedding (foreign key joins) for one-to-many, many-to-one, and many-to-many relationships.  
**FR-05**: The extension must support aggregation functions (count, sum, avg, min, max) via query parameters.  
**FR-06**: The extension must support RPC calls to whitelisted stored procedures via POST /rpc/:function_name.  
**FR-07**: The extension must support bulk insert and upsert (MERGE-based) operations.  
**FR-08**: The extension must support Prefer headers (return, resolution, count) per PostgREST spec.  
**FR-09**: The extension must authenticate users using email/password and generate signed JWT access and refresh tokens using Microsoft.IdentityModel.Tokens.  
**FR-10**: The extension must accept JWTs in Authorization: Bearer header and map claims to SESSION_CONTEXT before executing queries.  
**FR-11**: The extension must use SQL Server security policies (inline TVFs) that read SESSION_CONTEXT to enforce RLS.  
**FR-12**: The extension must reset/set session context on each DB connection checkout to prevent cross-request leakage.  
**FR-13**: The extension must support refresh token rotation and revocation lists (stored in DB or Redis).  
**FR-14**: The extension must log authentication events, authorization failures, and SQL errors to configurable audit log (file, syslog, database, or Application Insights).  
**FR-15**: The extension must validate and whitelist table names, column names, and stored procedure names to prevent SQL injection.  
**FR-16**: The extension must provide a CLI or migration tool to scaffold RLS inline TVFs and SECURITY POLICY statements from templates.  
**FR-17**: The extension must support password reset token generation and validation with configurable expiry.  
**FR-18**: The extension must support email verification token scaffolding (tokens stored, validation endpoint provided; email sending delegated to external service).  
**FR-19**: The extension must integrate with .NET diagnostic tools (dotnet-counters, dotnet-trace, Application Insights).

### 5.2 Non-functional Requirements
**NFR-01**: The extension must be deployable as a single executable or installer on Windows Server 2016+ and Linux (Ubuntu 20.04+, RHEL 8+, etc.) using .NET 9 runtime or self-contained deployment.  
**NFR-02**: The extension must support installation as a Windows Service (auto-start on boot) using Microsoft.Extensions.Hosting.WindowsServices and Linux systemd service using Microsoft.Extensions.Hosting.Systemd.  
**NFR-03**: The extension must support configuration via appsettings.json, environment variables, and Azure App Configuration.  
**NFR-04**: The extension must enforce HTTPS by default using Kestrel; allow HTTP only via explicit config flag.  
**NFR-05**: Authentication tokens must be signed with secrets loaded from secure storage (appsettings, environment variables, Azure Key Vault, or AWS Secrets Manager).  
**NFR-06**: The extension must be horizontally scalable (stateless instances behind load balancer).  
**NFR-07**: Default performance: simple SELECTs ≤ 100ms (95th percentile), complex embedded queries ≤ 300ms under baseline load (1000 req/min per instance).  
**NFR-08**: The extension must maintain audit logs for 90 days (configurable) and support log rotation or integration with centralized logging (Application Insights, Seq, ELK).  
**NFR-09**: The extension must follow OWASP Secure Coding practices and pass security audit before GA.  
**NFR-10**: The extension must provide health check endpoints (/health, /readiness) compatible with ASP.NET Core health checks for orchestration (Kubernetes, Azure App Service).  
**NFR-11**: The extension should optionally support Native AOT compilation for reduced startup time and memory footprint (post-MVP).

## 6. Deployment & Installation Requirements
- **Windows**: Installer (MSI via WiX or InnoSetup) or PowerShell install script that:
  - Copies self-contained executable to Program Files.
  - Copies default appsettings.json to ProgramData.
  - Registers Windows Service using `sc.exe` or PowerShell.
  - Opens firewall port (optional prompt).
  - Provides post-install wizard or CLI for initial DB connection and admin user setup.
- **Linux**: DEB/RPM package or self-contained executable with install.sh script that:
  - Copies executable to /usr/local/bin or /opt.
  - Installs systemd service unit.
  - Creates config directory /etc/mssqlrest or similar.
  - Provides post-install setup command.
- **Docker**: Official Docker image (mcr.microsoft.com/dotnet/aspnet:9.0 or alpine variant) with environment variable config and docker-compose example including SQL Server container.
- **Azure**: Deploy to Azure App Service, Azure Container Apps, or Azure Kubernetes Service (AKS) with integrated Key Vault and Application Insights.

## 7. Constraints & Assumptions
- Target database: Microsoft SQL Server 2019 or later (required for SESSION_CONTEXT and security policy features).
- Single DB service account is used by the extension; no per-app-user DB accounts.
- Application instances are stateless and can scale horizontally.
- RLS policies are authored by DB admins or via provided CLI/migration tool.
- Email sending for verification/reset is delegated to external service (SendGrid, SES, SMTP relay, Azure Communication Services); extension generates tokens only.
- .NET 9 runtime or self-contained deployment is available on target systems.

## 8. Glossary
- **RLS**: Row-Level Security
- **TVF**: Table-Valued Function
- **SESSION_CONTEXT**: SQL Server session-scoped key-value storage.
- **RPC**: Remote Procedure Call (PostgREST term for stored procedure invocation).
- **Prefer header**: HTTP header used by PostgREST clients to request specific response formats or behaviors.
- **Embedding**: Including related resources in a single query response via joins.
- **Minimal API**: ASP.NET Core lightweight API approach with minimal ceremony.
- **Native AOT**: Ahead-of-time compilation producing native code without JIT runtime.

## 9. Risks & Mitigations
| Risk | Impact | Mitigation |
|------|--------|------------|
| Incorrect session context handling leaks user identity | High | Enforce set/reset on every connection checkout; integration tests validate isolation |
| SQL injection via identifier interpolation | High | Whitelist all identifiers; use parameterized queries (SqlParameter) for values |
| Compromised refresh tokens | Medium | Store refresh tokens server-side with rotation; support revocation list |
| RLS policies misconfigured leading to data exposure | High | Provide policy templates, automated tests, and DB-level auditing |
| Complex query parsing leads to errors or security issues | Medium | Use proven parser patterns; unit tests for all operators; fuzz testing |
| Deployment failures on diverse Windows/Linux environments | Medium | Test on multiple OS versions; provide detailed installation docs and troubleshooting guide |
| .NET 9 runtime version conflicts | Low | Use self-contained or single-file deployment to bundle runtime |

## 10. Acceptance Criteria
- User can install the extension on Windows Server 2019 and Ubuntu 22.04 in < 15 minutes.
- User can register, sign in, and receive access/refresh tokens via REST API.
- GET /table/:name returns only rows allowed by RLS for the authenticated user.
- Complex queries with filters (or, and, in), ordering, embedding, and aggregations return correct results matching PostgREST behavior.
- RPC calls to whitelisted stored procedures execute and return results.
- Session context is reliably set per-request; parallel requests do not leak identity.
- All endpoints enforce HTTPS in production configuration.
- Audit logs capture auth events and errors with timestamps and user context.
- Application integrates with Application Insights for telemetry (optional but recommended).

## 11. Timeline & Milestones (revised for .NET 9)
- **Week 0–1**: Finalize architecture, set up .NET 9 solution structure, design query parser, configure CI/CD pipeline.
- **Week 1–2**: Implement authentication module (signup, login, token refresh, password reset tokens, email verification tokens) using ASP.NET Core Identity primitives and Microsoft.IdentityModel.Tokens.
- **Week 2–4**: Implement session context handling with Microsoft.Data.SqlClient, basic table CRUD with minimal APIs, and RLS SQL templates.
- **Week 4–6**: Implement full PostgREST query parser (filters, ordering, embedding, aggregations, RPC) with LINQ and SQL generation.
- **Week 6–7**: Implement bulk operations, upsert via MERGE, Prefer headers, computed columns.
- **Week 7–8**: Packaging for Windows (service + installer), Linux (systemd + packages), Docker image using .NET publish profiles.
- **Week 8–9**: Integration tests using Testcontainers for SQL Server, security audit, performance testing with BenchmarkDotNet.
- **Week 9–10**: Documentation, deployment guides, NuGet package (optional), GA release.

---
End of BRD v3 (.NET 9)