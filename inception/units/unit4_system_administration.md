# Unit 4: System Administration

## Unit Overview

**Business Capability**: Provide system administration, audit logging, configuration management, and backup/restore capabilities.

**Responsibility**: This unit handles all administrative functions including audit trail management, system configuration, backup operations, and administrative oversight of the entire system.

**Scope**:
- Audit log viewing and management
- System configuration settings
- Backup and restore operations
- Administrative reporting and statistics
- System health monitoring

---

## User Stories

### US-7.1: View Audit Logs
**As an** Admin  
**I want to** view system audit logs  
**So that** I can track all system activities and changes

**Acceptance Criteria:**
- Audit log interface accessible to Admin only
- Logged events include:
  - User login/logout
  - Account creation/modification/deletion
  - Report creation/editing/deletion
  - Signature submissions
  - Failed login attempts
  - System configuration changes
- Each log entry shows:
  - Timestamp
  - User/Admin username
  - Action performed
  - Affected entity (Report ID, User ID, etc.)
  - IP address
  - Result (Success/Failure)
- Search and filter capabilities:
  - Date range, User, Action type, Entity ID
- Export audit logs to CSV
- Logs retained for minimum 1 year


### US-7.2: System Configuration
**As an** Admin  
**I want to** configure system settings  
**So that** I can customize system behavior

**Acceptance Criteria:**
- Configuration interface for:
  - Session timeout duration
  - Password requirements
  - Maximum file upload size
  - Maximum images per report
  - Notification settings
  - Reminder frequency
  - Archive retention period
- Changes take effect immediately
- Configuration changes logged in audit trail
- Default values available for reset
- Validation for configuration values
- Confirmation required for critical changes

---

### US-7.3: Backup and Restore
**As an** Admin  
**I want to** backup and restore system data  
**So that** I can protect against data loss

**Acceptance Criteria:**
- Manual backup trigger available
- Automated daily backups scheduled
- Backup includes: All report data, user accounts, images, system configuration
- Backup stored securely with encryption
- Restore functionality with point-in-time selection
- Backup history showing date/time, size, status
- Test restore capability
- Backup retention: 30 days

---

### US-6.1: Generate Report Statistics
**As an** Admin  
**I want to** view system statistics  
**So that** I can monitor system usage and performance

**Acceptance Criteria:**
- Statistics dashboard showing:
  - Total reports created (by date range)
  - Reports by status (Draft, Pending, Completed)
  - Average time to complete signatures
  - Reports by branch
  - Reports by activity type
  - Most active users
- Date range selector (Last 7 days, Last 30 days, Last 90 days, Custom)
- Visual charts and graphs (bar, pie, line charts)
- Export statistics to Excel/CSV
- Refresh button to update data

---

## Database Tables Owned by This Unit

### AuditLogs Table
| Column | Type | Constraints | Description |
|--------|------|-------------|-------------|
| LogID | BIGINT | PRIMARY KEY, IDENTITY | Unique log entry ID |
| LogTimestamp | DATETIME | NOT NULL, DEFAULT GETDATE() | When event occurred |
| UserID | INT | FOREIGN KEY (Users.UserID), NULL | User who performed action |
| Username | NVARCHAR(50) | NULL | Username (for deleted users) |
| ActionType | NVARCHAR(100) | NOT NULL | Type of action performed |
| EntityType | NVARCHAR(50) | NULL | Type of entity affected |
| EntityID | NVARCHAR(50) | NULL | ID of affected entity |
| ActionDetails | NVARCHAR(MAX) | NULL | Detailed description |
| IPAddress | NVARCHAR(50) | NULL | Source IP address |
| Result | NVARCHAR(20) | NOT NULL | Success, Failure, Error |

### SystemConfiguration Table
| Column | Type | Constraints | Description |
|--------|------|-------------|-------------|
| ConfigID | INT | PRIMARY KEY, IDENTITY | Unique config ID |
| ConfigKey | NVARCHAR(100) | UNIQUE, NOT NULL | Configuration key |
| ConfigValue | NVARCHAR(500) | NOT NULL | Configuration value |
| DataType | NVARCHAR(20) | NOT NULL | Int, String, Boolean, Decimal |
| DefaultValue | NVARCHAR(500) | NOT NULL | Default value |
| Description | NVARCHAR(500) | NULL | Configuration description |
| LastModifiedBy | INT | FOREIGN KEY (Users.UserID) | Admin who modified |
| LastModifiedDate | DATETIME | NULL | Last modification date |

