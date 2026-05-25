# Threat Model: Shopping Cart Application

## 1. Overview

### 1.1 Service Purpose
The Shopping Cart application is a Ruby on Rails-based web service that provides e-commerce functionality for managing shopping carts and user interactions. The application enables users to:
- Create and manage user accounts with authentication
- Browse and manage shopping cart items
- Post microposts (messages/updates)
- Follow/unfollow other users
- View user profiles and activity feeds

### 1.2 Scope
This threat model covers the entire Shopping Cart web application including:
- Web-based user interface
- RESTful API endpoints
- Database layer (SQLite for development/test, PostgreSQL for production)
- User authentication and session management
- User-generated content management (microposts)
- Social networking features (following/followers)

### 1.3 Technology Stack
- **Framework**: Ruby on Rails 4.0.8
- **Language**: Ruby 2.0.0
- **Databases**: SQLite3 (development/test), PostgreSQL (production)
- **Authentication**: BCrypt with has_secure_password
- **Frontend**: JavaScript, Bootstrap, jQuery
- **Testing**: RSpec, Capybara, Cucumber
- **Deployment**: Production deployment capability with Rails 12factor

## 2. Data Flow Diagram

```
┌──────────────┐
│   End Users  │
│  (Browsers)  │
└──────┬───────┘
       │ HTTP/HTTPS
       │ Requests
       ▼
┌──────────────────────────────────────────┐
│        Rails Application Server          │
│                                          │
│  ┌────────────────────────────────┐    │
│  │     Controllers Layer          │    │
│  │  - UsersController            │    │
│  │  - SessionsController         │    │
│  │  - MicropostsController       │    │
│  │  - RelationshipsController    │    │
│  │  - StaticPagesController      │    │
│  └──────────┬─────────────────────┘    │
│             │                           │
│  ┌──────────▼─────────────────────┐    │
│  │     Models Layer               │    │
│  │  - User (authentication)       │    │
│  │  - Micropost                   │    │
│  │  - Relationship                │    │
│  └──────────┬─────────────────────┘    │
│             │                           │
└─────────────┼───────────────────────────┘
              │ ActiveRecord
              │ SQL Queries
              ▼
┌──────────────────────────────┐
│    Database Server           │
│  - Users table               │
│  - Microposts table          │
│  - Relationships table       │
│  (SQLite3/PostgreSQL)        │
└──────────────────────────────┘
```

### 2.1 External Data Flows
- **User Authentication**: Login credentials flow from browser to Rails application
- **User Registration**: New user data submitted via forms
- **Content Creation**: Micropost content submitted by users
- **Social Actions**: Follow/unfollow requests between users
- **Session Management**: Remember tokens stored in cookies

### 2.2 Internal Data Flows
- **Database Queries**: ActiveRecord ORM translates Ruby code to SQL
- **Password Hashing**: BCrypt processes passwords before storage
- **Session Tokens**: Secure random tokens generated for authentication
- **Email Processing**: Email addresses normalized to lowercase

## 3. Dependencies

### 3.1 Core Framework Dependencies
- **rails** (4.0.8) - Web application framework
- **sprockets** (2.11.0) - Asset pipeline
- **bcrypt-ruby** (3.1.2) - Password hashing

### 3.2 Database Dependencies
- **sqlite3** (1.3.8) - Development/test database
- **pg** (0.15.1) - Production PostgreSQL adapter

### 3.3 Frontend Dependencies
- **bootstrap-sass** (2.3.2.0) - UI framework
- **jquery-rails** (3.0.4) - JavaScript library
- **sass-rails** (4.0.1) - CSS preprocessing
- **coffee-rails** (4.0.1) - JavaScript preprocessing
- **uglifier** (2.1.1) - JavaScript compression
- **turbolinks** (1.1.1) - Page navigation

### 3.4 Utility Dependencies
- **faker** (1.1.2) - Test data generation
- **will_paginate** (3.0.4) - Pagination
- **bootstrap-will_paginate** (0.0.9) - Pagination styling
- **jbuilder** (1.0.2) - JSON API builder

