# Integration Architecture - Travel Accomplishment Report System

## Document Overview

**Purpose**: Define the overall application architecture, integration patterns, and cross-cutting concerns for the Travel Accomplishment Report System.

**Architecture Style**: Modular Monolith with Service-Oriented patterns
**Platform**: C# .NET 8 Windows Forms Desktop Application
**Database**: SQLite (single embedded database file)
**Communication**: In-process synchronous method calls through interfaces

---

## 1. Application Architecture Overview

### 1.1 Architectural Style
- **Modular Monolith**: Single application with well-defined module boundaries
- **Service-Oriented**: Clear service interfaces between modules
- **Layered Architecture**: UI, Business Services, Data Access layers
- **Repository Pattern**: Data access abstraction
- **Dependency Injection**: Loose coupling between components

### 1.2 Design Principles
- **Separation of Concerns**: Each unit has clear responsibilities
- **Interface-Based Communication**: Units communicate through interfaces
- **Single Responsibility**: Each service has one clear purpose
- **Dependency Inversion**: Depend on abstractions, not implementations
- **Open/Closed**: Open for extension, closed for modification

### 1.3 Technology Stack
- **Framework**: .NET 8
- **UI**: Windows Forms
- **Data Access**: Entity Framework Core 8
- **Database**: SQLite
- **DI Container**: Microsoft.Extensions.DependencyInjection
- **Password Hashing**: BCrypt.Net-Next
- **PDF Generation**: iTextSharp or QuestPDF
- **CSV Export**: CsvHelper
- **Compression**: System.IO.Compression

---

## 2. Application Structure

### 2.1 Solution Structure
```
TravelAccomplishmentReport.sln
│
├── TravelAccomplishmentReport.UI (Windows Forms)
│   ├── Forms/
│   │   ├── LoginForm.cs
│   │   ├── MainForm.cs
│   │   ├── ReportEditorForm.cs
│   │   ├── ReportViewForm.cs
│   │   ├── UserManagementForm.cs
│   │   ├── AuditLogViewerForm.cs
│   │   └── ...
│   ├── Controls/
│   │   ├── SignatureWorkflowPanel.cs
│   │   ├── NotificationPanel.cs
│   │   ├── ImageUploadControl.cs
│   │   └── ...
│   ├── Program.cs
│   └── Startup.cs
│
├── TravelAccomplishmentReport.Core
│   ├── Unit1.UserManagement/
│   │   ├── Services/
│   │   ├── Interfaces/
│   │   └── DTOs/
│   ├── Unit2.ReportLifecycle/
│   │   ├── Services/
│   │   ├── Interfaces/
│   │   └── DTOs/
│   ├── Unit3.SignatureWorkflow/
│   │   ├── Services/
│   │   ├── Interfaces/
│   │   └── DTOs/
│   └── Unit4.SystemAdministration/
│       ├── Services/
│       ├── Interfaces/
│       └── DTOs/
│
├── TravelAccomplishmentReport.Data
│   ├── Entities/
│   │   ├── User.cs
│   │   ├── Report.cs
│   │   ├── ReportSignature.cs
│   │   ├── AuditLog.cs
│   │   └── ...
│   ├── Repositories/
│   │   ├── UserRepository.cs
│   │   ├── ReportRepository.cs
│   │   └── ...
│   ├── Context/
│   │   └── ApplicationDbContext.cs
│   └── Migrations/
│
└── TravelAccomplishmentReport.Tests
    ├── Unit1.Tests/
    ├── Unit2.Tests/
    ├── Unit3.Tests/
    ├── Unit4.Tests/
    └── Integration.Tests/
```

### 2.2 Project Dependencies
```
UI → Core → Data
UI → Data (for DbContext registration)
Core → Data (for entities and repositories)
```

---

## 3. Application Startup and Initialization

### 3.1 Program.cs (Entry Point)
**Responsibilities**:
- Configure dependency injection container
- Register all services and repositories
- Configure database connection
- Run database migrations
- Create default admin account if needed
- Start background jobs
- Launch main form

