# Functional Design Specification (FDS)
## Project: MSSQL REST Extension (PostgREST-compatible + Auth & RLS)

### Revision
- Author: abdelrahmannagy1995
- Date: 2025-11-13
- Version: 3.0 (.NET 9)

---

## 1. Purpose
This document outlines the functional design for the MSSQL REST Extension, highlighting the architecture, components, and their interactions. It aims to serve as a guideline for the development team to ensure all functional requirements are met during implementation.

## 2. System Architecture
The MSSQL REST Extension will adopt a microservices architecture comprising the following components:
- **Authentication Service**: Responsible for user authentication and issuing JWT tokens.
- **Authorization Service**: Responsible for validating user permissions and roles.
- **API Gateway**: Routes requests from clients to the appropriate service and handles load balancing.
- **Data Service**: Interfaces with the SQL Server database to perform data operations.
- **Logging Service**: Captures logs for auditing and debugging purposes.
- **Notification Service**: Sends notifications via email or push notifications using external services.
- **Monitoring Service**: Monitors system performance and health.

### High-Level Services Interaction
- Clients send requests to the API Gateway.
- API Gateway forwards authentication requests to the Authentication Service.
- On successful authentication, the service issues a JWT token.
- The API Gateway routes requests to the Data Service for CRUD operations, with the token validated by the Authorization Service.

## 3. Data Model
The data model will utilize the following database design:
- **Users**: Stores user information (ID, email, hashed password, roles, etc.).
- **Roles**: Stores roles with associated permissions.
- **Audits**: Records all access and actions performed, tagging them with user IDs and timestamps.
- **Notifications**: Stores notification preferences for each user.

## 4. Complete API Design
The following endpoints will be exposed by the API:
1. **Authentication Endpoints**
   - POST /auth/signup: User signup, generates a user entry in the database.
   - POST /auth/login: User login, returns JWT token.
   - POST /auth/logout: User logout, invalidates the token.

2. **User Management Endpoints**
   - GET /users: Retrieve list of users (requires admin role).
   - GET /users/{id}: Retrieve user details (requires admin role).
   - PUT /users/{id}: Update user details (requires admin role).
   - DELETE /users/{id}: Delete user (requires admin role).

3. **Data Endpoints**
   - GET /data: Retrieve data based on query parameters (filtering based on user permissions).
   - POST /data: Create new records.
   - PUT /data/{id}: Update an existing record.
   - DELETE /data/{id}: Delete a record.

4. **Audit Endpoints**
   - GET /audits: Retrieve list of audits (requires admin role).

5. **Notification Endpoints**
   - POST /notifications: Send notifications to users.

6. **Monitoring Endpoints**
   - GET /health: Health check endpoint for the monitoring service.

## 5. Security
Security practices will be integral to the design:
- **JWT Tokens**: Use for authenticating API requests, ensuring tokens are signed and validated on each request.
- **Role-Based Access Control**: Implement RBAC to restrict access to certain endpoints based on user roles.
- **Data Encryption**: Use encryption for sensitive data stored in the database.

## 6. Performance Considerations
- Ensure the service can handle a minimum of 1000 concurrent users.
- Response time for API requests should be < 200ms under normal load conditions.

## 7. Logging & Monitoring
- All services will implement structured logging.
- Monitoring services will track performance metrics and log aggregations using appropriate tools.
- Alerts will be set for critical events based on thresholds.

## 8. Deployment Strategy
The application will be deployed using containerization (e.g., Docker) with orchestration tools (Kubernetes) for scaling and management.

### Future Considerations
- Integration with additional third-party services.
- Expansion of the API for broader functionality and features.

---
End of FDS v3 (.NET 9)