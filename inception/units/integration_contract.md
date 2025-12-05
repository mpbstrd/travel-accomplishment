# Integration Contract - Travel Accomplishment Report System

## Document Overview

**Purpose**: Define the service interfaces and integration points between the four business units of the Travel Accomplishment Report System.

**Architecture**: Modular Monolith with in-process communication
**Database**: Shared MSSQL database with clear table ownership
**Communication**: Synchronous in-process method calls through well-defined interfaces

---

## System Architecture Overview

### Business Units

1. **Unit 1: User Management & Authentication**
   - Foundation unit providing identity and access control
   - Owns: Users, Sessions, Login Attempts, Password History tables

2. **Unit 2: Report Lifecycle Management**
   - Core business functionality for report CRUD operations
   - Owns: Reports, NetworkEquipment, ActivityChecklist, ReportImages, ReportEditHistory tables

3. **Unit 3: Signature Workflow Engine**
   - Manages sequential signature workflow and notifications
   - Owns: ReportSignatures, WorkflowState, Notifications, SignatureAttempts tables

4. **Unit 4: System Administration**
   - Provides audit logging, configuration, backup, and statistics
   - Owns: AuditLogs, SystemConfiguration, BackupHistory, SystemStatistics tables

---

## Unit Dependencies

```
┌─────────────────────────────────────────────────────────────┐
│                  System Administration (Unit 4)              │
│              (Audit Logs, Config, Backup, Stats)             │
└──────────────────────────┬──────────────────────────────────┘
                           │ (consumed by all)
                           │
        ┌──────────────────┴──────────────────┐
        │                                     │
┌───────▼────────────┐              ┌────────▼────────────┐
│  User Management   │◄─────────────┤  Report Lifecycle   │
│  & Authentication  │              │    Management       │
│     (Unit 1)       │              │     (Unit 2)        │
└───────┬────────────┘              └────────┬────────────┘
        │                                     │
        │         ┌───────────────────────────┘
        │         │
        └────────►│
         ┌────────▼────────────┐
         │  Signature Workflow │
         │      Engine         │
         │     (Unit 3)        │
         └─────────────────────┘
```

---

## Unit 1: User Management & Authentication

### Responsibilities
- User account lifecycle (create, read, update, delete)
- Authentication (login, logout, session management)
- Authorization (role-based access control)
- Password management and security

### Service Interfaces Exposed

#### IAuthenticationService
```
AuthenticateUser(username: string, password: string) → UserSession
  - Validates credentials
  - Creates session token
  - Returns: { sessionToken, userId, username, role, expiresAt }
  - Logs authentication attempt

ValidateSession(sessionToken: string) → UserDetails
  - Validates active session
  - Returns: { userId, username, role, email }
  - Returns null if invalid/expired

LogoutUser(sessionToken: string) → Success
  - Invalidates session
  - Logs logout event

ExtendSession(sessionToken: string) → Success
  - Extends session expiration time
```

#### IUserManagementService
```
GetUserById(userId: int) → UserDetails
  - Returns: { userId, username, fullName, email, role, accountStatus }

GetUserByUsername(username: string) → UserDetails
  - Returns user details by username

CreateUser(userDetails: UserCreateRequest) → UserId
  - Admin only
  - Validates uniqueness
  - Hashes password
  - Logs creation

UpdateUser(userId: int, userDetails: UserUpdateRequest) → Success
  - Admin only
  - Logs changes

DeleteUser(userId: int) → Success
  - Admin only (soft delete)
  - Checks for pending reports
  - Logs deletion

ResetPassword(userId: int, newPassword: string) → Success
  - Admin only
  - Validates password requirements
  - Logs password reset
```

#### IAuthorizationService
```
CheckPermission(userId: int, action: string) → Boolean
  - Validates user has permission for action
  - Actions: CreateReport, EditReport, DeleteReport, ManageUsers, ViewAuditLogs, etc.

GetUserRole(userId: int) → Role
  - Returns: Admin or User

IsAdmin(userId: int) → Boolean
  - Quick check for admin role
```

### Data Contracts

#### UserSession
```
{
  sessionToken: string,
  userId: int,
  username: string,
  role: string,
  expiresAt: datetime
}
```

#### UserDetails
```
{
  userId: int,
  username: string,
  fullName: string,
  email: string,
  role: string,
  accountStatus: string,
  lastLoginDate: datetime
}
```

---

## Unit 2: Report Lifecycle Management