**Initialization Sequence**:
1. Create ServiceCollection
2. Register services (ConfigureServices)
3. Build ServiceProvider
4. Run database migrations
5. Seed initial data
6. Start background jobs
7. Show login form
8. On successful login, show main form

### 3.2 Dependency Injection Configuration

**Service Lifetimes**:
- **Singleton**: 
  - ConfigurationService (cached)
  - Background job services
  - File storage service
- **Scoped**:
  - DbContext
  - All business services
  - All repositories
- **Transient**:
  - Validators
  - Export services
  - Helper classes

**Registration Pattern**:
```
services.AddDbContext<ApplicationDbContext>(options =>
    options.UseSqlite(connectionString));

// Unit 1 Services
services.AddScoped<IAuthenticationService, AuthenticationService>();
services.AddScoped<IUserManagementService, UserManagementService>();
services.AddScoped<IAuthorizationService, AuthorizationService>();
services.AddScoped<IPasswordService, PasswordService>();
services.AddScoped<IAccountLockoutService, AccountLockoutService>();

// Unit 1 Repositories
services.AddScoped<IUserRepository, UserRepository>();
services.AddScoped<IUserSessionRepository, UserSessionRepository>();
services.AddScoped<ILoginAttemptRepository, LoginAttemptRepository>();
services.AddScoped<IPasswordHistoryRepository, PasswordHistoryRepository>();

// Unit 2 Services
services.AddScoped<IReportManagementService, ReportManagementService>();
services.AddScoped<IReportStatusService, ReportStatusService>();
services.AddScoped<IImageManagementService, ImageManagementService>();
services.AddScoped<IReportExportService, ReportExportService>();
services.AddSingleton<IFileStorageService, FileStorageService>();
services.AddScoped<IReportValidationService, ReportValidationService>();

// Unit 2 Repositories
services.AddScoped<IReportRepository, ReportRepository>();
services.AddScoped<INetworkEquipmentRepository, NetworkEquipmentRepository>();
services.AddScoped<IActivityChecklistRepository, ActivityChecklistRepository>();
services.AddScoped<IReportImageRepository, ReportImageRepository>();
services.AddScoped<IReportEditHistoryRepository, ReportEditHistoryRepository>();

// Unit 3 Services
services.AddScoped<ISignatureService, SignatureService>();
services.AddScoped<IWorkflowService, WorkflowService>();
services.AddScoped<INotificationService, NotificationService>();
services.AddSingleton<ISignatureReminderService, SignatureReminderService>();
services.AddScoped<ISignatureValidationService, SignatureValidationService>();

// Unit 3 Repositories
services.AddScoped<IReportSignatureRepository, ReportSignatureRepository>();
services.AddScoped<IWorkflowStateRepository, WorkflowStateRepository>();
services.AddScoped<INotificationRepository, NotificationRepository>();
services.AddScoped<ISignatureAttemptRepository, SignatureAttemptRepository>();

// Unit 4 Services
services.AddScoped<IAuditLogService, AuditLogService>();
services.AddSingleton<IConfigurationService, ConfigurationService>();
services.AddScoped<IBackupService, BackupService>();
services.AddScoped<IStatisticsService, StatisticsService>();
services.AddScoped<ISystemHealthService, SystemHealthService>();

// Unit 4 Repositories
services.AddScoped<IAuditLogRepository, AuditLogRepository>();
services.AddScoped<ISystemConfigurationRepository, SystemConfigurationRepository>();
services.AddScoped<IBackupHistoryRepository, BackupHistoryRepository>();
services.AddScoped<ISystemStatisticsRepository, SystemStatisticsRepository>();
```

### 3.3 Database Connection Management

**Connection String**:
```
Data Source=./AppData/TravelReports.db;Cache=Shared;
```

**Connection Pooling**:
- Enabled by default in EF Core
- Shared cache mode for SQLite
- Connection timeout: 30 seconds

