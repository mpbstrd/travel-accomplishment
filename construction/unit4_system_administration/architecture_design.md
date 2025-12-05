# Unit 4: System Administration - Architecture Design

## 1. Unit Overview

**Business Capability**: Provide system administration, audit logging, configuration management, and backup/restore capabilities.

**Responsibilities**:
- Audit trail management for all system activities
- System configuration settings management
- Database backup and restore operations
- System statistics and reporting
- Administrative oversight and monitoring

**Scope**:
- Cross-cutting concern consumed by all units
- Provides audit logging for compliance
- Manages system-wide configuration
- Handles data backup and recovery
- Generates administrative reports and statistics

**Technology Stack**:
- Platform: C# .NET 8 Windows Forms
- Data Access: Entity Framework Core with SQLite
- Backup: SQLite database file copy with compression
- CSV Export: CsvHelper library
- Dependency Injection: Microsoft.Extensions.DependencyInjection

---

## 2. Feature Components

### 2.1 Audit Log Viewer Feature
**Responsibility**: Display and search system audit logs

**UI Components**:
- AuditLogViewerForm (Windows Form)
  - Audit log DataGridView
  - Search text box
  - Filter panel:
    - Date range picker
    - User filter dropdown
    - Action type filter dropdown
    - Entity type filter dropdown
    - Result filter (Success/Failure)
  - Apply Filters button
  - Clear Filters button
  - Export to CSV button
  - Pagination controls
  - Results count label

**Feature Operations**:
- Display audit logs in grid
- Search across multiple fields
- Apply multiple filters
- Export filtered results to CSV
- Paginate large result sets
- Sort by columns

### 2.2 System Configuration Feature
**Responsibility**: Manage system-wide configuration settings

**UI Components**:
- SystemConfigurationForm (Windows Form)
  - Configuration list grouped by category:
    - Security Settings
    - Session Settings
    - Report Settings
    - Notification Settings
    - Backup Settings
  - Configuration items with:
    - Key name
    - Current value
    - Data type
    - Description
    - Edit button
    - Reset to Default button
  - Save All Changes button
  - Cancel button
- ConfigurationEditorDialog (Modal Dialog)
  - Configuration key (read-only)
  - Value input (text/number/boolean based on type)
  - Description display
  - Default value display
  - Save button
  - Cancel button

**Feature Operations**:
- List all configuration settings
- Edit configuration values
- Validate configuration changes
- Reset to default values
- Save configuration changes
- Log configuration changes

### 2.3 Backup and Restore Feature
**Responsibility**: Manage database backups and restore operations

**UI Components**:
- BackupManagementForm (Windows Form)
  - Backup history DataGridView
  - Create Backup button
  - Restore button (per backup)
  - Delete button (per backup)
  - Test Restore button
  - Backup schedule settings
  - Progress bar for operations
  - Status label
- BackupConfirmationDialog (Modal Dialog)
  - Backup details display
  - Confirmation message
  - Proceed button
  - Cancel button
- RestoreConfirmationDialog (Modal Dialog)
  - Warning message (data will be overwritten)
  - Backup details display
  - Confirmation checkbox
  - Restore button
  - Cancel button

**Feature Operations**:
- Create manual backup
- Schedule automatic backups
- View backup history
- Restore from backup
- Test restore (to temporary location)
- Delete old backups
- Display backup size and date

### 2.4 Statistics Dashboard Feature
**Responsibility**: Display system usage statistics and analytics

**UI Components**:
- StatisticsDashboardForm (Windows Form)
  - Summary cards:
    - Total Reports
    - Reports by Status (pie chart)
    - Active Users
    - Pending Signatures
  - Date range selector
  - Charts:
    - Reports by Branch (bar chart)
    - Reports by Activity Type (pie chart)
    - Reports Over Time (line chart)
    - Average Signature Time (bar chart)
  - Refresh button
  - Export to Excel button
  - Print button

**Feature Operations**:
- Display summary statistics
- Generate charts and graphs
- Filter by date range
- Export statistics to Excel/CSV
- Refresh data
- Print dashboard


---

## 3. Business Services

### 3.1 AuditLogService
**Responsibility**: Record and retrieve audit log entries

**Public Operations**:
- LogActionAsync(userId, actionType, entityType, entityId, details, result) → bool
  - Records audit log entry
  - Captures timestamp, user, action, entity, IP address
  - Returns success status
  - Called by all units for important operations
  
- GetAuditLogsAsync(filters) → PagedResult<AuditLogDto>
  - Retrieves audit logs with filters
  - Filters: dateRange, userId, actionType, entityType, result
  - Returns paginated results
  - Admin only
  
- SearchAuditLogsAsync(searchTerm, filters) → PagedtLogDto>
  - Searches audit logs by term
  - Searches: username, actionType, entityType, details
  - Returns paginated results
  
