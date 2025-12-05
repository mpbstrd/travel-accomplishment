# Unit 1: User Management & Authentication - Architecture Design

## 1. Unit Overview

**Business Capability**: Manage user accounts and authenticate users accessing the Travel Accomplishment Report system.

**Responsibilities**:
- User account lifecycle management (create, read, update, delete)
- User authentication (login, logout)
- Session management with timeout
- Password security and management
- Role-based access control (Admin/User)
- Account lockout after failed attempts

**Scope**:
- Foundation unit providing identity and access control for entire system
- All other units depend on this for authentication and authorization
- Manages user identity, credentials, and permissions

**Technology Stack**:
- Platform: C# .NET 8 Windows Forms
- Data Access: Entity Framework Core with SQLite
- Password Hashing: BCrypt.Net or Argon2
- Dependency Injection: Microsoft.Extensions.DependencyInjection

---

## 2. Feature Components

### 2.1 Login Feature
**Responsibility**: Handle user authentication and session creation

**UI Components**:
- LoginForm (Windows Form)
  - Username text box
  - Password text box with visibility toggle
  - Remember me checkbox
  - Login button
  - Error message label

**Feature Operations**:
- Authenticate user credentials
- Create session on successful login
- Track failed login attempts
- Enforce account lockout policy
- Display appropriate error messages

### 2.2 User Management Feature
**Responsibility**: Admin-only user account management

**UI Components**:
- UserManagementForm (Windows Form)
  - User list DataGridView
  - Search text box
  - Filter controls
  - Add User button
  - Edit/Delete action buttons per row
- UserEditorDialog (Modal Dialog)
  - User details form fields
  - Role selection dropdown
  - Account status toggle
  - Save/Cancel buttons
- ChangePasswordDialog (Modal Dialog)
  - Current password field
  - New password field
  - Confirm password field
  - Password strength indicator
  - Save/Cancel buttons

**Feature Operations**:
- List all users with search/filter
- Create new user accounts
- Edit existing user details
- Soft delete user accounts
- Reset user passwords
- Validate admin permissions

### 2.3 Session Management Feature
**Responsibility**: Manage active user sessions and timeouts

**UI Components**:
- SessionTimeoutWarningDialog (Modal Dialog)
  - Countdown timer display
  - Extend Session button
  - Logout button

**Feature Operations**:
- Create and store sessions in database
- Validate session tokens
- Track session activity
- Enforce 30-minute timeout
- Display timeout warnings
- Extend session on user activity
- Invalidate sessions on logout

### 2.4 Profile Management Feature
**Responsibility**: Allow users to manage their own profile

**UI Components**:
- UserProfileForm (Windows Form)
  - Display user information
  - Change password button
  - Update profile button

**Feature Operations**:
- View own profile information
- Change own password
- Update profile details (limited fields)

---

## 3. Business Services

### 3.1 AuthenticationService
**Responsibility**: Handle user authentication and session management

**Public Operations**:
- AuthenticateUser(username, password) → UserSession
  - Validates credentials against database
  - Checks account status (active/inactive)
  - Checks account lockout status
  - Creates session on success
  - Records login attempt
  - Returns session token and user details
  
- ValidateSession(sessionToken) → UserDetails
  - Validates session exists and is active
  - Checks session expiration
  - Updates last activity timestamp
  - Returns user details if valid
  
- LogoutUser(sessionToken) → bool
  - Invalidates session
  - Records logout event
  
- ExtendSession(sessionToken) → bool
  - Extends session expiration time
  - Updates last activity timestamp

**Dependencies**:
- IUserRepository
- IUserSessionRepository
- ILoginAttemptRepository
- IPasswordHasher
- IAuditLogService (Unit 4)

### 3.2 UserManagementService
**Responsibility**: Manage user account lifecycle

**Public Operations**:
- GetUserById(userId) → UserDetails
  - Retrieves user by ID
  - Returns null if not found or deleted
  
- GetUserByUsername(username) → UserDetails
  - Retrieves user by username
  
- GetAllUsers(includeInactive) → List<UserDetails>
  - Returns all users
  - Optionally includes inactive accounts
  