### Responsibilities
- Report creation and data entry
- Report viewing, searching, and filtering
- Image upload and storage
- Report editing (Admin) and deletion (Admin)
- Export functionality (PDF, CSV)

### Service Interfaces Exposed

#### IReportManagementService
```
CreateReport(userId: int, reportData: ReportCreateRequest) → ReportId
  - Generates unique report ID
  - Sets status to Draft
  - Sets creator as Prepared By
  - Returns report ID

UpdateReport(reportId: int, reportData: ReportUpdateRequest) → Success
  - Validates report is in Draft status or user is Admin
  - Updates report data
  - Logs changes if Admin edit

GetReportById(reportId: int) → ReportDetails
  - Returns complete report information
  - Includes all sections, images, signatures

GetReportsByUser(userId: int) → ReportList
  - Returns all reports created by user

SearchReports(searchCriteria: SearchRequest) → ReportList
  - Filters: dateRange, status, branch, activityType, createdBy, etc.
  - Returns paginated results

DeleteReport(reportId: int, userId: int, reason: string) → Success
  - Admin only (soft delete)
  - Logs deletion with reason

RestoreReport(reportId: int) → Success
  - Admin only
  - Restores archived report
```

#### IReportStatusService
```
GetReportStatus(reportId: int) → Status
  - Returns: Draft, Pending Branch, Pending NISD, Completed

UpdateReportStatus(reportId: int, newStatus: string) → Success
  - Called by Signature Workflow unit
  - Updates report status

ValidateReportComplete(reportId: int) → Boolean
  - Checks all required fields filled
  - Validates checklist completion
  - Returns true if ready for signature
```

#### IImageManagementService
```
UploadImage(reportId: int, imageFile: FileData, checklistItem: int) → ImageId
  - Validates file type and size
  - Stores image file
  - Records metadata
  - Returns image ID

GetReportImages(reportId: int) → ImageList
  - Returns all images for report
  - Includes: imageId, fileName, fileSize, uploadedDate, checklistItem

DeleteImage(imageId: int) → Success
  - Removes image from report
  - Deletes file from storage
```

#### IReportExportService
```
ExportReportToPDF(reportId: int) → PDFFile
  - Generates formatted PDF
  - Includes all data, images, signatures

ExportReportsToCSV(reportIds: int[]) → CSVFile
  - Exports report list to CSV
  - Includes filterable fields
```

### Data Contracts

#### ReportDetails
```
{
  reportId: int,
  reportNumber: string,
  branchName: string,
  dateOfTravel: date,
  timeStarted: time,
  timeEnded: time,
  purposeOfTravel: string,
  reportStatus: string,
  createdBy: int,
  createdDate: datetime,
  networkEquipment: NetworkEquipmentData,
  activityChecklist: ChecklistData,
  images: ImageData[],
  signatures: SignatureData[]
}
```

---

## Unit 3: Signature Workflow Engine

### Responsibilities
- Sequential signature workflow enforcement
- Signature validation and capture
- Workflow state management
- In-app notifications for signature requirements

### Service Interfaces Exposed

#### ISignatureService
```
SubmitSignature(reportId: int, userId: int, signatureType: string, signatoryName: string) → Success
  - Validates workflow sequence
  - Validates report completeness (for Step 1)
  - Records signature with timestamp
  - Advances workflow state
  - Updates report status
  - Creates notifications
  - Logs signature event

GetSignatureStatus(reportId: int) → SignatureStatusDetails
  - Returns status of all three steps
  - Returns: { step1Complete, step1Date, step2Complete, step2Date, step3Complete, step3Date }

GetReportSignatures(reportId: int) → SignatureList
  - Returns all signatures for report
  - Includes: signatureType, signatoryName, timestamp

ValidateSignatureEligibility(reportId: int, userId: int, signatureType: string) → Boolean
  - Checks if user can sign at current step
  - Validates workflow sequence
```

#### IWorkflowService
```
InitializeWorkflow(reportId: int) → Success
  - Creates workflow state for new report
  - Sets current step to 1

GetWorkflowState(reportId: int) → WorkflowState
  - Returns current workflow state
  - Returns: { currentStep, step1Complete, step2Complete, step3Complete, workflowStatus }

GetCurrentStep(reportId: int) → StepNumber
  - Returns: 1, 2, 3, or 4 (completed)

IsWorkflowComplete(reportId: int) → Boolean
  - Returns true if all three signatures complete
```