- ExportAuditLogsAsync(filters, format) → byte[]
  - Exports audit logs to CSV
  - Applies filters before export
  - Returns file as byte array
  - Admin only
  
- GetAuditLogByIdAsync(logId) → AuditLogDto
  - Retrieves single audit log entry
  
- GetRecentActivityAsync(count) → List<AuditLogDto>
  - Returns most recent audit log entries
  - Used for dashboard display
  
- CleanupOldLogsAsync(retentionDays) → int
  - Removes logs older than retention period
  - Default: 365 days (1 year)
  - Returns count of deleted logs
  - Background job

**Dependencies**:
- IAuditLogRepository
- IUserManagementService (Unit 1) for user details

### 3.2 ConfigurationService
**Responsibility**: Manage system configuration settings

**Public Operations**:
- GetConfigurationAsync(configKey) → string
  - Returns configuration value by key
  - Returns default if not found
  - Used by all units
  
- GetConfigurationTypedAsync<T>(configKey) → T
  - Returns configuration value with type conversion
  - Supports: int, bool, decimal, string
  
- SetConfigurationAsync(configKey, configValue, userId) → bool
  - Updates configuration value
  - Validates value based on data type
  - Logs change to audit trail
  - Admin only
  
- GetAllConfigurationsAsync() → List<ConfigurationDto>
  - Returns all configuration settings
  - Grouped by category
  - Admin only
  
- ResetConfigurationAsync(configKey) → bool
  - Resets configuration to default value
  - Logs reset action
  - Admin only
  
- ValidateConfigurationValueAsync(configKey, value) → ValidationResult
  - Validates configuration value
  - Checks data type and constraints
  - Returns validation result
  
- GetConfigurationCategoriesAsync() → List<string>
  - Returns list of configuration categories
  - Used for UI grouping

**Dependencies**:
- ISystemConfigurationRepository
- IAuditLogService (internal)

### 3.3 BackupService
**Responsibility**: Manage database backup and restore operations

**Public Operations**:
- CreateBackupAsync(userId, backupType) → int (backupId)
  - Creates database backup
  - Types: Manual, Automatic
  - Copies SQLite database file
  - Compresses backup file
  - Records backup metadata
  - Returns backup ID
  - Admin only
  
- RestoreBackupAsync(backupId, userId) → bool
  - Restores database from backup
  - Validates backup file exists
  - Creates pre-restore backup
  - Replaces current database
  - Logs restore action
  - Requires confirmation
  - Admin only
  
- GetBackupHistoryAsync() → List<BackupHistoryDto>
  - Returns all backup records
  - Ordered by date (newest first)
  - Admin only
  
- DeleteBackupAsync(backupId, userId) → bool
  - Deletes backup file and record
  - Validates backup not in use
  - Logs deletion
  - Admin only
  
- TestRestoreAsync(backupId) → bool
  - Tests backup file integrity
  - Restores to temporary location
  - Validates database structure
  - Returns success status
  
- ScheduleAutomaticBackupAsync() → void
  - Background job for automatic backups
  - Runs daily at configured time
  - Creates automatic backup
  - Cleans up old backups
  
- CleanupOldBackupsAsync(retentionDays) → int
  - Removes backups older than retention period
  - Default: 30 days
  - Returns count of deleted backups
  - Background job
  
- GetBackupSizeAsync(backupId) → long
  - Returns backup file size in bytes

**Dependencies**:
- IBackupHistoryRepository
- IConfigurationService (internal)
- IAuditLogService (internal)
- System.IO for file operations
- System.IO.Compression for compression

### 3.4 StatisticsService
**Responsibility**: Generate system statistics and analytics

**Public Operations**:
- GetSystemStatisticsAsync(dateRange) → SystemStatisticsDto
  - Returns comprehensive system statistics
  - Includes: report counts, user activity, signature metrics
  - Filtered by date range
  - Admin only
  
- GetReportStatisticsAsync(dateRange) → ReportStatisticsDto
  - Returns report-specific statistics
  - Includes: total, by status, by branch, by activity type
  
- GetUserActivityStatisticsAsync(dateRange) → UserActivityStatisticsDto
  - Returns user activity statistics
  - Includes: active users, login counts, report creation
  
- GetSignatureStatisticsAsync(dateRange) → SignatureStatisticsDto
  - Returns signature workflow statistics
  - Includes: average time per step, completion rate
  
- GenerateStatisticsReportAsync(dateRange, format) → byte[]
  - Generates statistics report file
  - Formats: CSV, Excel
  - Returns file as byte array
  
- RefreshStatisticsAsync() → bool
  - Recalculates current statistics
  - Updates SystemStatistics table
  - Background job (runs daily)
  