**Migration Strategy**:
- Code-First migrations
- Automatic migration on startup
- Migration history tracked in __EFMigrationsHistory table

### 3.4 Initial Data Seeding

**Default Admin Account**:
- Username: admin
- Password: Admin@123 (must be changed on first login)
- Role: Admin
- Created if no admin exists

**Default Configuration Values**:
- Seed all configuration keys with default values
- Only seed if configuration table is empty

---

## 4. Database Architecture

### 4.1 Database Schema Overview

**Unit 1 Tables**:
- Users
- UserSessions
- LoginAttempts
- PasswordHistory

**Unit 2 Tables**:
- Reports
- NetworkEquipment
- ActivityChecklist
- ReportImages
- ReportEditHistory

**Unit 3 Tables**:
- ReportSignatures
- WorkflowState
- Notifications
- SignatureAttempts

**Unit 4 Tables**:
- AuditLogs
- SystemConfiguration
- BackupHistory
- SystemStatistics

### 4.2 Cross-Unit Relationships

**Foreign Key Relationships**:
- Reports.CreatedBy → Users.UserId
- Reports.LastModifiedBy → Users.UserId
- Reports.DeletedBy → Users.UserId
- ReportImages.UploadedBy → Users.UserId
- ReportEditHistory.EditedBy → Users.UserId
- ReportSignatures.SignatoryUserId → Users.UserId
- Notifications.UserId → Users.UserId
- SignatureAttempts.UserId → Users.UserId
- AuditLogs.UserId → Users.UserId
- SystemConfiguration.LastModifiedBy → Users.UserId
- BackupHistory.InitiatedBy → Users.UserId

**Cascade Rules**:
- User deletions: RESTRICT (prevent deletion if referenced)
- Report deletions: CASCADE for child entities (NetworkEquipment, ActivityChecklist, etc.)
- Soft delete pattern for Users and Reports

### 4.3 Database Indexes

**Performance Indexes**:
- Users: Username, Email, IsDeleted
- UserSessions: SessionToken, UserId, ExpiresAt
- Reports: ReportNumber, CreatedBy, DateOfTravel, ReportStatus, IsDeleted, BranchName
- ReportSignatures: ReportId, SignatoryUserId, SignatureType
- WorkflowState: ReportId, CurrentStep, WorkflowStatus
- Notifications: UserId, ReportId, IsRead, CreatedDate
- AuditLogs: LogTimestamp, UserId, ActionType, EntityType, Result
- SystemConfiguration: ConfigKey, Category

---

## 5. Integration Patterns

### 5.1 Service Communication Pattern

**Interface-Based Communication**:
- All units expose interfaces for their services
- Units depend on interfaces, not implementations
- Dependency injection resolves implementations at runtime

**Example Flow**:
```
UI Layer
  ↓ (calls)
ReportManagementService (Unit 2)
  ↓ (calls)
IAuthenticationService.ValidateSessionAsync() (Unit 1)
  ↓ (calls)
IWorkflowService.InitializeWorkflowAsync() (Unit 3)
  ↓ (calls)
IAuditLogService.LogActionAsync() (Unit 4)
```

### 5.2 Transaction Management

**Unit of Work Pattern**:
- DbContext acts as Unit of Work
- Scoped lifetime ensures single DbContext per request
- Transactions managed at service layer

**Transaction Scope**:
- Single unit operations: Implicit transaction (EF Core)
- Cross-unit operations: Explicit transaction using TransactionScope
- Rollback on exception

**Example**:
```
using var transaction = await _dbContext.Database.BeginTransactionAsync();
try
{
    // Unit 2: Create report
    var report = await _reportRepository.CreateAsync(reportData);
    
    // Unit 3: Initialize workflow
    await _workflowService.InitializeWorkflowAsync(report.ReportId);
    
    // Unit 4: Log action
    await _auditLogService.LogActionAsync(...);
    
    await transaction.CommitAsync();
}
catch
{
    await transaction.RollbackAsync();
    throw;
}
```

### 5.3 Error Handling Strategy