#### INotificationService
```
CreateNotification(userId: int, reportId: int, notificationType: string, message: string) → NotificationId
  - Creates in-app notification
  - Types: SignatureRequired, SignatureCompleted, ReportCompleted, Reminder

GetUserNotifications(userId: int) → NotificationList
  - Returns all notifications for user
  - Includes read/unread status

GetUnreadNotificationCount(userId: int) → Count
  - Returns count of unread notifications

MarkNotificationAsRead(notificationId: int) → Success
  - Marks notification as read

DismissNotification(notificationId: int) → Success
  - Dismisses notification

GetPendingSignatureReports(userId: int) → ReportList
  - Returns reports awaiting user's signature
```

### Data Contracts

#### SignatureStatusDetails
```
{
  reportId: int,
  currentStep: int,
  step1: { complete: boolean, signatoryName: string, timestamp: datetime },
  step2: { complete: boolean, signatoryName: string, timestamp: datetime },
  step3: { complete: boolean, signatoryName: string, timestamp: datetime },
  workflowStatus: string
}
```

#### NotificationData
```
{
  notificationId: int,
  userId: int,
  reportId: int,
  notificationType: string,
  message: string,
  isRead: boolean,
  createdDate: datetime
}
```

---

## Unit 4: System Administration

### Responsibilities
- Audit logging for all system activities
- System configuration management
- Backup and restore operations
- System statistics and reporting

### Service Interfaces Exposed

#### IAuditLogService
```
LogAction(userId: int, actionType: string, entityType: string, entityId: string, details: string, result: string) → Success
  - Records audit log entry
  - Captures: timestamp, user, action, entity, IP address, result
  - Called by all units for important operations

GetAuditLogs(filters: AuditLogFilters) → AuditLogList
  - Filters: dateRange, userId, actionType, entityType, result
  - Returns paginated results
  - Admin only

ExportAuditLogs(filters: AuditLogFilters, format: string) → File
  - Exports audit logs to CSV
  - Admin only
```

#### IConfigurationService
```
GetConfiguration(configKey: string) → ConfigValue
  - Returns configuration value
  - Used by all units

SetConfiguration(configKey: string, configValue: string, userId: int) → Success
  - Updates configuration
  - Validates value
  - Logs change
  - Admin only

GetAllConfigurations() → ConfigurationList
  - Returns all configurations
  - Admin only

ResetConfiguration(configKey: string) → Success
  - Resets to default value
  - Admin only
```

#### IBackupService
```
CreateBackup(userId: int, backupType: string) → BackupId
  - Initiates backup operation
  - Types: Manual, Automatic
  - Returns backup ID
  - Admin only

GetBackupHistory() → BackupList
  - Returns backup history
  - Admin only

RestoreBackup(backupId: int, userId: int) → Success
  - Restores from backup
  - Requires confirmation
  - Admin only

DeleteOldBackups() → Success
  - Removes backups older than retention period
  - Called by scheduled job
```

#### IStatisticsService
```
GetSystemStatistics(dateRange: DateRange) → StatisticsData
  - Returns system statistics
  - Includes: report counts, user activity, signature metrics
  - Admin only

GenerateStatisticsReport(dateRange: DateRange, format: string) → File
  - Generates statistics report (CSV/Excel)
  - Admin only

RefreshStatistics() → Success
  - Recalculates current statistics
  - Called by scheduled job
```

### Data Contracts

#### AuditLogEntry
```
{
  logId: long,
  logTimestamp: datetime,
  userId: int,
  username: string,
  actionType: string,
  entityType: string,
  entityId: string,
  actionDetails: string,
  ipAddress: string,
  result: string
}
```

#### ConfigurationItem
```
{
  configKey: string,
  configValue: string,
  dataType: string,
  defaultValue: string,
  description: string
}
```

---

## Integration Patterns

### Pattern 1: User Authentication Flow
```
1. User enters credentials in UI
2. UI calls Unit1.AuthenticationService.AuthenticateUser()
3. Unit1 validates credentials
4. Unit1 calls Unit4.AuditLogService.LogAction() (login attempt)
5. Unit1 returns session token
6. UI stores session token for subsequent requests
```

### Pattern 2: Report Creation Flow
```
1. User fills report form in UI
2. UI calls Unit1.AuthenticationService.ValidateSession()
3. UI calls Unit2.ReportManagementService.CreateReport()
4. Unit2 creates report in Draft status
5. Unit2 calls Unit3.WorkflowService.InitializeWorkflow()
6. Unit2 calls Unit4.AuditLogService.LogAction() (report created)
7. Unit2 returns report ID
```