- GetDashboardSummaryAsync() → DashboardSummaryDto
  - Returns summary data for dashboard
  - Includes key metrics and charts data

**Dependencies**:
- ISystemStatisticsRepository
- IReportManagementService (Unit 2)
- IUserManagementService (Unit 1)
- ISignatureService (Unit 3)

### 3.5 SystemHealthService
**Responsibility**: Monitor system health and performance

**Public Operations**:
- GetSystemHealthAsync() → SystemHealthDto
  - Returns system health status
  - Includes: database size, disk space, memory usage
  
- GetDatabaseSizeAsync() → long
  - Returns database file size in bytes
  
- GetDiskSpaceAsync() → DiskSpaceDto
  - Returns available disk space
  - Warns if low disk space
  
- CheckDatabaseIntegrityAsync() → bool
  - Runs SQLite integrity check
  - Returns true if database is healthy
  
- OptimizeDatabaseAsync() → bool
  - Runs SQLite VACUUM command
  - Optimizes database file size
  - Admin only

**Dependencies**:
- System.IO for file system operations
- SQLite connection for integrity check

---

## 4. Data Models

### 4.1 AuditLog Entity
**Maps to**: AuditLogs table

**Properties**:
- LogId (long, primary key)
- LogTimestamp (DateTime, required)
- UserId (int, foreign key, nullable)
- Username (string, nullable, max 50)
- ActionType (string, required, max 100)
- EntityType (string, nullable, max 50)
- EntityId (string, nullable, max 50)
- ActionDetails (string, nullable)
- IPAddress (string, nullable, max 50)
- Result (string, required, max 20) - "Success", "Failure", "Error"

**Relationships**:
- User (User from Unit 1) - navigation property (nullable)

**Validation Rules**:
- ActionType: required, max 100 characters
- Result: must be "Success", "Failure", or "Error"
- LogTimestamp: required, auto-generated

**Business Logic**:
- Immutable once created (no updates or deletes)
- Username stored separately for deleted users
- Retention: minimum 1 year

### 4.2 SystemConfiguration Entity
**Maps to**: SystemConfiguration table

**Properties**:
- ConfigId (int, primary key)
- ConfigKey (string, unique, required, max 100)
- ConfigValue (string, required, max 500)
- DataType (string, required, max 20) - "Int", "String", "Boolean", "Decimal"
- DefaultValue (string, required, max 500)
- Description (string, nullable, max 500)
- Category (string, nullable, max 50)
- LastModifiedBy (int, foreign key, nullable)
- LastModifiedDate (DateTime, nullable)

**Relationships**:
- LastModifiedByUser (User from Unit 1) - navigation property

**Validation Rules**:
- ConfigKey: required, unique, max 100 characters
- ConfigValue: required, max 500 characters
- DataType: must be valid type
- Value must match DataType

**Configuration Keys**:
- Security.PasswordMinLength (Int, default: 8)
- Security.PasswordRequireUppercase (Boolean, default: true)
- Security.PasswordRequireLowercase (Boolean, default: true)
- Security.PasswordRequireNumber (Boolean, default: true)
- Security.PasswordHistoryCount (Int, default: 3)
- Security.AccountLockoutThreshold (Int, default: 5)
- Security.AccountLockoutDuration (Int, default: 15)
- Session.TimeoutMinutes (Int, default: 30)
- Session.ExtendOnActivity (Boolean, default: true)
- Report.MaxImagesPerReport (Int, default: 10)
- Report.MaxImageSizeMB (Int, default: 15)
- Report.AutoSaveIntervalMinutes (Int, default: 2)
- Notification.ReminderThresholdHours (Int, default: 24)
- Notification.ReminderFrequencyHours (Int, default: 24)
- Backup.RetentionDays (Int, default: 30)
- Backup.AutomaticBackupEnabled (Boolean, default: true)
- Backup.AutomaticBackupTime (String, default: "02:00")
- AuditLog.RetentionDays (Int, default: 365)
- FileStorage.BasePath (String, default: "./AppData/Images")

### 4.3 BackupHistory Entity
**Maps to**: BackupHistory table

**Properties**:
- BackupId (int, primary key)
- BackupDate (DateTime, required)
- BackupType (string, required, max 20) - "Manual", "Automatic"
- BackupPath (string, required, max 500)
- BackupSize (long, required)
- BackupStatus (string, required, max 20) - "Success", "Failed", "InProgress"
- InitiatedBy (int, foreign key, nullable)
- CompletedDate (DateTime, nullable)
- ErrorMessage (string, nullable, max 500)

**Relationships**:
- InitiatedByUser (User from Unit 1) - navigation property

**Validation Rules**:
- BackupType: must be "Manual" or "Automatic"
- BackupStatus: must be valid status
- BackupPath: required, max 500 characters

