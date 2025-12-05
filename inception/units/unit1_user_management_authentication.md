# Unit 1: User Management & Authentication

## Unit Overview

**Business Capability**: Manage user accounts and authenticate users accessing the Travel Accomplishment Report system.

**Responsibility**: This unit handles all aspects of user identity, authentication, authorization, and account lifecycle management. It provides the foundation for secure access to the system.

**Scope**:
- User account creation, editing, and deletion (Admin only)
- User authentication (login/logout)
- Password management and security
- Session management
- Role-based access control (Admin vs User roles)

---

## User Stories

### US-1.1: Admin Account Creation
**As an** Admin  
**I want to** create new user accounts  
**So that** NISD personnel can access the system

**Acceptance Criteria:**
- Admin can access account creation interface
- Required fields: Username, Full Name, Email, Role (Admin/User), Password
- Email validation enforced
- Password must meet security requirements (min 8 characters, alphanumeric)
- System prevents duplicate usernames/emails
- Success message displayed upon account creation
- New user receives account credentials
- Account is immediately active unless specified otherwise

---

### US-1.2: Admin Account Editing
**As an** Admin  
**I want to** edit existing user accounts  
**So that** I can update user information and permissions

**Acceptance Criteria:**
- Admin can search and select user accounts to edit
- Editable fields: Full Name, Email, Role, Account Status (Active/Inactive)
- Password reset option available
- Changes are logged with timestamp and admin identifier
- User is notified of account changes
- System prevents admin from deleting their own account while logged in

---

### US-1.3: Admin Account Deletion
**As an** Admin  
**I want to** delete user accounts  
**So that** I can remove access for personnel who no longer require it

**Acceptance Criteria:**
- Admin can select user accounts for deletion
- Confirmation dialog displayed before deletion
- System shows warning if user has pending reports/signatures
- Deleted accounts are archived (soft delete) not permanently removed
- Reports created by deleted users remain accessible
- Deletion action is logged with timestamp and admin identifier

---

### US-1.4: User Login
**As a** User  
**I want to** log into the system securely  
**So that** I can access my reports and perform my duties

**Acceptance Criteria:**
- Login screen with username and password fields
- Authentication validates credentials against database
- Failed login attempts are tracked (max 5 attempts before temporary lockout)
- Successful login redirects to dashboard
- Session timeout after 30 minutes of inactivity
- "Remember Me" option for trusted devices
- Password visibility toggle available

---

### US-8.1: Role-Based Access Control
**As the** System  
**I want to** enforce role-based permissions  
**So that** users can only access authorized features

**Acceptance Criteria:**
- **Admin Permissions:**
  - Full access to all features
  - Account management
  - Report editing and deletion
  - System configuration
  - Audit logs
  - Statistics and analytics
- **User Permissions:**
  - Create and submit reports
  - View own reports
  - View reports requiring their signature
  - Sign reports as designated signatory
  - Update own profile
- Permission checks enforced at:
  - UI level (hide/disable unauthorized features)
  - Business logic level (reject unauthorized operations)
  - Database level (row-level security where applicable)
- Unauthorized access attempts logged
- Clear error messages for permission denials

---

### US-8.2: Secure Password Management
**As a** User  
**I want to** manage my password securely  
**So that** my account remains protected

**Acceptance Criteria:**
- Password requirements enforced:
  - Minimum 8 characters
  - At least one uppercase letter
  - At least one lowercase letter
  - At least one number
  - At least one special character (optional but recommended)
- Password strength indicator during creation/change
- Passwords hashed using industry-standard algorithm (bcrypt/Argon2)
- Password change functionality in user profile
- Current password required to set new password
- Password history maintained (cannot reuse last 3 passwords)
- Password reset option available (Admin can reset user passwords)
- Account lockout after 5 failed login attempts (15-minute lockout)

---

### US-8.3: Session Management
**As the** System  
**I want to** manage user sessions securely  
**So that** unauthorized access is prevented