- SearchUsers(searchTerm, filters) → List<UserDetails>
  - Searches users by name, username, email
  - Applies filters (role, status)
  
- CreateUser(userCreateRequest) → int (userId)
  - Validates uniqueness (username, email)
  - Validates password requirements
  - Hashes password
  - Creates user record
  - Records creation in audit log
  
- UpdateUser(userId, userUpdateRequest) → bool
  - Validates admin permission
  - Updates user details
  - Records changes in audit log
  
- DeleteUser(userId, deletedBy) → bool
  - Soft deletes user (sets IsDeleted flag)
  - Checks for pending reports/signatures
  - Records deletion in audit log
  
- ResetPassword(userId, newPassword, resetBy) → bool
  - Validates password requirements
  - Checks password history
  - Hashes new password
  - Records password change

**Dependencies**:
- IUserRepository
- IPasswordHistoryRepository
- IPasswordHasher
- IPasswordValidator
- IAuditLogService (Unit 4)

### 3.3 AuthorizationService
**Responsibility**: Enforce role-based access control

**Public Operations**:
- CheckPermission(userId, action) → bool
  - Validates user has permission for action
  - Actions: CreateReport, EditReport, DeleteReport, ManageUsers, ViewAuditLogs, etc.
  
- GetUserRole(userId) → Role
  - Returns user's role (Admin or User)
  
- IsAdmin(userId) → bool
  - Quick check if user is admin
  
- CanAccessFeature(userId, featureName) → bool
  - Checks if user can access specific feature

**Dependencies**:
- IUserRepository

### 3.4 PasswordService
**Responsibility**: Handle password security operations

**Public Operations**:
- HashPassword(plainPassword) → string
  - Hashes password using BCrypt or Argon2
  
- VerifyPassword(plainPassword, hashedPassword) → bool
  - Verifies password against hash
  
- ValidatePasswordRequirements(password) → ValidationResult
  - Checks minimum length (8 characters)
  - Checks for uppercase letter
  - Checks for lowercase letter
  - Checks for number
  - Returns validation result with messages
  
- CheckPasswordHistory(userId, newPassword) → bool
  - Checks if password was used in last 3 passwords
  
- RecordPasswordChange(userId, passwordHash) → void
  - Stores password in history table

**Dependencies**:
- IPasswordHistoryRepository
- BCrypt.Net or Argon2 library

### 3.5 AccountLockoutService
**Responsibility**: Manage account lockout after failed attempts

**Public Operations**:
- RecordFailedAttempt(username) → void
  - Records failed login attempt
  - Checks if lockout threshold reached (5 attempts)
  - Locks account if threshold exceeded
  
- IsAccountLocked(username) → bool
  - Checks if account is currently locked
  - Checks if lockout period expired (15 minutes)
  
- UnlockAccount(username) → void
  - Manually unlocks account (admin action)
  - Clears failed attempt counter
  
- GetFailedAttemptCount(username) → int
  - Returns current failed attempt count
  
- ClearFailedAttempts(username) → void
  - Resets failed attempt counter (after successful login)

**Dependencies**:
- ILoginAttemptRepository

---

## 4. Data Models

### 4.1 User Entity
**Maps to**: Users table

**Properties**:
- UserId (int, primary key)
- Username (string, unique, required, max 50)
- FullName (string, required, max 100)
- Email (string, unique, required, max 100)
- PasswordHash (string, required, max 255)
- Role (string, required, max 20) - "Admin" or "User"
- AccountStatus (string, required, max 20) - "Active" or "Inactive"
- CreatedDate (DateTime, required)
- CreatedBy (int, foreign key to Users)
- LastModifiedDate (DateTime, nullable)
- LastModifiedBy (int, foreign key to Users, nullable)
- LastLoginDate (DateTime, nullable)
- IsDeleted (bool, required, default false)
- DeletedDate (DateTime, nullable)
- DeletedBy (int, foreign key to Users, nullable)