**Business Logic**:
- Backup files stored in configured backup directory
- Filename format: TravelReports_YYYYMMDD_HHMMSS.db.gz
- Compressed using GZip
- Retention: 30 days (configurable)

### 4.4 SystemStatistics Entity
**Maps to**: SystemStatistics table

**Properties**:
- StatId (int, primary key)
- StatDate (DateTime, required)
- TotalReports (int, required, default 0)
- DraftReports (int, required, default 0)
- PendingReports (int, required, default 0)
- CompletedReports (int, required, default 0)
- TotalUsers (int, required, default 0)
- ActiveUsers (int, required, default 0)
- TotalSignatures (int, required, default 0)
- AvgSignatureTimeHours (decimal, nullable)

**Validation Rules**:
- All counts must be >= 0
- StatDate: required, unique per day

**Business Logic**:
- Calculated daily by background job
- Historical data for trend analysis
- Used for dashboard charts

---

## 5. Repository Pattern

### 5.1 IAuditLogRepository Interface
**Responsibility**: Data access for AuditLog entity

**Operations**:
- CreateAsync(auditLog) → AuditLog
- GetByIdAsync(logId) → AuditLog
- GetAllAsync(filters) → PagedResult<AuditLog>
- SearchAsync(searchTerm, filters) → PagedResult<AuditLog>
- GetRecentAsync(count) → List<AuditLog>
- DeleteOldLogsAsync(olderThanDate) → int

**Implementation**: AuditLogRepository (EF Core)
- Immutable records (no update/delete operations)
- Supports complex filtering
- Implements pagination
- Optimized for read operations

### 5.2 ISystemConfigurationRepository Interface
**Responsibility**: Data access for SystemConfiguration entity

**Operations**:
- GetByKeyAsync(configKey) → SystemConfiguration
- GetAllAsync() → List<SystemConfiguration>
- GetByCategoryAsync(category) → List<SystemConfiguration>
- UpdateAsync(configuration) → SystemConfiguration
- ResetToDefaultAsync(configKey) → bool

**Implementation**: SystemConfigurationRepository (EF Core)
- Cached for performance
- Thread-safe access
- Validates data type conversions

### 5.3 IBackupHistoryRepository Interface
**Responsibility**: Data access for BackupHistory entity

**Operations**:
- CreateAsync(backup) → BackupHistory
- GetByIdAsync(backupId) → BackupHistory
- GetAllAsync() → List<BackupHistory>
- UpdateAsync(backup) → BackupHistory
- DeleteAsync(backupId) → bool
- DeleteOldBackupsAsync(olderThanDate) → int

**Implementation**: BackupHistoryRepository (EF Core)
- Tracks backup operations
- Supports cleanup of old records

### 5.4 ISystemStatisticsRepository Interface
**Responsibility**: Data access for SystemStatistics entity

**Operations**:
- CreateAsync(statistics) → SystemStatistics
- GetByDateAsync(date) → SystemStatistics
- GetByDateRangeAsync(startDate, endDate) → List<SystemStatistics>
- GetLatestAsync() → SystemStatistics
- UpdateAsync(statistics) → SystemStatistics

**Implementation**: SystemStatisticsRepository (EF Core)
- One record per day
- Historical data for analytics

---

## 6. Service Interfaces (Exposed to Other Units)

### 6.1 IAuditLogService Interface
**Exposed Operations**:
- LogActionAsync(userId, actionType, entityType, entityId, details, result) → bool
- GetAuditLogsAsync(filters) → PagedResult<AuditLogDto>
- ExportAuditLogsAsync(filters, format) → byte[]

**Data Contracts**:

**AuditLogDto**:
- LogId (long)
- LogTimestamp (DateTime)
- Username (string)
- ActionType (string)
- EntityType (string)
- EntityId (string)
- ActionDetails (string)
- IPAddress (string)
- Result (string)

**AuditLogFilters**:
- StartDate (DateTime?)
- EndDate (DateTime?)
- UserId (int?)
- ActionType (string)
- EntityType (string)
- Result (string)
- PageNumber (int)
- PageSize (int)

### 6.2 IConfigurationService Interface
**Exposed Operations**:
- GetConfigurationAsync(configKey) → string
- GetConfigurationTypedAsync<T>(configKey) → T
- SetConfigurationAsync(configKey, configValue, userId) → bool
- GetAllConfigurationsAsync() → List<ConfigurationDto>
- ResetConfigurationAsync(configKey) → bool

**Data Contracts**:

**ConfigurationDto**:
- ConfigKey (string)
- ConfigValue (string)
- DataType (string)
- DefaultValue (string)
- Description (string)
- Category (string)

### 6.3 IBackupService Interface
**Exposed Operations**:
- CreateBackupAsync(userId, backupType) → int
- RestoreBackupAsync(backupId, userId) → bool
- GetBackupHistoryAsync() → List<BackupHistoryDto>
- DeleteBackupAsync(backupId, userId) → bool