**Acceptance Criteria:**
- Session timeout after 30 minutes of inactivity
- Warning displayed 2 minutes before timeout
- Option to extend session before timeout
- Secure session tokens
- Session invalidated on logout
- Only one active session per user (configurable)
- Session data encrypted
- Session hijacking protection
- "Remember Me" creates extended session (7 days) with secure storage

---

## Database Tables Owned by This Unit

### Users Table
| Column | Type | Constraints | Description |
|--------|------|-------------|-------------|
| UserID | INT | PRIMARY KEY, IDENTITY | Unique user identifier |
| Username | NVARCHAR(50) | UNIQUE, NOT NULL | Login username |
| FullName | NVARCHAR(100) | NOT NULL | User's full name |
| Email | NVARCHAR(100) | UNIQUE, NOT NULL | User's email address |
| PasswordHash | NVARCHAR(255) | NOT NULL | Hashed password |
| Role | NVARCHAR(20) | NOT NULL | Admin or User |
| AccountStatus | NVARCHAR(20) | NOT NULL | Active or Inactive |
| CreatedDate | DATETIME | NOT NULL, DEFAULT GETDATE() | Account creation timestamp |
| CreatedBy | INT | FOREIGN KEY (Users.UserID) | Admin who created account |
| LastModifiedDate | DATETIME | NULL | Last modification timestamp |
| LastModifiedBy | INT | FOREIGN KEY (Users.UserID) | Admin who last modified |
| LastLoginDate | DATETIME | NULL | Last successful login |
| IsDeleted | BIT | NOT NULL, DEFAULT 0 | Soft delete flag |
| DeletedDate | DATETIME | NULL | Deletion timestamp |
| DeletedBy | INT | FOREIGN KEY (Users.UserID) | Admin who deleted account |

### UserSessions Table
| Column | Type | Constraints | Description |
|--------|------|-------------|-------------|
| SessionID | INT | PRIMARY KEY, IDENTITY | Unique session identifier |
| UserID | INT | FOREIGN KEY (Users.UserID), NOT NULL | User associated with session |
| SessionToken | NVARCHAR(255) | UNIQUE, NOT NULL | Encrypted session token |
| CreatedDate | DATETIME | NOT NULL, DEFAULT GETDATE() | Session start time |
| ExpiresAt | DATETIME | NOT NULL | Session expiration time |
| LastActivityDate | DATETIME | NOT NULL | Last user activity |
| IsActive | BIT | NOT NULL, DEFAULT 1 | Session active flag |
| IPAddress | NVARCHAR(50) | NULL | User's IP address |
| UserAgent | NVARCHAR(255) | NULL | Browser/device info |

### LoginAttempts Table
| Column | Type | Constraints | Description |
|--------|------|-------------|-------------|
| AttemptID | INT | PRIMARY KEY, IDENTITY | Unique attempt identifier |
| Username | NVARCHAR(50) | NOT NULL | Attempted username |
| AttemptDate | DATETIME | NOT NULL, DEFAULT GETDATE() | Attempt timestamp |
| IsSuccessful | BIT | NOT NULL | Success/failure flag |
| IPAddress | NVARCHAR(50) | NULL | Source IP address |
| FailureReason | NVARCHAR(255) | NULL | Reason for failure |

### PasswordHistory Table
| Column | Type | Constraints | Description |
|--------|------|-------------|-------------|
| HistoryID | INT | PRIMARY KEY, IDENTITY | Unique history identifier |
| UserID | INT | FOREIGN KEY (Users.UserID), NOT NULL | User identifier |
| PasswordHash | NVARCHAR(255) | NOT NULL | Previous password hash |
| ChangedDate | DATETIME | NOT NULL, DEFAULT GETDATE() | Password change date |

---

## Service Interfaces Exposed

This unit exposes the following operations to other units:

### Authentication Operations
- **AuthenticateUser**(username, password) → UserSession
  - Validates credentials and creates session
  - Returns session token and user details
  
- **ValidateSession**(sessionToken) → UserDetails
  - Validates active session
  - Returns user information if valid
  