**Relationships**:
- CreatedByUser (User) - navigation property
- LastModifiedByUser (User) - navigation property
- DeletedByUser (User) - navigation property
- UserSessions (collection of UserSession)
- PasswordHistories (collection of PasswordHistory)

**Validation Rules**:
- Username: 3-50 characters, alphanumeric only
- FullName: 2-100 characters, letters and spaces
- Email: valid email format, max 100 characters
- Role: must be "Admin" or "User"
- AccountStatus: must be "Active" or "Inactive"

### 4.2 UserSession Entity
**Maps to**: UserSessions table

**Properties**:
- SessionId (int, primary key)
- UserId (int, foreign key, required)
- SessionToken (string, unique, required, max 255)
- CreatedDate (DateTime, required)
- ExpiresAt (DateTime, required)
- LastActivityDate (DateTime, required)
- IsActive (bool, required, default true)
- IPAddress (string, nullable, max 50)
- UserAgent (string, nullable, max 255)

**Relationships**:
- User (User) - navigation property

**Business Logic**:
- Session expires after 30 minutes of inactivity
- SessionToken is cryptographically secure random value
- LastActivityDate updated on each request

### 4.3 LoginAttempt Entity
**Maps to**: LoginAttempts table

**Properties**:
- AttemptId (int, primary key)
- Username (string, required, max 50)
- AttemptDate (DateTime, required)
- IsSuccessful (bool, required)
- IPAddress (string, nullable, max 50)
- FailureReason (string, nullable, max 255)

**Business Logic**:
- Records all login attempts (success and failure)
- Used for account lockout enforcement
- Used for security auditing

### 4.4 PasswordHistory Entity
**Maps to**: PasswordHistory table

**Properties**:
- HistoryId (int, primary key)
- UserId (int, foreign key, required)
- PasswordHash (string, required, max 255)
- ChangedDate (DateTime, required)

**Relationships**:
- User (User) - navigation property

**Business Logic**:
- Stores last 3 passwords per user
- Prevents password reuse
- Older entries automatically removed

---

## 5. Repository Pattern

### 5.1 IUserRepository Interface
**Responsibility**: Data access for User entity

**Operations**:
- GetByIdAsync(userId) → User
- GetByUsernameAsync(username) → User
- GetByEmailAsync(email) → User
- GetAllAsync(includeDeleted) → List<User>
- SearchAsync(searchTerm, filters) → List<User>
- CreateAsync(user) → User
- UpdateAsync(user) → User
- DeleteAsync(userId) → bool
- ExistsAsync(username, email) → bool

**Implementation**: UserRepository (EF Core)
- Uses DbContext for database operations
- Implements async operations
- Handles soft delete filtering
- Includes related entities when needed

### 5.2 IUserSessionRepository Interface
**Responsibility**: Data access for UserSession entity

**Operations**:
- GetByTokenAsync(sessionToken) → UserSession
- GetActiveSessionsByUserAsync(userId) → List<UserSession>
- CreateAsync(session) → UserSession
- UpdateAsync(session) → UserSession
- InvalidateAsync(sessionToken) → bool
- InvalidateAllUserSessionsAsync(userId) → bool
- CleanupExpiredSessionsAsync() → int

**Implementation**: UserSessionRepository (EF Core)
- Manages session lifecycle
- Handles session expiration
- Cleanup of expired sessions

### 5.3 ILoginAttemptRepository Interface
**Responsibility**: Data access for LoginAttempt entity

**Operations**:
- CreateAsync(attempt) → LoginAttempt
- GetRecentAttemptsByUsernameAsync(username, minutes) → List<LoginAttempt>
- GetFailedAttemptCountAsync(username, minutes) → int
- ClearAttemptsAsync(username) → bool

**Implementation**: LoginAttemptRepository (EF Core)
- Tracks login attempts
- Supports account lockout logic
- Cleanup of old attempts

### 5.4 IPasswordHistoryRepository Interface
**Responsibility**: Data access for PasswordHistory entity

**Operations**:
- CreateAsync(history) → PasswordHistory
- GetRecentPasswordsAsync(userId, count) → List<PasswordHistory>
- CleanupOldPasswordsAsync(userId, keepCount) → int