**Data Contracts**:

**BackupHistoryDto**:
- BackupId (int)
- BackupDate (DateTime)
- BackupType (string)
- BackupSize (long)
- BackupStatus (string)
- InitiatedByName (string)

### 6.4 IStatisticsService Interface
**Exposed Operations**:
- GetSystemStatisticsAsync(dateRange) → SystemStatisticsDto
- GenerateStatisticsReportAsync(dateRange, format) → byte[]
- RefreshStatisticsAsync() → bool

**Data Contracts**:

**SystemStatisticsDto**:
- TotalReports (int)
- ReportsByStatus (Dictionary<string, int>)
- ReportsByBranch (Dictionary<string, int>)
- ReportsByActivityType (Dictionary<string, int>)
- ActiveUsers (int)
- TotalSignatures (int)
- AvgSignatureTimeHours (decimal)
- ReportsOverTime (List<DateCountDto>)

---

## 7. Dependencies

### 7.1 Internal Dependencies
- Entity Framework Core (data access)
- System.IO.Compression (backup compression)
- CsvHelper (CSV export)
- Microsoft.Extensions.DependencyInjection (DI container)

### 7.2 External Unit Dependencies
**Depends On**:
- Unit 1 (User Management & Authentication):
  - IUserManagementService for user details in audit logs
  - IAuthorizationService for admin permission checks
- Unit 2 (Report Lifecycle Management):
  - IReportManagementService for report statistics
- Unit 3 (Signature Workflow Engine):
  - ISignatureService for signature statistics

**Consumed By**:
- All Units: Audit logging, configuration retrieval

### 7.3 Database Dependencies
**Tables Owned**:
- AuditLogs
- SystemConfiguration
- BackupHistory
- SystemStatistics

**Foreign Key References**:
- AuditLogs.UserId → Users.UserId (Unit 1, nullable)
- SystemConfiguration.LastModifiedBy → Users.UserId (Unit 1)
- BackupHistory.InitiatedBy → Users.UserId (Unit 1)

---

## 8. Component Diagram

```
┌──────────────────────────────────────────────────────────────────┐
│              Unit 4: System Administration                        │
├──────────────────────────────────────────────────────────────────┤
│                                                                   │
│  ┌───────────────────────────────────────────────────────────┐  │
│  │              UI Layer (Windows Forms)                      │  │
│  │  ┌──────────┐  ┌──────────┐  ┌──────────┐  ┌──────────┐  │  │
│  │  │AuditLog  │  │System    │  │Backup    │  │Statistics│  │  │
│  │  │Viewer    │  │Config    │  │Mgmt      │  │Dashboard │  │  │
│  │  │Form      │  │Form      │  │Form      │  │Form      │  │  │
│  │  └────┬─────┘  └────┬─────┘  └────┬─────┘  └────┬─────┘  │  │
│  └───────┼─────────────┼─────────────┼─────────────┼─────────┘  │
│          │             │             │             │             │
│  ┌───────▼─────────────▼─────────────▼─────────────▼─────────┐  │
│  │              Business Services Layer                        │  │
│  │  ┌──────────────────┐  ┌──────────────────┐               │  │
│  │  │AuditLog          │  │Configuration     │               │  │
│  │  │Service           │  │Service           │               │  │
│  │  └────────┬─────────┘  └────────┬─────────┘               │  │
│  │           │                     │                          │  │
│  │  ┌────────▼─────────┐  ┌───────▼──────────┐              │  │
│  │  │Backup            │  │Statistics        │              │  │
│  │  │Service           │  │Service           │              │  │
│  │  └────────┬─────────┘  └───────┬──────────┘              │  │
│  │           │                     │                          │  │
│  │  ┌────────▼─────────────────────▼──────────┐              │  │
│  │  │SystemHealthService                       │              │  │
│  │  └──────────────────┬───────────────────────┘              │  │
│  └─────────────────────┼──────────────────────────────────────┘  │
│                        │                                          │
│  ┌─────────────────────▼──────────────────────────────────────┐  │
│  │              Repository Layer                               │  │
│  │  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐     │  │
│  │  │AuditLog      │  │SystemConfig  │  │BackupHistory │     │  │
│  │  │Repository    │  │Repository    │  │Repository    │     │  │
│  │  └──────┬───────┘  └──────┬───────┘  └──────┬───────┘     │  │
│  │         │                 │                 │              │  │
│  │  ┌──────▼─────────────────▼─────────────────▼───────────┐ │  │
│  │  │SystemStatisticsRepository                            │ │  │
│  │  └──────────────────────┬───────────────────────────────┘ │  │
│  └─────────────────────────┼─────────────────────────────────┘  │
│                            │                                     │
│  ┌─────────────────────────▼─────────────────────────────────┐  │
│  │              Data Access Layer (EF Core)                   │  │
│  │  ┌────────────────────────────────────────────────────┐   │  │
│  │  │            DbContext (SQLite)                       │   │  │
│  │  │  - AuditLogs                                        │   │  │
│  │  │  - SystemConfiguration                              │   │  │
│  │  │  - BackupHistory                                    │   │  │
│  │  │  - SystemStatistics                                 │   │  │
│  │  └────────────────────────────────────────────────────┘   │  │
│  └────────────────────────────────────────────────────────────┘  │
│                                                                   │
│  ┌──────────────────────────────────────────────────────────┐   │
│  │         Service Interfaces (Exposed to Other Units)       │   │
│  │  - IAuditLogService                                       │   │
│  │  - IConfigurationService                                  │   │
│  │  - IBackupService                                         │   │
│  │  - IStatisticsService                                     │   │
│  └──────────────────────────────────────────────────────────┘   │
│                                                                   │
│  ┌──────────────────────────────────────────────────────────┐   │
│  │         File System Storage                               │   │
│  │  - Backups stored in: /AppData/Backups/                  │   │
│  │  - Compressed with GZip                                   │   │
│  └──────────────────────────────────────────────────────────┘   │
└──────────────────────────────────────────────────────────────────┘
```