### BackupHistory Table
| Column | Type | Constraints | Description |
|--------|------|-------------|-------------|
| BackupID | INT | PRIMARY KEY, IDENTITY | Unique backup ID |
| BackupDate | DATETIME | NOT NULL, DEFAULT GETDATE() | Backup timestamp |
| BackupType | NVARCHAR(20) | NOT NULL | Manual, Automatic |
| BackupPath | NVARCHAR(500) | NOT NULL | Backup file location |
| BackupSize | BIGINT | NOT NULL | Size in bytes |
| BackupStatus | NVARCHAR(20) | NOT NULL | Success, Failed, InProgress |
| InitiatedBy | INT | FOREIGN KEY (Users.UserID), NULL | Admin who initiated |
| CompletedDate | DATETIME | NULL | Completion timestamp |

### SystemStatistics Table
| Column | Type | Constraints | Description |
|--------|------|-------------|-------------|
| StatID | INT | PRIMARY KEY, IDENTITY | Unique stat ID |
| StatDate | DATE | NOT NULL | Date of statistics |
| TotalReports | INT | NOT NULL, DEFAULT 0 | Total reports created |
| DraftReports | INT | NOT NULL, DEFAULT 0 | Reports in draft |
| PendingReports | INT | NOT NULL, DEFAULT 0 | Reports pending signatures |
| CompletedReports | INT | NOT NULL, DEFAULT 0 | Completed reports |
| TotalUsers | INT | NOT NULL, DEFAULT 0 | Total active users |
| ActiveUsers | INT | NOT NULL, DEFAULT 0 | Users who logged in |
| TotalSignatures | INT | NOT NULL, DEFAULT 0 | Signatures submitted |
| AvgSignatureTime | DECIMAL(10,2) | NULL | Avg hours to complete |

---

## Service Interfaces Exposed

### Audit Logging Operations
- **LogAction**(userId, actionType, entityType, entityId, details, result) → Success
- **GetAuditLogs**(filters) → AuditLogList
- **ExportAuditLogs**(filters, format) → File

### Configuration Operations
- **GetConfiguration**(configKey) → ConfigValue
- **SetConfiguration**(configKey, configValue, userId) → Success
- **GetAllConfigurations**() → ConfigurationList
- **ResetConfiguration**(configKey) → Success

### Backup Operations
- **CreateBackup**(userId, backupType) → BackupId
- **GetBackupHistory**() → BackupList
- **RestoreBackup**(backupId, userId) → Success
- **DeleteOldBackups**() → Success

### Statistics Operations
- **GetSystemStatistics**(dateRange) → StatisticsData
- **GenerateStatisticsReport**(dateRange, format) → File
- **RefreshStatistics**() → Success

---

## Dependencies on Other Units

### Depends On:
- **User Management & Authentication Unit**: User information for audit logs, admin permission validation
- **Report Lifecycle Management Unit**: Report statistics, report data for backup
- **Signature Workflow Engine Unit**: Signature statistics, workflow data for backup

### Consumed By:
- **All Units**: Audit logging, configuration retrieval

---

## Business Rules

1. **Audit Log Retention**: Logs retained for minimum 1 year
2. **Admin Only Access**: Only Admins can access audit logs and system configuration
3. **Immutable Logs**: Audit logs cannot be modified or deleted
4. **Automatic Backups**: Daily backups scheduled automatically
5. **Backup Retention**: Backups retained for 30 days
6. **Configuration Validation**: All configuration changes validated before applying
7. **Configuration Logging**: All configuration changes logged in audit trail
8. **Statistics Refresh**: Statistics updated daily
9. **Encryption Required**: Backups must be encrypted
10. **Restore Confirmation**: Restore operations require explicit confirmation

---

## UI Components (Desktop App)

### Screens/Forms:
1. **Audit Log Viewer**
   - Log list/grid with filtering
   - Search bar
   - Export button
   - Pagination controls

2. **System Configuration Screen**
   - Configuration list grouped by category
   - Edit/Reset buttons
   - Save Changes button

3. **Backup Management Screen**
   - Backup history list
   - Create Backup button
   - Restore/Delete buttons
   - Progress indicator

4. **Statistics Dashboard**
   - Summary cards
   - Charts (pie, bar, line)
   - Date range selector
   - Export button

---

## Notes

- This unit should be built last as it depends on all other units
- Audit logging is critical for compliance and security
- All units should call audit logging for important operations
- Backup operations should be tested regularly
- Statistics should be calculated efficiently
- Admin access should be strictly controlled