**Implementation**: PasswordHistoryRepository (EF Core)
- Maintains password history
- Enforces history limit (last 3)
- Cleanup of old passwords

---

## 6. Service Interfaces (Exposed to Other Units)

### 6.1 IAuthenticationService Interface
**Exposed Operations**:
- AuthenticateUserAsync(username, password) → AuthenticationResult
- ValidateSessionAsync(sessionToken) → SessionValidationResult
- LogoutUserAsync(sessionToken) → bool
- ExtendSessionAsync(sessionToken) → bool

**Data Contracts**:

**AuthenticationResult**:
- Success (bool)
- SessionToken (string)
- UserId (int)
- Username (string)
- FullName (string)
- Role (string)
- ExpiresAt (DateTime)
- ErrorMessage (string)

**SessionValidationResult**:
- IsValid (bool)
- UserId (int)
- Username (string)
- FullName (string)
- Role (string)
- ErrorMessage (string)

### 6.2 IUserManagementService Interface
**Exposed Operations**:
- GetUserByIdAsync(userId) → UserDto
- GetUserByUsernameAsync(username) → UserDto
- GetAllUsersAsync(includeInactive) → List<UserDto>
- CreateUserAsync(createRequest) → int
- UpdateUserAsync(userId, updateRequest) → bool
- DeleteUserAsync(userId, deletedBy) → bool
- ResetPasswordAsync(userId, newPassword, resetBy) → bool

**Data Contracts**:

**UserDto**:
- UserId (int)
- Username (string)
- FullName (string)
- Email (string)
- Role (string)
- AccountStatus (string)
- LastLoginDate (DateTime?)
- CreatedDate (DateTime)

**UserCreateRequest**:
- Username (string)
- FullName (string)
- Email (string)
- Password (string)
- Role (string)
- CreatedBy (int)

**UserUpdateRequest**:
- FullName (string)
- Email (string)
- Role (string)
- AccountStatus (string)
- ModifiedBy (int)

### 6.3 IAuthorizationService Interface
**Exposed Operations**:
- CheckPermissionAsync(userId, action) → bool
- GetUserRoleAsync(userId) → string
- IsAdminAsync(userId) → bool
- CanAccessFeatureAsync(userId, featureName) → bool

**Permission Actions**:
- CreateReport
- EditOwnReport
- EditAnyReport
- DeleteReport
- ManageUsers
- ViewAuditLogs
- SystemConfiguration
- BackupRestore
- ViewStatistics

---

## 7. Dependencies

### 7.1 Internal Dependencies
- Entity Framework Core (data access)
- BCrypt.Net or Argon2 (password hashing)
- Microsoft.Extensions.DependencyInjection (DI container)
- System.Security.Cryptography (session token generation)

### 7.2 External Unit Dependencies
**Depends On**:
- Unit 4 (System Administration): IAuditLogService for logging authentication events

**Consumed By**:
- Unit 2 (Report Lifecycle Management): User validation, permission checks
- Unit 3 (Signature Workflow Engine): User authentication for signatures
- Unit 4 (System Administration): User information for audit logs

### 7.3 Database Dependencies
**Tables Owned**:
- Users
- UserSessions
- LoginAttempts
- PasswordHistory

**Foreign Key References**:
- Users.CreatedBy → Users.UserId
- Users.LastModifiedBy → Users.UserId
- Users.DeletedBy → Users.UserId
- UserSessions.UserId → Users.UserId
- PasswordHistory.UserId → Users.UserId

---

## 8. Component Diagram