### 3.5 Testing Dependencies
- **rspec-rails** (2.13.1) - Testing framework
- **capybara** (2.1.0) - Integration testing
- **selenium-webdriver** (2.35.1) - Browser automation
- **factory_girl_rails** (4.2.0) - Test fixtures
- **cucumber-rails** (1.3.0) - BDD testing
- **database_cleaner** - Test database cleanup
- **guard-rspec** (2.5.0) - Automated testing
- **spork-rails** (4.0.0) - Test server

### 3.6 Production Dependencies
- **rails_12factor** (0.0.2) - Heroku deployment support

### 3.7 Infrastructure Dependencies
- Web server (WEBrick development, production server TBD)
- Database server
- Ruby runtime environment
- Operating system libraries

## 4. Entry Points

### 4.1 Web Interface Entry Points

#### 4.1.1 Public Pages (Unauthenticated)
- **GET /** - Home page
- **GET /signup** - User registration form
- **GET /signin** - User login form
- **GET /help** - Help page
- **GET /about** - About page
- **GET /contact** - Contact page
- **GET /users/:id** - Public user profile view

#### 4.1.2 Authentication Entry Points
- **POST /sessions** - User login (authentication)
- **DELETE /signout** - User logout
- **POST /users** - User registration

#### 4.1.3 Authenticated User Entry Points
- **GET /users** - User index (requires sign-in)
- **GET /users/:id/edit** - Edit profile (requires correct user)
- **PUT/PATCH /users/:id** - Update profile (requires correct user)
- **DELETE /users/:id** - Delete user (requires admin)
- **GET /users/:id/following** - View following list
- **GET /users/:id/followers** - View followers list

#### 4.1.4 Micropost Entry Points
- **POST /microposts** - Create micropost (requires sign-in)
- **DELETE /microposts/:id** - Delete micropost (requires ownership)

#### 4.1.5 Relationship Entry Points
- **POST /relationships** - Follow user (requires sign-in)
- **DELETE /relationships/:id** - Unfollow user (requires sign-in)

### 4.2 Data Entry Points
- **Form inputs**: User registration, login, profile updates
- **Text fields**: Micropost content (user-generated content)
- **URL parameters**: User IDs, micropost IDs, relationship IDs
- **Cookies**: Session remember tokens
- **HTTP Headers**: Accept, Content-Type, User-Agent, etc.

### 4.3 API Entry Points
While the application is primarily web-based, the Swagger documentation suggests planned API endpoints:
- Shopping cart management endpoints
- Item management endpoints
- User management API endpoints

## 5. Exit Points

### 5.1 Data Output Points
- **HTML Responses**: Rendered views with user data
- **JSON Responses**: API responses (if implemented)
- **HTTP Redirects**: Post-action redirects
- **Flash Messages**: Success/error notifications
- **Cookies**: Remember token storage

### 5.2 Database Writes
- User record creation and updates
- Password digest storage
- Micropost creation and deletion
- Relationship creation and deletion
- Session token updates

### 5.3 External Communication
- **Email**: User email addresses stored (potential future email notifications)
- **Logs**: Application and error logs
- **Error Pages**: 404, 422, 500 error pages

### 5.4 Data Exposure Points
- User profiles (name, email, microposts)
- Micropost content and timestamps
- User relationships (followers/following lists)
- Pagination data
- User feed aggregation

## 6. Assets

### 6.1 Critical Assets

#### 6.1.1 User Credentials
- **Password Digests**: BCrypt-hashed passwords
- **Email Addresses**: Unique user identifiers
- **Remember Tokens**: Session authentication tokens
- **Sensitivity**: Critical - compromise leads to account takeover

#### 6.1.2 User Data
- **Personal Information**: Names, email addresses, age
- **User-Generated Content**: Microposts
- **Social Graph**: Following/follower relationships
- **Sensitivity**: High - privacy concerns

#### 6.1.3 Session Data
- **Remember Tokens**: Authentication session identifiers
- **Session Cookies**: Persistent login state
- **Sensitivity**: Critical - session hijacking risk

### 6.2 Important Assets

#### 6.2.1 Application Data
- **User Accounts**: Complete user records
- **Microposts**: User-generated content
- **Relationships**: Social connection data
- **Timestamps**: Activity tracking
- **Sensitivity**: Medium to High

#### 6.2.2 System Configuration
- **Database Credentials**: Database connection strings
- **Secret Keys**: Rails secret_key_base
- **Environment Variables**: Configuration settings
- **Sensitivity**: Critical - full system compromise

### 6.3 Supporting Assets
- **Static Content**: Images, CSS, JavaScript files
- **Application Code**: Ruby source files
- **Database Schema**: Table structures
- **Test Data**: Development/test fixtures
- **Sensitivity**: Low to Medium

## 7. Trust Levels

### 7.1 Unauthenticated Users (Public)
**Trust Level**: None
- **Access**: Public pages only (home, about, help, contact, signup, signin)
- **Capabilities**: View public content, register, attempt login
- **Restrictions**: Cannot access user data, create content, or modify data
- **Validation**: All input treated as untrusted

### 7.2 Authenticated Users (Regular Users)
**Trust Level**: Low to Medium
- **Access**: Own profile, create microposts, manage relationships
- **Capabilities**:
  - View all users and public profiles
  - Create, view, and delete own microposts
  - Follow/unfollow other users
  - Update own profile information
  - View personalized feed
- **Restrictions**: 
  - Cannot access other users' edit functions
  - Cannot delete other users' content
  - Cannot perform administrative actions
- **Validation**: Authenticated but input still untrusted

### 7.3 Authenticated Correct User
**Trust Level**: Medium
- **Access**: Full control over own account
- **Capabilities**:
  - All regular user capabilities
  - Edit own profile
  - Update own password
- **Restrictions**: Limited to own resources only
- **Validation**: Authorization verified per request

### 7.4 Administrator Users
**Trust Level**: High
- **Access**: Full system access
- **Capabilities**:
  - All user capabilities
  - Delete any user account
  - View all user data
  - Administrative functions
- **Restrictions**: Admin flag must be set in database
- **Validation**: Admin status verified for privileged operations

### 7.5 System/Application
**Trust Level**: Full
- **Access**: Complete database and file system access
- **Capabilities**:
  - Execute database queries
  - Process all user input
  - Generate session tokens
  - Hash passwords
  - Manage application state
- **Restrictions**: Bound by Rails framework security
- **Validation**: Internal operations trusted

### 7.6 Database
**Trust Level**: Full
- **Access**: All application data
- **Capabilities**:
  - Store and retrieve all data
  - Execute SQL queries
  - Enforce data integrity constraints
- **Restrictions**: No direct user access
- **Validation**: SQL queries via ActiveRecord ORM

## 8. STRIDE Threat Analysis

### 8.1 Spoofing Identity Threats

#### T-S-01: Weak Password Policy
**Threat**: Attackers can compromise accounts through brute force or dictionary attacks.
- **Attack Vector**: Login form accepts weak passwords (minimum 6 characters only)
- **Impact**: Unauthorized account access
- **Affected Assets**: User credentials, user data
- **Likelihood**: Medium
- **Severity**: High

#### T-S-02: Session Token Prediction
**Threat**: Weak session token generation could allow session hijacking.
- **Attack Vector**: If SecureRandom is compromised or implementation flawed
- **Impact**: Account takeover without credentials
- **Affected Assets**: Session tokens, user accounts
- **Likelihood**: Low
- **Severity**: High

#### T-S-03: Email Enumeration
**Threat**: Attackers can determine valid email addresses through login error messages.
- **Attack Vector**: Different error messages for invalid email vs invalid password
- **Impact**: User enumeration for targeted attacks
- **Affected Assets**: User email addresses
- **Likelihood**: Medium
- **Severity**: Low

#### T-S-04: Insufficient Authentication on API Endpoints
**Threat**: Planned API endpoints may lack proper authentication.
- **Attack Vector**: Direct API calls bypassing web authentication
- **Impact**: Unauthorized data access
- **Affected Assets**: All user data
- **Likelihood**: Medium (if APIs implemented)
- **Severity**: High

### 8.2 Tampering Threats

#### T-T-01: SQL Injection
**Threat**: Malicious SQL injection through user inputs.
- **Attack Vector**: User input in database queries (mitigated by ActiveRecord but still possible)
- **Impact**: Database compromise, data theft, data manipulation
- **Affected Assets**: Entire database
- **Likelihood**: Low (ActiveRecord provides protection)
- **Severity**: Critical

#### T-T-02: Cross-Site Request Forgery (CSRF)
**Threat**: Attackers trick authenticated users into performing unwanted actions.
- **Attack Vector**: Malicious sites submitting forms to the application
- **Impact**: Unauthorized actions on behalf of users
- **Affected Assets**: User accounts, microposts, relationships
- **Likelihood**: Medium (Rails has built-in protection if properly configured)
- **Severity**: High

#### T-T-03: Mass Assignment Vulnerability
**Threat**: Attackers modify unintended model attributes through parameter manipulation.
- **Attack Vector**: Adding unexpected parameters to form submissions
- **Impact**: Privilege escalation (e.g., setting admin flag), data corruption
- **Affected Assets**: User accounts, admin privileges
- **Likelihood**: Low (Strong parameters used)
- **Severity**: Critical

#### T-T-04: File Upload Manipulation
**Threat**: If file uploads are added, malicious files could be uploaded.
- **Attack Vector**: Uploading executable files or oversized files
- **Impact**: Server compromise, resource exhaustion
- **Affected Assets**: Server infrastructure
- **Likelihood**: Low (feature not currently implemented)
- **Severity**: High

### 8.3 Repudiation Threats

#### T-R-01: Insufficient Audit Logging
**Threat**: Actions cannot be traced back to specific users.
- **Attack Vector**: No comprehensive logging of user actions
- **Impact**: Cannot prove who performed specific actions
- **Affected Assets**: All user actions
- **Likelihood**: High
- **Severity**: Medium

#### T-R-02: No Timestamp Verification
**Threat**: Users can claim actions happened at different times.
- **Attack Vector**: Lack of immutable timestamp logging
- **Impact**: Disputes about when actions occurred
- **Affected Assets**: Microposts, relationships
- **Likelihood**: Low
- **Severity**: Low

#### T-R-03: No Account Activity Monitoring
**Threat**: Suspicious account activity goes undetected.
- **Attack Vector**: No monitoring of failed login attempts, unusual access patterns
- **Impact**: Delayed detection of compromised accounts
- **Affected Assets**: User accounts
- **Likelihood**: Medium
- **Severity**: Medium

### 8.4 Information Disclosure Threats

#### T-I-01: Sensitive Data in URLs
**Threat**: User IDs and sensitive parameters exposed in URLs.
- **Attack Vector**: URLs logged in browser history, server logs, referrer headers
- **Impact**: Information leakage
- **Affected Assets**: User IDs, session identifiers
- **Likelihood**: High
- **Severity**: Low to Medium

#### T-I-02: Detailed Error Messages
**Threat**: Error pages reveal system information.
- **Attack Vector**: Stack traces, database errors in development mode
- **Impact**: System architecture disclosure aids attackers
- **Affected Assets**: Application structure, database schema
- **Likelihood**: Medium
- **Severity**: Medium

#### T-I-03: Insecure Password Storage (Historical)
**Threat**: While BCrypt is used, older Rails versions had vulnerabilities.
- **Attack Vector**: Database breach exposing password hashes
- **Impact**: Offline password cracking
- **Affected Assets**: Password digests
- **Likelihood**: Low (BCrypt is strong)
- **Severity**: Critical if breached

#### T-I-04: User Enumeration via Public Profiles
**Threat**: All registered users are publicly discoverable.
- **Attack Vector**: /users endpoint lists all users
- **Impact**: Privacy violation, targeted phishing
- **Affected Assets**: User names, emails
- **Likelihood**: High
- **Severity**: Low to Medium

#### T-I-05: No HTTPS Enforcement
**Threat**: Communication occurs over unencrypted HTTP.
- **Attack Vector**: Man-in-the-middle attacks on network traffic
- **Impact**: Session hijacking, credential theft
- **Affected Assets**: All data in transit
- **Likelihood**: High (if HTTPS not configured)
- **Severity**: Critical

#### T-I-06: Remember Token in Cookie
**Threat**: Session tokens stored in cookies vulnerable to theft.
- **Attack Vector**: XSS attacks, network sniffing (if no HTTPS)
- **Impact**: Session hijacking
- **Affected Assets**: Session tokens, user sessions
- **Likelihood**: Medium
- **Severity**: High

### 8.5 Denial of Service Threats

#### T-D-01: Resource Exhaustion via Pagination Bypass
**Threat**: Attackers request all products/users without pagination limits.
- **Attack Vector**: Directly calling endpoints without page parameters
- **Impact**: Server memory exhaustion, service unavailability
- **Affected Assets**: Server resources
- **Likelihood**: High (acknowledged in README)
- **Severity**: High

#### T-D-02: Micropost Flooding
**Threat**: Users create excessive microposts to exhaust storage.
- **Attack Vector**: Rapid POST requests to /microposts
- **Impact**: Database bloat, performance degradation
- **Affected Assets**: Database storage, application performance
- **Likelihood**: Medium
- **Severity**: Medium

#### T-D-03: Relationship Flooding
**Threat**: Users create excessive follow/unfollow relationships.
- **Attack Vector**: Rapid POST/DELETE to /relationships
- **Impact**: Database performance issues
- **Affected Assets**: Relationships table
- **Likelihood**: Low
- **Severity**: Low to Medium

#### T-D-04: Account Creation Flooding
**Threat**: Automated account creation exhausts resources.
- **Attack Vector**: Automated POST to /signup
- **Impact**: Database bloat, spam accounts
- **Affected Assets**: Users table
- **Likelihood**: High (no CAPTCHA)
- **Severity**: Medium

#### T-D-05: SQL Query Performance
**Threat**: Complex queries on large datasets cause timeouts.
- **Attack Vector**: Feed generation with many followed users
- **Impact**: Slow response times, timeouts
- **Affected Assets**: Application performance
- **Likelihood**: Medium
- **Severity**: Medium

### 8.6 Elevation of Privilege Threats

#### T-E-01: Horizontal Privilege Escalation
**Threat**: Users access other users' private resources.
- **Attack Vector**: Manipulating user IDs in requests
- **Impact**: Unauthorized access to other accounts
- **Affected Assets**: User profiles, microposts
- **Likelihood**: Low (before_action filters in place)
- **Severity**: High

#### T-E-02: Vertical Privilege Escalation to Admin
**Threat**: Regular users gain admin privileges.
- **Attack Vector**: Mass assignment, parameter tampering to set admin flag
- **Impact**: Full system compromise
- **Affected Assets**: All application data and functionality
- **Likelihood**: Low (Strong parameters protect admin flag)
- **Severity**: Critical

#### T-E-03: Insecure Direct Object Reference
**Threat**: Direct access to resources via predictable IDs.
- **Attack Vector**: Incrementing IDs in URLs to access other resources
- **Impact**: Unauthorized data access
- **Affected Assets**: Users, microposts, relationships
- **Likelihood**: Low (authorization checks in place)
- **Severity**: High

#### T-E-04: Admin Account Compromise
**Threat**: Admin account credentials compromised.
- **Attack Vector**: Phishing, credential stuffing, brute force
- **Impact**: Full application control
- **Affected Assets**: Entire application
- **Likelihood**: Low
- **Severity**: Critical

## 9. Countermeasures

### 9.1 Authentication & Access Control Countermeasures

#### C-01: Strengthen Password Policy
**Addresses**: T-S-01
- **Implementation**:
  - Increase minimum password length to 12 characters
  - Enforce password complexity (uppercase, lowercase, numbers, special characters)
  - Implement password strength meter
  - Check against common password dictionaries
- **Priority**: High
- **Effort**: Medium

#### C-02: Implement Multi-Factor Authentication (MFA)
**Addresses**: T-S-01, T-S-02
- **Implementation**:
  - Add TOTP-based 2FA (Google Authenticator, Authy)
  - SMS-based backup codes
  - Recovery codes for account recovery
- **Priority**: High
- **Effort**: High

#### C-03: Rate Limiting on Authentication
**Addresses**: T-S-01, T-D-04
- **Implementation**:
  - Limit login attempts per IP address (5 per 15 minutes)
  - Implement account lockout after failed attempts
  - Add progressive delays after failures
  - Use gems like rack-attack or devise
- **Priority**: High
- **Effort**: Medium

#### C-04: Generic Authentication Error Messages
**Addresses**: T-S-03
- **Implementation**:
  - Return "Invalid email or password" for all authentication failures
  - Avoid indicating whether email exists
  - Same response time for existing/non-existing users
- **Priority**: Medium
- **Effort**: Low

#### C-05: Secure Session Management
**Addresses**: T-S-02, T-I-06
- **Implementation**:
  - Use secure, httpOnly cookies for session tokens
  - Implement session timeout (30 minutes inactivity)
  - Regenerate session tokens on privilege changes
  - Implement logout from all devices feature
- **Priority**: High
- **Effort**: Medium

#### C-06: Authorization Verification
**Addresses**: T-E-01, T-E-03
- **Implementation**:
  - Verify authorization on every protected endpoint
  - Use CanCanCan or Pundit for authorization management
  - Implement resource ownership checks
  - Whitelist approach for access control
- **Priority**: Critical
- **Effort**: Medium

### 9.2 Input Validation & Data Integrity Countermeasures

#### C-07: Comprehensive Input Validation
**Addresses**: T-T-01, T-T-03
- **Implementation**:
  - Validate all user inputs on server side
  - Use strong parameters for all controllers
  - Implement whitelist input validation
  - Sanitize HTML in micropost content
  - Validate email format strictly
  - Limit micropost content length (140-280 characters)
- **Priority**: Critical
- **Effort**: Medium

#### C-08: SQL Injection Prevention
**Addresses**: T-T-01
- **Implementation**:
  - Always use parameterized queries via ActiveRecord
  - Never use raw SQL with user input
  - Escape user input if raw SQL necessary
  - Use database user with minimal privileges
  - Regular security audits of database queries
- **Priority**: Critical
- **Effort**: Low (mostly already implemented)

#### C-09: CSRF Protection
**Addresses**: T-T-02
- **Implementation**:
  - Ensure protect_from_forgery is enabled in ApplicationController
  - Use authenticity_token in all forms
  - Verify CSRF token on state-changing operations
  - Implement SameSite cookie attribute
- **Priority**: High
- **Effort**: Low (Rails provides this)

#### C-10: Output Encoding
**Addresses**: XSS attacks
- **Implementation**:
  - HTML escape all user-generated content
  - Use Rails built-in escaping (erb, <%= %>)
  - Sanitize rich text if implemented
  - Set proper Content-Type headers
  - Implement Content Security Policy (CSP)
- **Priority**: Critical
- **Effort**: Low to Medium

### 9.3 Data Protection Countermeasures

#### C-11: Enforce HTTPS/TLS
**Addresses**: T-I-05, T-I-06
- **Implementation**:
  - Force SSL in production (config.force_ssl = true)
  - Use HSTS headers (Strict-Transport-Security)
  - Redirect all HTTP to HTTPS
  - Use TLS 1.2 or higher
  - Implement certificate pinning for mobile apps
- **Priority**: Critical
- **Effort**: Low to Medium

#### C-12: Secure Cookie Configuration
**Addresses**: T-I-06, T-S-02
- **Implementation**:
  - Set secure flag on cookies (HTTPS only)
  - Set httpOnly flag to prevent JavaScript access
  - Set SameSite attribute to Strict or Lax
  - Use short cookie expiration times
  - Encrypt cookie contents
- **Priority**: High
- **Effort**: Low

#### C-13: Data Encryption at Rest
**Addresses**: T-I-03
- **Implementation**:
  - Encrypt sensitive database fields (emails, personal data)
  - Use database-level encryption for backups
  - Implement key rotation procedures
  - Store encryption keys securely (environment variables, key management service)
- **Priority**: High
- **Effort**: High

#### C-14: Privacy Controls
**Addresses**: T-I-04
- **Implementation**:
  - Add user privacy settings
  - Allow users to hide profiles from public listing
  - Implement private accounts feature
  - GDPR compliance features (data export, deletion)
  - Add opt-in for email communications
- **Priority**: Medium
- **Effort**: High

### 9.4 Denial of Service Countermeasures

#### C-15: Implement Pagination Limits
**Addresses**: T-D-01
- **Implementation**:
  - Enforce maximum items per page (10-50)
  - Remove "fetch all" endpoints
  - Add pagination to all list endpoints
  - Implement cursor-based pagination for large datasets
- **Priority**: High (acknowledged issue)
- **Effort**: Medium

#### C-16: Rate Limiting on Content Creation
**Addresses**: T-D-02, T-D-03, T-D-04
- **Implementation**:
  - Limit microposts per user per hour (10-20)
  - Limit follow/unfollow actions per hour (50)
  - Limit account creation per IP (3 per day)
  - Use rack-attack or similar middleware
- **Priority**: High
- **Effort**: Medium

#### C-17: Resource Quotas
**Addresses**: T-D-02
- **Implementation**:
  - Maximum microposts per user (1000)
  - Maximum relationships per user (5000)
  - Automatic cleanup of old inactive accounts
  - Database query timeout limits
- **Priority**: Medium
- **Effort**: Medium

#### C-18: CAPTCHA on Registration
**Addresses**: T-D-04
- **Implementation**:
  - Add reCAPTCHA to signup form
  - Add CAPTCHA to login after failed attempts
  - Implement invisible CAPTCHA for better UX
- **Priority**: High
- **Effort**: Low to Medium

#### C-19: Query Optimization
**Addresses**: T-D-05
- **Implementation**:
  - Add database indexes on frequently queried fields
  - Optimize feed query with caching
  - Implement pagination on feeds
  - Use eager loading to prevent N+1 queries
  - Consider Redis for caching
- **Priority**: Medium
- **Effort**: Medium to High

### 9.5 Logging & Monitoring Countermeasures

#### C-20: Comprehensive Audit Logging
**Addresses**: T-R-01, T-R-03
- **Implementation**:
  - Log all authentication attempts (success/failure)
  - Log all privilege escalation attempts
  - Log all data modifications with user ID and timestamp
  - Log all admin actions
  - Use structured logging (JSON format)
  - Store logs securely with integrity protection
- **Priority**: High
- **Effort**: Medium

#### C-21: Security Monitoring & Alerting
**Addresses**: T-R-03, T-D-01, T-D-02, T-D-04
- **Implementation**:
  - Monitor failed login attempts
  - Alert on unusual activity patterns
  - Monitor for SQL injection attempts
  - Track API rate limit violations
  - Implement SIEM integration
  - Set up automated incident response
- **Priority**: High
- **Effort**: High

#### C-22: Error Handling & Information Leakage Prevention
**Addresses**: T-I-02
- **Implementation**:
  - Disable detailed error pages in production
  - Custom error pages for 404, 500 errors
  - Log detailed errors server-side only
  - Sanitize error messages shown to users
  - Remove development gems in production
- **Priority**: High
- **Effort**: Low

### 9.6 Dependency & Configuration Countermeasures

#### C-23: Dependency Management
**Addresses**: All threats (infrastructure)
- **Implementation**:
  - Regular dependency updates (bundle update)
  - Monitor for security vulnerabilities (bundler-audit)
  - Remove unused dependencies
  - Pin dependency versions
  - Review dependency security advisories
- **Priority**: Critical
- **Effort**: Ongoing/Low per update

#### C-24: Security Headers
**Addresses**: Various threats
- **Implementation**:
  - X-Frame-Options: DENY (clickjacking protection)
  - X-Content-Type-Options: nosniff
  - X-XSS-Protection: 1; mode=block
  - Content-Security-Policy (CSP)
  - Referrer-Policy: strict-origin-when-cross-origin
  - Use secure_headers gem
- **Priority**: High
- **Effort**: Low

#### C-25: Database Security
**Addresses**: T-T-01, T-I-03
- **Implementation**:
  - Separate database users for different environments
  - Minimal database privileges for application user
  - Encrypt database connections
  - Regular database backups
  - Database access logging
  - Network isolation for database server
- **Priority**: High
- **Effort**: Medium

#### C-26: Secrets Management
**Addresses**: T-E-02, infrastructure
- **Implementation**:
  - Never commit secrets to version control
  - Use environment variables for secrets
  - Use Rails encrypted credentials (Rails 5.2+)
  - Rotate secrets regularly
  - Use key management service (AWS KMS, Vault)
- **Priority**: Critical
- **Effort**: Medium

### 9.7 Admin & Privilege Management Countermeasures

#### C-27: Admin Account Protection
**Addresses**: T-E-04
- **Implementation**:
  - Require MFA for admin accounts
  - Strong password requirements for admins
  - Separate admin interface from user interface
  - Audit all admin actions
  - Limit number of admin accounts
  - Regular review of admin access
- **Priority**: Critical
- **Effort**: Medium

#### C-28: Principle of Least Privilege
**Addresses**: T-E-01, T-E-02
- **Implementation**:
  - Grant minimum necessary permissions
  - Role-based access control (RBAC)
  - Regular access reviews
  - Automatic privilege expiration
  - Approval workflow for privilege elevation
- **Priority**: High
- **Effort**: Medium to High

### 9.8 Development & Deployment Countermeasures

#### C-29: Secure Development Practices
**Addresses**: All threats (preventive)
- **Implementation**:
  - Security training for developers
  - Code review with security focus
  - Static code analysis (Brakeman for Rails)
  - Dynamic security testing (DAST)
  - Penetration testing
  - Bug bounty program
- **Priority**: High
- **Effort**: Ongoing/High

#### C-30: Environment Separation
**Addresses**: Infrastructure security
- **Implementation**:
  - Separate databases for dev/test/production (fix current issue)
  - Different credentials per environment
  - Production data never in development
  - Network isolation for production
  - Separate logging and monitoring per environment
- **Priority**: Critical (acknowledged in README)
- **Effort**: Medium

#### C-31: Incident Response Plan
**Addresses**: All threats (reactive)
- **Implementation**:
  - Document incident response procedures
  - Define roles and responsibilities
  - Establish communication channels
  - Regular incident response drills
  - Post-incident review process
  - Breach notification procedures
- **Priority**: High
- **Effort**: Medium

## 10. Summary & Recommendations

### 10.1 Critical Priorities (Immediate Action Required)
1. **Enforce HTTPS/TLS** (C-11) - Protect all data in transit
2. **Separate Production Database** (C-30) - Fix acknowledged security risk
3. **Implement Rate Limiting** (C-03, C-16) - Prevent abuse and DoS
4. **Secure Secrets Management** (C-26) - Protect sensitive configuration
5. **Update Dependencies** (C-23) - Address known vulnerabilities in Rails 4.0.8 and Ruby 2.0.0

### 10.2 High Priorities (Short Term)
1. **Strengthen Authentication** (C-01, C-02) - Better password policy and MFA
2. **Fix Pagination Issues** (C-15) - Address acknowledged DoS vulnerability
3. **Implement Security Headers** (C-24) - Basic security hardening
4. **Comprehensive Logging** (C-20, C-21) - Enable detection and response
5. **Database Security** (C-25) - Protect data at rest

### 10.3 Medium Priorities (Medium Term)
1. **Privacy Controls** (C-14) - GDPR compliance and user privacy
2. **Admin Protection** (C-27, C-28) - Secure privileged accounts
3. **Data Encryption** (C-13) - Additional layer of protection
4. **Security Monitoring** (C-21) - Proactive threat detection

### 10.4 Ongoing Activities
1. **Security Training** (C-29) - Continuous developer education
2. **Dependency Updates** (C-23) - Regular vulnerability patching
3. **Security Testing** (C-29) - Regular assessments and penetration testing
4. **Access Reviews** - Periodic review of user privileges

### 10.5 Framework Upgrade Recommendation
**Critical**: The application uses Rails 4.0.8 and Ruby 2.0.0, both of which are severely outdated and no longer supported. These versions contain known security vulnerabilities. **Strongly recommend upgrading to Rails 6.x/7.x and Ruby 3.x** as a foundational security improvement.

### 10.6 Compliance Considerations
- **GDPR**: Implement data export, deletion, and privacy controls
- **PCI-DSS**: If payment processing is added, full PCI compliance required
- **OWASP Top 10**: Address identified vulnerabilities aligned with OWASP standards
- **SOC 2**: Implement audit logging and security monitoring for compliance

---

**Document Version**: 1.0  
**Last Updated**: 2024  
**Review Schedule**: Quarterly or after significant application changes  
**Owner**: Security Team  
**Approved By**: [To be completed]