---

## 9. Database Schema


### 9.1 AuditLogs Table
```
Table: AuditLogs
Primary Key: LogId (BIGINT, IDENTITY)
Indexes:
  - IX_AuditLogs_LogTimestamp
  - IX_AuditLogs_UserId
  - IX_AuditLogs_ActionType
  - IX_AuditLogs_EntityType
  - IX_AuditLogs_Result

Columns:
  LogId               BIGINT          PRIMARY KEY IDENTITY
  LogTimestamp        DATETIME        NOT NULL DEFAULT CURRENT_TIMESTAMP
  UserId              INT             NULL FOREIGN KEY REFERENCES Users(UserId)
  Username            NVARCHAR(50)    NULL
  ActionType          NVARCHAR(100)   NOT NULL
  EntityType          NVARCHAR(50)    NULL
  EntityId            NVARCHAR(50)    NULL
  ActionDetails       NVARCHAR(MAX)   NULL
  IPAddress           NVARCHAR(50)    NULL
  Result              NVARCHAR(20)    NOT NULL CHECK (Result IN ('Success', 'Failure', 'Error'))
```

### 9.2 SystemConfiguration Table
```
Table: SystemConfiguration
Primary Key: ConfigId (INT, IDENTITY)
Indexes:
  - IX_SystemConfiguration_ConfigKey (UNIQUE)
  - IX_SystemConfiguration_Category

Columns:
  ConfigId            INT             PRIMARY KEY IDENTITY
  ConfigKey           NVARCHAR(100)   NOT NULL UNIQUE
  ConfigValue         NVARCHAR(500)   NOT NULL
  DataType            NVARCHAR(20)    NOT NULL CHECK (DataType IN ('Int', 'String', 'Boolean', 'Decimal'))
  DefaultValue        NVARCHAR(500)   NOT NULL
  Description         NVARCHAR(500)   NULL
  Category            NVARCHAR(50)    NULL
  LastModifiedBy      INT             NULL FOREIGN KEY REFERENCES Users(UserId)
  LastModifiedDate    DATETIME        NULL
```

### 9.3 BackupHistory Table
```
Table: BackupHistory
Primary Key: BackupId (INT, IDENTITY)
Indexes:
  - IX_BackupHistory_BackupDate
  - IX_BackupHistory_BackupStatus

Columns:
  BackupId            INT             PRIMARY KEY IDENTITY
  BackupDate          DATETIME        NOT NULL DEFAULT CURRENT_TIMESTAMP
  BackupType          NVARCHAR(20)    NOT NULL CHECK (BackupType IN ('Manual', 'Automatic'))
  BackupPath          NVARCHAR(500)   NOT NULL
  BackupSize          BIGINT          NOT NULL
  BackupStatus        NVARCHAR(20)    NOT NULL CHECK (BackupStatus IN ('Success', 'Failed', 'InProgress'))
  InitiatedBy         INT             NULL FOREIGN KEY REFERENCES Users(UserId)
  CompletedDate       DATETIME        NULL
  ErrorMessage        NVARCHAR(500)   NULL
```