- **LogoutUser**(sessionToken) → Success
  - Invalidates user session
  
- **ExtendSession**(sessionToken) → Success
  - Extends session expiration time

### User Management Operations
- **GetUserById**(userId) → UserDetails
  - Retrieves user information
  
- **GetUserByUsername**(username) → UserDetails
  - Retrieves user by username
  
- **CreateUser**(userDetails) → UserId
  - Creates new user account (Admin only)
  
- **UpdateUser**(userId, userDetails) → Success
  - Updates user information (Admin only)
  
- **DeleteUser**(userId) → Success
  - Soft deletes user account (Admin only)
  
- **ResetPassword**(userId, newPassword) → Success
  - Resets user password (Admin only)

### Authorization Operations
- **CheckPermission**(userId, action) → Boolean
  - Checks if user has permission for action
  
- **GetUserRole**(userId) → Role
  - Returns user's role (Admin/User)
  
- **IsAdmin**(userId) → Boolean
  - Checks if user is admin

---

## Dependencies on Other Units

### Depends On:
- **System Administration Unit**: For audit logging of authentication events

### Consumed By:
- **Report Lifecycle Management Unit**: User validation, permission checks
- **Signature Workflow Engine Unit**: User authentication for signatures
- **System Administration Unit**: User information for audit logs

---

## Business Rules

1. **Username Uniqueness**: Usernames must be unique across the system
2. **Email Uniqueness**: Email addresses must be unique across the system
3. **Password Complexity**: Passwords must meet minimum security requirements
4. **Password History**: Users cannot reuse their last 3 passwords
5. **Account Lockout**: After 5 failed login attempts, account is locked for 15 minutes
6. **Session Timeout**: Sessions expire after 30 minutes of inactivity
7. **Soft Delete**: Deleted accounts are archived, not permanently removed
8. **Admin Protection**: Admins cannot delete their own account while logged in
9. **Role Enforcement**: Only Admins can manage user accounts
10. **Active Account Requirement**: Only active accounts can log in

---

## UI Components (Desktop App)

### Screens/Forms:
1. **Login Screen**
   - Username and password fields
   - Remember me checkbox
   - Login button
   - Password visibility toggle

2. **User Management Screen** (Admin only)
   - User list/grid
   - Search and filter controls
   - Add user button
   - Edit/Delete actions per user

3. **Create/Edit User Dialog**
   - User details form
   - Role selection
   - Account status toggle
   - Save/Cancel buttons

4. **Change Password Dialog**
   - Current password field
   - New password field
   - Confirm password field
   - Password strength indicator
   - Save/Cancel buttons

5. **Session Timeout Warning Dialog**
   - Countdown timer
   - Extend session button
   - Logout button

---

## Validation Rules

### Username:
- Required
- 3-50 characters
- Alphanumeric only (no spaces)
- Must be unique

### Full Name:
- Required
- 2-100 characters
- Letters and spaces only

### Email:
- Required
- Valid email format
- Must be unique
- Max 100 characters

### Password:
- Required
- Minimum 8 characters
- At least one uppercase letter
- At least one lowercase letter
- At least one number
- Special characters recommended

### Role:
- Required
- Must be either "Admin" or "User"

---

## Error Handling

### Common Errors:
- **Invalid Credentials**: "Invalid username or password"
- **Account Locked**: "Account locked due to multiple failed attempts. Try again in X minutes"
- **Account Inactive**: "This account has been deactivated. Contact administrator"
- **Session Expired**: "Your session has expired. Please log in again"
- **Duplicate Username**: "Username already exists"
- **Duplicate Email**: "Email address already registered"
- **Weak Password**: "Password does not meet security requirements"
- **Permission Denied**: "You do not have permission to perform this action"

---

## Notes

- This unit is the foundation of the system and should be built first
- All other units depend on this for authentication and authorization
- Security is critical - follow best practices for password hashing and session management
- Consider using established libraries for password hashing (bcrypt, Argon2)
- Session tokens should be cryptographically secure random values
- All authentication events should be logged for audit purposes