### Pattern 3: Signature Submission Flow
```
1. User submits signature in UI
2. UI calls Unit1.AuthenticationService.ValidateSession()
3. UI calls Unit3.SignatureService.SubmitSignature()
4. Unit3 calls Unit2.ReportStatusService.ValidateReportComplete() (for Step 1)
5. Unit3 validates workflow sequence
6. Unit3 records signature
7. Unit3 calls Unit2.ReportStatusService.UpdateReportStatus()
8. Unit3 calls Unit3.NotificationService.CreateNotification() (next signatory)
9. Unit3 calls Unit4.AuditLogService.LogAction() (signature submitted)
10. Unit3 returns success
```

### Pattern 4: Configuration Retrieval Flow
```
1. Any unit needs configuration value
2. Unit calls Unit4.ConfigurationService.GetConfiguration()
3. Unit4 returns configuration value
4. Unit uses value in business logic
```

### Pattern 5: Admin Edit Report Flow
```
1. Admin edits report in UI
2. UI calls Unit1.AuthorizationService.IsAdmin()
3. UI calls Unit2.ReportManagementService.UpdateReport()
4. Unit2 validates admin permission
5. Unit2 updates report data
6. Unit2 calls Unit4.AuditLogService.LogAction() (admin edit with changes)
7. Unit2 calls Unit3.NotificationService.CreateNotification() (notify creator)
8. Unit2 returns success
```

---

## Cross-Cutting Concerns

### Error Handling
- All service methods return consistent error responses
- Errors logged to Unit 4 audit logs
- User-friendly error messages returned to UI
- Technical details logged for debugging

### Transaction Management
- Database transactions managed at unit boundary
- Multi-unit operations use distributed transaction pattern
- Rollback on failure to maintain consistency

### Validation
- Each unit validates its own domain data
- Shared validation rules accessed via Unit 4 configuration
- Validation errors returned with clear messages

### Security
- All service calls validate session token (via Unit 1)
- Permission checks performed before operations
- Sensitive data encrypted at rest
- All security events logged to audit trail

### Performance
- Database queries optimized with indexes
- Pagination used for large result sets
- Caching considered for frequently accessed data (configurations)
- Background jobs for statistics and cleanup operations

---

## Database Integration

### Table Ownership
Each unit owns specific tables and is responsible for:
- Table schema definition
- Data integrity within its tables
- CRUD operations on its tables

### Cross-Unit Data Access
- Units access other units' data through service interfaces only
- No direct database queries across unit boundaries
- Foreign key relationships maintained for referential integrity

### Shared Data
- User information (Unit 1) referenced by all units
- Report information (Unit 2) referenced by Units 3 and 4
- Configuration (Unit 4) accessed by all units

---

## Build and Deployment Order

### Recommended Build Order:
1. **Unit 1: User Management & Authentication** (Foundation)
   - No dependencies on other units
   - Required by all other units

2. **Unit 4: System Administration** (Infrastructure)
   - Depends on Unit 1 for user information
   - Provides audit logging and configuration for other units

3. **Unit 2: Report Lifecycle Management** (Core Business)
   - Depends on Units 1 and 4
   - Core functionality of the system

4. **Unit 3: Signature Workflow Engine** (Business Process)
   - Depends on Units 1, 2, and 4
   - Orchestrates the signature workflow

### Parallel Development:
- Units 2 and 4 can be developed in parallel after Unit 1 is complete
- Unit 3 should be developed after Units 1 and 2 are stable

---

## Testing Strategy

### Unit Testing
- Each unit tested independently
- Mock dependencies on other units
- Test business logic and validation

### Integration Testing
- Test interactions between units
- Verify service contracts
- Test complete workflows (e.g., report creation to completion)

### End-to-End Testing
- Test complete user scenarios
- Verify UI to database flow
- Test all three signature workflow steps

---

## Notes

- All service interfaces are conceptual and should be implemented according to the chosen technology stack
- Communication is in-process (method calls) since this is a modular monolith
- Database transactions should span unit boundaries when necessary for consistency
- Audit logging should be called for all important operations
- Configuration should be cached for performance
- Consider implementing a service locator or dependency injection pattern for unit communication
- All units should handle errors gracefully and log appropriately
- Session validation should be performed at the UI layer before calling business services
- Consider implementing retry logic for transient failures
- Monitor performance of cross-unit calls and optimize as needed

---

**Document Version**: 1.0  
**Date**: December 5, 2025  
**Status**: Ready for Architecture Design Phase