**Layered Error Handling**:
1. **UI Layer**: Display user-friendly messages
2. **Service Layer**: Catch and log exceptions, return error results
3. **Repository Layer**: Let exceptions bubble up

**Error Types**:
- **Validation Errors**: Return ValidationResult with errors
- **Business Rule Violations**: Throw custom exceptions
- **Database Errors**: Log and return generic error message
- **File System Errors**: Log and return specific error message

**Logging**:
- All exceptions logged to audit trail
- Include stack trace for debugging
- Sensitive data excluded from logs

### 5.4 Validation Strategy

**Multi-Layer Validation**:
1. **UI Layer**: Real-time validation, immediate feedback
2. **Service Layer**: Business rule validation
3. **Entity Layer**: Data annotation attributes
4. **Database Layer**: Constraints and check rules

**Validation Result Pattern**:
```
public class ValidationResult
{
    public bool IsValid { get; set; }
    public List<string> Errors { get; set; }
}
```

---

## 6. Cross-Cutting Concerns

### 6.1 Logging and Audit Trail

**Audit Logging**:
- All important operations logged via IAuditLogService
- Captured: timestamp, user, action, entity, result, IP address
- Immutable records
- Retention: 1 year minimum

**Actions to Log**:
- Authentication events
- User management operations
- Report CRUD operations
- Signature submissions
- Configuration changes
- Backup/restore operations

### 6.2 Configuration Management

**Centralized Configuration**:
- All settings stored in SystemConfiguration table
- Cached in memory for performance
- Accessed via IConfigurationService
- Type-safe retrieval with GetConfigurationTypedAsync<T>()

**Configuration Categories**:
- Security Settings
- Session Settings
- Report Settings
- Notification Settings
- Backup Settings
- Audit Log Settings

### 6.3 Session Management

**Session Flow**:
1. User logs in via IAuthenticationService
2. Session created in database
3. Session token returned to UI
4. UI stores session token
5. All subsequent requests include session token
6. Services validate session before operations
7. Session timeout after 30 minutes inactivity
8. Session invalidated on logout

**Session Validation**:
- Performed at UI layer before service calls
- Validates token exists and is active
- Checks expiration time
- Updates last activity timestamp

### 6.4 Security

**Authentication**:
- Password hashing with BCrypt (work factor 12)
- Session tokens: cryptographically secure random values
- Account lockout after 5 failed attempts (15 minutes)
- Password history: last 3 passwords

**Authorization**:
- Role-based access control (Admin/User)
- Permission checks before operations
- UI elements hidden/disabled based on permissions

**Data Protection**:
- Passwords never stored in plain text
- Session tokens encrypted
- Sensitive data excluded from logs
- File system permissions for image storage

### 6.5 File Storage

**Image Storage**:
- File system storage (not database)
- Base path: ./AppData/Images/
- Organization: YYYY/MM/DD/
- Filename: {ReportId}_{Guid}.{extension}
- Relative paths stored in database

**Backup Storage**:
- Base path: ./AppData/Backups/
- Filename: TravelReports_YYYYMMDD_HHMMSS.db.gz
- Compressed with GZip
- Retention: 30 days

---

## 7. Background Jobs

### 7.1 Job Scheduler

**Implementation**:
- System.Threading.Timer for scheduled tasks
- Singleton services for background jobs
- Error handling and logging

**Background Jobs**:
1. **Automatic Backup Job**
   - Frequency: Daily at configured time (default 2:00 AM)
   - Action: Create database backup
   
2. **Statistics Calculation Job**
   - Frequency: Daily at configured time (default 1:00 AM)
   - Action: Calculate and store daily statistics
   
3. **Signature Reminder Job**
   - Frequency: Every 6 hours
   - Action: Create reminders for pending signatures > 24 hours
   
4. **Audit Log Cleanup Job**
   - Frequency: Weekly (Sunday at 3:00 AM)
   - Action: Delete logs older than retention period
   
5. **Backup Cleanup Job**
   - Frequency: Daily at configured time (default 3:00 AM)
   - Action: Delete backups older than retention period
   