### 9.4 SystemStatistics Table
```
Table: SystemStatistics
Primary Key: StatId (INT, IDENTITY)
Indexes:
  - IX_SystemStatistics_StatDate (UNIQUE)

Columns:
  StatId              INT             PRIMARY KEY IDENTITY
  StatDate            DATE            NOT NULL UNIQUE
  TotalReports        INT             NOT NULL DEFAULT 0
  DraftReports        INT             NOT NULL DEFAULT 0
  PendingReports      INT             NOT NULL DEFAULT 0
  CompletedReports    INT             NOT NULL DEFAULT 0
  TotalUsers          INT             NOT NULL DEFAULT 0
  ActiveUsers         INT             NOT NULL DEFAULT 0
  TotalSignatures     INT             NOT NULL DEFAULT 0
  AvgSignatureTimeHours DECIMAL(10,2) NULL
```

### 9.5 Relationships
- AuditLogs.UserId → Users.UserId (many-to-one, nullable)
- SystemConfiguration.LastModifiedBy → Users.UserId (many-to-one)
- BackupHistory.InitiatedBy → Users.UserId (many-to-one, nullable)

---

## 10. Implementation Notes

### 10.1 Entity Framework Core Configuration
- Use Code-First approach with migrations
- Configure indexes for query performance
- AuditLogs table uses BIGINT for primary key (large volume)
- Implement read-only behavior for AuditLogs (no updates/deletes)
- Use async operations throughout
- Configure cascade delete rules (restrict for user references)

### 10.2 Audit Logging Strategy
- Log all important system operations
- Capture: timestamp, user, action, entity, result, IP address
- Action types:
  - User.Login, User.Logout, User.LoginFailed
  - User.Created, User.Updated, User.Deleted
  - Report.Created, Report.Updated, Report.Deleted, Report.Submitted
  - Signature.Submitted, Signature.Failed
  - Configuration.Updated, Configuration.Reset
  - Backup.Created, Backup.Restored, Backup.Deleted
- Store username separately for deleted users
- Immutable records (no updates or deletes)
- Retention: minimum 1 year (configurable)
- Background job for cleanup of old logs

### 10.3 Configuration Management
- Store all system-wide settings in database
- Configuration cached in memory for performance
- Cache invalidated on configuration change
- Thread-safe access to configuration
- Validate configuration values before saving
- Log all configuration changes to audit trail
- Support type conversion (string to int, bool, decimal)
- Provide default values for all configurations
- Group configurations by category for UI display

### 10.4 Backup Strategy
- Backup SQLite database file
- Compress backup with GZip
- Store backups in configured directory
- Filename format: TravelReports_YYYYMMDD_HHMMSS.db.gz
- Automatic backups scheduled daily (configurable time)
- Manual backups on-demand (admin only)
- Retention: 30 days (configurable)
- Background job for cleanup of old backups
- Pre-restore backup before restore operation
- Test restore capability to validate backup integrity

### 10.5 Backup and Restore Process
**Backup Process**:
1. Close all database connections
2. Copy SQLite database file to temp location
3. Compress file with GZip
4. Move compressed file to backup directory
5. Record backup metadata in BackupHistory table
6. Reopen database connections

**Restore Process**:
1. Validate backup file exists and is readable
2. Create pre-restore backup (safety)
3. Close all database connections
4. Decompress backup file
5. Replace current database file
6. Reopen database connections
7. Verify database integrity
8. Log restore action to audit trail

### 10.6 Statistics Calculation
- Background job runs daily (configurable time)
- Calculates statistics for previous day
- Stores in SystemStatistics table
- Statistics include:
  - Report counts (total, by status)
  - User activity (total, active)
  - Signature metrics (total, average time)
- Historical data for trend analysis
- Used for dashboard charts and reports
- On-demand refresh available (admin only)

### 10.7 System Health Monitoring
- Monitor database file size
- Monitor available disk space
- Warn if disk space low (< 1GB)
- Check database integrity (SQLite PRAGMA integrity_check)
- Optimize database periodically (VACUUM command)
- Display health status in admin dashboard

### 10.8 Error Handling
- Use try-catch blocks in service methods
- Return meaningful error messages to UI
- Log all errors to audit trail
- Handle file system errors (disk full, permissions)
- Handle database errors (constraint violations, deadlocks)
- Implement retry logic for transient failures
- Display user-friendly error messages

### 10.9 Dependency Injection Setup
- Register services with appropriate lifetime:
  - Scoped: Services, Repositories, DbContext
  - Singleton: ConfigurationService (cached), Background jobs
  - Transient: Validators, Export services
- Use constructor injection throughout
- Register interfaces with implementations
- Configure in Program.cs or Startup class

### 10.10 Background Jobs
- Use System.Threading.Timer for scheduled tasks
- Background jobs:
  - Automatic backup (daily at configured time)
  - Statistics calculation (daily)
  - Audit log cleanup (weekly)
  - Backup cleanup (daily)
- Jobs run at low-priority intervals
- Error handling and logging for background jobs
- Configurable job schedules