```
┌─────────────────────────────────────────────────────────────────┐
│                    Unit 1: User Management & Authentication      │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  ┌──────────────────────────────────────────────────────────┐  │
│  │              UI Layer (Windows Forms)                     │  │
│  │  ┌──────────┐  ┌──────────────┐  ┌──────────────────┐   │  │
│  │  │ LoginForm│  │UserMgmtForm  │  │SessionTimeout    │   │  │
│  │  │          │  │              │  │WarningDialog     │   │  │
│  │  └────┬─────┘  └──────┬───────┘  └────────┬─────────┘   │  │
│  └───────┼────────────────┼──────────────────┼─────────────┘  │
│          │                │                  │                 │
│  ┌───────▼────────────────▼──────────────────▼─────────────┐  │
│  │              Business Services Layer                      │  │
│  │  ┌──────────────────┐  ┌──────────────────┐             │  │
│  │  │Authentication    │  │UserManagement    │             │  │
│  │  │Service           │  │Service           │             │  │
│  │  └────────┬─────────┘  └────────┬─────────┘             │  │
│  │           │                     │                        │  │
│  │  ┌────────▼─────────┐  ┌───────▼──────────┐            │  │
│  │  │Authorization     │  │Password          │            │  │
│  │  │Service           │  │Service           │            │  │
│  │  └────────┬─────────┘  └───────┬──────────┘            │  │
│  │           │                     │                        │  │
│  │  ┌────────▼─────────────────────▼──────────┐            │  │
│  │  │AccountLockoutService                     │            │  │
│  │  └──────────────────┬───────────────────────┘            │  │
│  └─────────────────────┼──────────────────────────────────┘  │
│                        │                                      │
│  ┌─────────────────────▼──────────────────────────────────┐  │
│  │              Repository Layer                            │  │
│  │  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐  │  │
│  │  │User          │  │UserSession   │  │LoginAttempt  │  │  │
│  │  │Repository    │  │Repository    │  │Repository    │  │  │
│  │  └──────┬───────┘  └──────┬───────┘  └──────┬───────┘  │  │
│  │         │                 │                 │           │  │
│  │  ┌──────▼─────────────────▼─────────────────▼───────┐  │  │
│  │  │PasswordHistoryRepository                          │  │  │
│  │  └──────────────────────┬────────────────────────────┘  │  │
│  └─────────────────────────┼───────────────────────────────┘  │
│                            │                                   │
│  ┌─────────────────────────▼───────────────────────────────┐  │
│  │              Data Access Layer (EF Core)                 │  │
│  │  ┌──────────────────────────────────────────────────┐   │  │
│  │  │            DbContext (SQLite)                     │   │  │
│  │  │  - Users                                          │   │  │
│  │  │  - UserSessions                                   │   │  │
│  │  │  - LoginAttempts                                  │   │  │
│  │  │  - PasswordHistory                                │   │  │
│  │  └──────────────────────────────────────────────────┘   │  │
│  └──────────────────────────────────────────────────────────┘  │
│                                                                  │
│  ┌──────────────────────────────────────────────────────────┐  │
│  │         Service Interfaces (Exposed to Other Units)       │  │
│  │  - IAuthenticationService                                 │  │
│  │  - IUserManagementService                                 │  │
│  │  - IAuthorizationService                                  │  │
│  └──────────────────────────────────────────────────────────┘  │
└─────────────────────────────────────────────────────────────────┘
```

---

## 9. Database Schema

### 9.1 Users Table
```
Table: Users
Primary Key: UserId (INT, IDENTITY)
Indexes:
  - IX_Users_Username (UNIQUE)
  - IX_Users_Email (UNIQUE)
  - IX_Users_IsDeleted

Columns:
  UserId              INT             PRIMARY KEY IDENTITY
  Username            NVARCHAR(50)    NOT NULL UNIQUE
  FullName            NVARCHAR(100)   NOT NULL
  Email               NVARCHAR(100)   NOT NULL UNIQUE
  PasswordHash        NVARCHAR(255)   NOT NULL
  Role                NVARCHAR(20)    NOT NULL CHECK (Role IN ('Admin', 'User'))
  AccountStatus       NVARCHAR(20)    NOT NULL CHECK (AccountStatus IN ('Active', 'Inactive'))
  CreatedDate         DATETIME        NOT NULL DEFAULT CURRENT_TIMESTAMP
  CreatedBy           INT             NULL FOREIGN KEY REFERENCES Users(UserId)
  LastModifiedDate    DATETIME        NULL
  LastModifiedBy      INT             NULL FOREIGN KEY REFERENCES Users(UserId)
  LastLoginDate       DATETIME        NULL
  IsDeleted           BIT             NOT NULL DEFAULT 0
  DeletedDate         DATETIME        NULL
  DeletedBy           INT             NULL FOREIGN KEY REFERENCES Users(UserId)
```