6. **Session Cleanup Job**
   - Frequency: Hourly
   - Action: Delete expired sessions

### 7.2 Job Management

**Job Lifecycle**:
- Started on application startup
- Run in background thread
- Error handling with retry logic
- Logging to audit trail
- Stopped on application shutdown

---

## 8. Performance Considerations

### 8.1 Database Performance

**Optimization Strategies**:
- Indexes on frequently queried columns
- Async operations throughout
- Connection pooling (EF Core default)
- Pagination for large result sets
- Eager loading with Include() for related entities
- Lazy loading disabled (explicit loading only)

**Query Optimization**:
- Use projections (Select) to limit data retrieved
- Avoid N+1 queries with Include()
- Use compiled queries for frequently executed queries
- Monitor query performance with logging

### 8.2 Caching Strategy

**Configuration Caching**:
- Configuration values cached in memory
- Cache invalidated on configuration change
- Thread-safe access

**Potential Caching**:
- User roles/permissions (optional)
- Dropdown values (branches, activity types)
- Report statistics (short-term cache)

### 8.3 UI Responsiveness

**Async Operations**:
- All database operations async
- UI remains responsive during operations
- Progress indicators for long-running operations
- Background threads for heavy processing

**Pagination**:
- Report lists: 20 per page
- Audit logs: 50 per page
- Notifications: 20 per page

---

## 9. Testing Strategy

### 9.1 Unit Testing

**Test Coverage**:
- Business services (mocked repositories)
- Validation logic
- Business rules
- Data transformations

**Mocking**:
- Mock repositories with test data
- Mock external unit dependencies
- Use in-memory collections for test data

### 9.2 Integration Testing

**Test Coverage**:
- Repository operations (in-memory SQLite)
- Service integration (real dependencies)
- Database migrations
- Transaction management

**Test Database**:
- In-memory SQLite for fast tests
- Isolated test data per test
- Cleanup after each test

### 9.3 End-to-End Testing

**Test Scenarios**:
- Complete user workflows
- Report creation to completion
- Signature workflow
- Backup and restore
- Error scenarios

---

## 10. Deployment

### 10.1 Deployment Package

**Application Files**:
- Executable (TravelAccomplishmentReport.exe)
- DLL dependencies
- Configuration files
- Database file (empty or with seed data)

**Directory Structure**:
```
TravelAccomplishmentReport/
├── TravelAccomplishmentReport.exe
├── *.dll (dependencies)
├── appsettings.json
└── AppData/
    ├── TravelReports.db (created on first run)
    ├── Images/ (created on first run)
    └── Backups/ (created on first run)
```

### 10.2 Installation

**Installation Steps**:
1. Copy application files to target directory
2. Run application (creates database and directories)
3. Login with default admin account
4. Change default admin password
5. Configure system settings

**Requirements**:
- Windows 10 or later
- .NET 8 Runtime
- 500MB disk space minimum
- Network access (intranet only)

### 10.3 Updates

**Update Process**:
1. Backup database
2. Stop application
3. Replace application files
4. Run application (migrations run automatically)
5. Verify functionality

---

## 11. Future Migration Path

### 11.1 Database Migration (SQLite to MSSQL)

**Migration Strategy**:
- Design with MSSQL compatibility in mind
- Use EF Core abstractions (avoid SQLite-specific features)
- Test migrations with both databases
- Provide migration tool for data transfer

**Considerations**:
- Connection string changes
- Transaction handling differences
- Performance tuning for MSSQL
- Backup strategy changes

### 11.2 Architecture Evolution

**Potential Enhancements**:
- Web-based UI (ASP.NET Core)
- Mobile app (Xamarin/MAUI)
- Microservices architecture
- Cloud deployment
- API layer for external integrations

---

**Document Version**: 1.0  
**Date**: December 5, 2025  
**Status**: Ready for Implementation  
**Technology**: C# .NET 8, Windows Forms, Entity Framework Core, SQLite