### 10.11 Testing Considerations
- Unit test services with mocked repositories
- Integration test repositories with in-memory SQLite
- Test backup and restore operations
- Test configuration management
- Test audit logging
- Test statistics calculation
- Mock external unit dependencies (Unit 1, 2, 3)

### 10.12 Performance Considerations
- Use async/await throughout
- Implement connection pooling (EF Core default)
- Add indexes on frequently queried columns
- Cache configuration values in memory
- Paginate audit log results
- Optimize statistics queries
- Background jobs run at off-peak times
- Compress backup files to save disk space

### 10.13 Security Best Practices
- Validate all inputs
- Admin-only access to all features
- Use parameterized queries (EF Core default)
- Secure backup files (file permissions)
- Encrypt sensitive configuration values (optional)
- Log all administrative actions
- Validate user permissions before operations
- Regular security audits

---

## 11. Integration Points

### 11.1 With Unit 1 (User Management & Authentication)
- Receive audit log calls for:
  - Login attempts (success/failure)
  - User account operations
  - Password changes
  - Session management
- Call IUserManagementService.GetUserByIdAsync() for user details in audit logs
- Call IAuthorizationService.IsAdminAsync() for permission checks

### 11.2 With Unit 2 (Report Lifecycle Management)
- Receive audit log calls for:
  - Report creation
  - Report modification
  - Report deletion
  - Image upload/deletion
  - Export operations
- Provide configuration values for:
  - Max images per report
  - Max image size
  - Auto-save interval
  - File storage path
- Call IReportManagementService for report statistics

### 11.3 With Unit 3 (Signature Workflow Engine)
- Receive audit log calls for:
  - Signature submissions
  - Workflow state changes
  - Notification creation
- Provide configuration values for:
  - Reminder threshold
  - Reminder frequency
- Call ISignatureService for signature statistics

### 11.4 Application Startup
- Initialize DbContext with SQLite connection
- Run EF Core migrations for Admin tables
- Load configuration into cache
- Seed default configuration values if not exist
- Register all services in DI container
- Start background jobs (backup, statistics, cleanup)
- Verify backup directory exists

---

## 12. Build Order and Dependencies

### 12.1 Build Order
1. Data Models (Entities)
2. Repository Interfaces
3. Repository Implementations
4. Service Interfaces
5. Configuration Service (needed by other services)
6. Audit Log Service (needed by other services)
7. Backup Service
8. Statistics Service
9. System Health Service
10. Background Jobs
11. UI Components
12. Integration Testing

### 12.2 Critical Path
- Build early (after Unit 1)
- Provides cross-cutting concerns for all units
- Audit logging needed by all units
- Configuration service needed by all units
- Can be developed in parallel with Unit 2

---

## 13. Common Audit Log Actions

### 13.1 User Management Actions
- User.Login
- User.Logout
- User.LoginFailed
- User.Created
- User.Updated
- User.Deleted
- User.PasswordChanged
- User.PasswordReset
- User.AccountLocked
- User.AccountUnlocked

### 13.2 Report Management Actions
- Report.Created
- Report.Updated
- Report.Deleted
- Report.Restored
- Report.Submitted
- Report.StatusChanged
- Report.Exported
- Image.Uploaded
- Image.Deleted

### 13.3 Signature Actions
- Signature.Submitted
- Signature.Failed
- Workflow.Initialized
- Workflow.Advanced
- Workflow.Completed

### 13.4 System Administration Actions
- Configuration.Updated
- Configuration.Reset
- Backup.Created
- Backup.Restored
- Backup.Deleted
- Statistics.Refreshed
- Database.Optimized

---

## 14. Future Enhancements

### 14.1 Potential Improvements
- Real-time system monitoring dashboard
- Email alerts for system issues
- Advanced analytics and reporting
- Custom report builder
- Data export to external systems
- Integration with external backup services (cloud storage)
- Automated database optimization
- Performance metrics tracking
- User activity heatmaps
- Predictive analytics for system capacity
- Compliance reporting (GDPR, audit requirements)
- Multi-database support (MSSQL migration)
- Distributed logging (centralized log server)
- Advanced search with full-text indexing
- Audit log replay for troubleshooting
- Configuration versioning and rollback
- Scheduled reports via email
- Custom dashboard widgets

### 14.2 Scalability Considerations
- Move to centralized logging service (Elasticsearch, Splunk)
- Implement distributed caching (Redis)
- Use message queue for audit logging (RabbitMQ)
- Separate read/write databases (CQRS)
- Implement data archival strategy
- Consider NoSQL for audit logs (high volume)
- Cloud backup integration (Azure Blob, AWS S3)
- Horizontal scaling for statistics calculation

---

**Document Version**: 1.0  
**Date**: December 5, 2025  
**Status**: Ready for Implementation  
**Technology**: C# .NET 8, Windows Forms, Entity Framework Core, SQLite