### 9.2 UserSessions Table
```
Table: UserSessions
Primary Key: SessionId (INT, IDENTITY)
Indexes:
  - IX_UserSessions_SessionToken (UNIQUE)
  - IX_UserSessions_UserId
  - IX_UserSessions_ExpiresAt
  - IX_UserSessions_IsActive

Columns:
  SessionId           INT             PRIMARY KEY IDENTITY
  UserId              INT             NOT NULL FOREIGN KEY REFERENCES Users(UserId)
  SessionToken        NVARCHAR(255)   NOT NULL UNIQUE
  CreatedDate         DATETIME        NOT NULL DEFAULT CURRENT_TIMESTAMP
  ExpiresAt           DATETIME        NOT NULL
  LastActivityDate    DATETIME        NOT NULL
  IsActive            BIT             NOT NULL DEFAULT 1
  IPAddress           NVARCHAR(50)    NULL
  UserAgent           NVARCHAR(255)   NULL
```

### 9.3 LoginAttempts Table
```
Table: LoginAttempts
Primary Key: AttemptId (INT, IDENTITY)
Indexes:
  - IX_LoginAttempts_Username
  - IX_LoginAttempts_AttemptDate

Columns:
  AttemptId           INT             PRIMARY KEY IDENTITY
  Username            NVARCHAR(50)    NOT NULL
  AttemptDate         DATETIME        NOT NULL DEFAULT CURRENT_TIMESTAMP
  IsSuccessful        BIT             NOT NULL
  IPAddress           NVARCHAR(50)    NULL
  FailureReason       NVARCHAR(255)   NULL
```

### 9.4 PasswordHistory Table
```
Table: PasswordHistory
Primary Key: HistoryId (INT, IDENTITY)
Indexes:
  - IX_PasswordHistory_UserId
  - IX_PasswordHistory_ChangedDate

Columns:
  HistoryId           INT             PRIMARY KEY IDENTITY
  UserId              INT             NOT NULL FOREIGN KEY REFERENCES Users(UserId)
  PasswordHash        NVARCHAR(255)   NOT NULL
  ChangedDate         DATETIME        NOT NULL DEFAULT CURRENT_TIMESTAMP
```

### 9.5 Relationships
- Users.CreatedBy → Users.UserId (self-referencing)
- Users.LastModifiedBy → Users.UserId (self-referencing)
- Users.DeletedBy → Users.UserId (self-referencing)
- UserSessions.UserId → Users.UserId (one-to-many)
- PasswordHistory.UserId → Users.UserId (one-to-many)

---

## 10. Implementation Notes

### 10.1 Entity Framework Core Configuration
- Use Code-First approach with migrations
- Configure entity relationships in DbContext OnModelCreating
- Implement soft delete global query filter for Users
- Use async operations throughout
- Configure indexes for performance
- Set up cascade delete rules appropriately

### 10.2 Password Security
- Use BCrypt.Net-Next NuGet package (recommended) or Argon2
- BCrypt work factor: 12 (balance between security and performance)
- Never store plain text passwords
- Password validation before hashing
- Implement password strength indicator in UI

### 10.3 Session Management
- Generate session tokens using cryptographically secure random generator
- Session token length: 64 characters (base64 encoded)
- Store sessions in database for persistence
- Implement background job to cleanup expired sessions
- Update LastActivityDate on each authenticated request
- Session timeout: 30 minutes (configurable via Unit 4)

### 10.4 Account Lockout
- Lockout threshold: 5 failed attempts
- Lockout duration: 15 minutes
- Track attempts by username (not user ID)
- Clear attempts on successful login
- Admin can manually unlock accounts
- Consider IP-based lockout for additional security

### 10.5 Dependency Injection Setup
- Register services with appropriate lifetime:
  - Scoped: Services, Repositories, DbContext
  - Singleton: Configuration, Logging
  - Transient: Validators, Helpers
- Use constructor injection throughout
- Register interfaces with implementations
- Configure in Program.cs or Startup class

### 10.6 Error Handling
- Use try-catch blocks in service methods
- Return meaningful error messages to UI
- Log exceptions to audit log (Unit 4)
- Never expose sensitive information in error messages
- Validate inputs at service layer
- Use custom exception types for business rule violations

### 10.7 Validation
- Implement validation at multiple layers:
  - UI: Immediate feedback (TextBox validation)
  - Service: Business rule validation
  - Entity: Data annotation attributes
- Use FluentValidation library (optional) for complex validation
- Return validation results with specific error messages
- Validate before database operations

### 10.8 Testing Considerations
- Unit test services with mocked repositories
- Integration test repositories with in-memory SQLite
- Test authentication flows end-to-end
- Test account lockout scenarios
- Test session timeout behavior
- Test password validation rules
- Mock IAuditLogService for unit tests

### 10.9 Performance Considerations
- Use async/await throughout
- Implement connection pooling (EF Core default)
- Add indexes on frequently queried columns
- Use pagination for user lists
- Cache user roles/permissions (optional)
- Cleanup expired sessions regularly (background job)
- Limit password history to last 3 entries

### 10.10 Security Best Practices
- Never log passwords (plain or hashed)
- Use parameterized queries (EF Core default)
- Validate all inputs
- Implement HTTPS for production (desktop app consideration)
- Secure session tokens
- Implement CSRF protection if web-based
- Regular security audits
- Keep dependencies updated

---

## 11. Integration Points

### 11.1 With Unit 2 (Report Lifecycle Management)
- Unit 2 calls IAuthenticationService.ValidateSessionAsync() before operations
- Unit 2 calls IAuthorizationService.CheckPermissionAsync() for permission checks
- Unit 2 calls IUserManagementService.GetUserByIdAsync() for user details
- Unit 1 provides user context for report creation (CreatedBy field)

### 11.2 With Unit 3 (Signature Workflow Engine)
- Unit 3 calls IAuthenticationService.ValidateSessionAsync() before signature operations
- Unit 3 calls IUserManagementService.GetUserByIdAsync() for signatory details
- Unit 3 calls IAuthorizationService.CheckPermissionAsync() for signature permissions

### 11.3 With Unit 4 (System Administration)
- Unit 1 calls IAuditLogService.LogActionAsync() for:
  - Login attempts (success/failure)
  - User account creation
  - User account modification
  - User account deletion
  - Password changes
  - Session creation/termination
- Unit 4 calls IUserManagementService.GetUserByIdAsync() for audit log user details

### 11.4 Application Startup
- Initialize DbContext with SQLite connection
- Run EF Core migrations
- Register all services in DI container
- Create default admin account if none exists
- Start session cleanup background job

---

## 12. Build Order and Dependencies

### 12.1 Build Order
1. Data Models (Entities)
2. Repository Interfaces
3. Repository Implementations
4. Service Interfaces
5. Service Implementations
6. UI Components
7. Integration Testing

### 12.2 Critical Path
- This is the foundation unit - must be built first
- All other units depend on authentication and authorization
- Database schema must be created before other units
- Service interfaces must be stable before other units integrate

---

## 13. Future Enhancements

### 13.1 Potential Improvements
- Multi-factor authentication (MFA)
- Password expiration policy
- Single sign-on (SSO) integration
- OAuth/OpenID Connect support
- Biometric authentication
- Session management dashboard for admins
- User activity tracking
- Advanced password policies (dictionary check, complexity rules)
- Account recovery via email
- User profile pictures

### 13.2 Scalability Considerations
- Move to distributed session storage (Redis) if needed
- Implement caching layer for user permissions
- Consider separate authentication service if system grows
- Implement rate limiting for login attempts
- Add load balancing support

---

**Document Version**: 1.0  
**Date**: December 5, 2025  
**Status**: Ready for Implementation  
**Technology**: C# .NET 8, Windows Forms, Entity Framework Core, SQLite
