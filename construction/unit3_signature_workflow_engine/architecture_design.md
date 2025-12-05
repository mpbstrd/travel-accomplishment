# Unit 3: Signature Workflow Engine - Architecture Design

## 1. Unit Overview

**Business Capability**: Manage the sequential three-step signature workflow for Travel Accomplishment Reports.

**Responsibilities**:
- Enforce sequential signature workflow (Prepared By → Branch Acknowledgement → NISD Acknowledgement)
- Capture and validate signatures with timestamps
- Manage workflow state transitions
- Provide in-app notifications for signature requirements
- Track signature attempts and completions

**Scope**:
- Sequential workflow enforcement (no skipping steps)
- Signature disclaimer and entry
- Workflow state management
- In-app notification system
- Signature status tracking and display

**Technology Stack**:
- Platform: C# .NET 8 Windows Forms
- Data Access: Entity Framework Core with SQLite
- Dependency Injection: Microsoft.Extensions.DependencyInjection
- Background Jobs: System.Threading.Timer for reminders

---

## 2. Feature Components

### 2.1 Signature Entry Feature
**Responsibility**: Capture signatures with disclaimer acceptance

**UI Components**:
- SignatureDialog (Modal Dialog)
  - Disclaimer text display (prominent)
  - Disclaimer acceptance checkbox
  - Signatory name text box
  - Password confirmation field (optional security)
  - Sign button (enabled after disclaimer accepted)
  - Cancel button
  - Validation error labels

**Feature Operations**:
- Display signature disclaimer
- Validate disclaimer acceptance
- Validate signatory name
- Capture signature with timestamp
- Submit signature to workflow
- Display confirmation message

### 2.2 Signature Workflow Display Feature
**Responsibility**: Show workflow progress and status

**UI Components**:
- SignatureWorkflowPanel (User Control)
  - Three-step progress indicator
  - Step 1: Prepared By section
    - Status icon (completed/pending)
    - Signatory name (if completed)
    - Timestamp (if completed)
    - Sign button (if current step)
  - Step 2: Branch Acknowledgement section
    - Status icon (completed/pending/locked)
    - Signatory name (if completed)
    - Timestamp (if completed)
    - Sign button (if current step)
  - Step 3: NISD Acknowledgement section
    - Status icon (completed/pending/locked)
    - Signatory name (if completed)
    - Timestamp (if completed)
    - Sign button (if current step)
  - Overall workflow status label

**Feature Operations**:
- Display current workflow state
- Show completed signatures
- Highlight current step
- Enable/disable sign buttons based on workflow state
- Refresh workflow status

### 2.3 Notification Management Feature
**Responsibility**: Display and manage in-app notifications

**UI Components**:
- NotificationPanel (User Control)
  - Notification list
  - Unread count badge
  - Notification items:
    - Report ID and branch
    - Notification message
    - Timestamp
    - Link to report button
    - Mark as read button
    - Dismiss button
  - Clear all button
  - Filter (All/Unread/Signature Required)

**Feature Operations**:
- Display notifications
- Show unread count
- Mark notifications as read
- Dismiss notifications
- Navigate to report from notification
- Filter notifications

### 2.4 Pending Signatures Dashboard Feature
**Responsibility**: Show reports awaiting user's signature

**UI Components**:
- PendingSignaturesDashboard (User Control)
  - Report list requiring signature
  - Report cards with:
    - Report ID and branch
    - Date of travel
    - Current workflow step
    - Days pending
    - Quick sign button
  - Sort options (date, branch, days pending)
  - Pagination controls

**Feature Operations**:
- List reports awaiting signature
- Sort and filter pending reports
- Quick navigation to report
- Display pending duration


---

## 3. Business Services

### 3.1 SignatureService
**Responsibility**: Handle signature submission and validation

**Public Operations**:
- SubmitSignatureAsync(reportId, userId, signatureType, signatoryName) → SignatureResult
  - Validates workflow sequence (correct step)
  - Validates report completeness (for Step 1)
  - Validates disclaimer acceptance
  - Validates signatory name format
  - Records signature with server timestamp
  - Advances workflow state
  - Updates report status
  - Creates notifications for next signatory
  - Logs signature event to audit
  - Returns success/failure result
  
- GetSignatureStatusAsync(reportId) → SignatureStatusDto
  - Returns status of all three signature steps
  - Includes signatory names and timestamps
  - Returns current workflow step
  
- GetReportSignaturesAsync(reportId) → List<SignatureDto>
  - Returns all signatures for report
  - Ordered by signature type (Step 1, 2, 3)
  
- ValidateSignatureEligibilityAsync(reportId, userId, signatureType) → ValidationResult
  - Checks if user can sign at current step
  - Validates workflow sequence
  - Checks if step already signed
  - Returns validation result with errors
  
- CanUserSignReportAsync(reportId, userId) → bool
  - Quick check if user can sign report at current step
  - Used for UI button enabling

**Dependencies**:
- IReportSignatureRepository
- IWorkflowStateRepository
- ISignatureAttemptRepository
- IReportStatusService (Unit 2)
- IUserManagementService (Unit 1)
- INotificationService (internal)
- IAuditLogService (Unit 4)

### 3.2 WorkflowService
**Responsibility**: Manage workflow state and transitions

**Public Operations**:
- InitializeWorkflowAsync(reportId) → bool
  - Creates workflow state for new report
  - Sets current step to 1
  - Sets all steps as incomplete
  - Returns success status
  
- GetWorkflowStateAsync(reportId) → WorkflowStateDto
  - Returns current workflow state
  - Includes step completion status
  - Returns current step number
  
- AdvanceWorkflowAsync(reportId, completedStep) → bool
  - Advances workflow to next step
  - Updates step completion flags
  - Updates current step number
  - Validates step sequence
  - Returns success status
  
- GetCurrentStepAsync(reportId) → int
  - Returns current step number (1, 2, 3, or 4 if completed)
  
- IsWorkflowCompleteAsync(reportId) → bool
  - Returns true if all three signatures complete
  
- GetWorkflowProgressAsync(reportId) → WorkflowProgressDto
  - Returns detailed progress information
  - Includes completion percentage
  - Includes estimated completion time

**Dependencies**:
- IWorkflowStateRepository
- IReportSignatureRepository

### 3.3 NotificationService
**Responsibility**: Manage in-app notifications

**Public Operations**:
- CreateNotificationAsync(userId, reportId, notificationType, message) → int (notificationId)
  - Creates new notification
  - Types: SignatureRequired, SignatureCompleted, ReportCompleted, Reminder
  - Returns notification ID
  
- GetUserNotificationsAsync(userId, includeRead) → List<NotificationDto>
  - Returns all notifications for user
  - Optionally filters to unread only
  - Ordered by date (newest first)
  
- GetUnreadNotificationCountAsync(userId) → int
  - Returns count of unread notifications
  - Used for badge display
  
- MarkNotificationAsReadAsync(notificationId) → bool
  - Marks notification as read
  - Updates read timestamp
  
- MarkAllAsReadAsync(userId) → bool
  - Marks all user notifications as read
  
- DismissNotificationAsync(notificationId) → bool
  - Dismisses notification
  - Updates dismissed flag and timestamp
  
- GetPendingSignatureReportsAsync(userId) → List<ReportSummaryDto>
  - Returns reports awaiting user's signature
  - Includes workflow step information
  - Ordered by date (oldest first)
  
- CreateSignatureRequiredNotificationAsync(reportId, nextSignatoryUserId) → int
  - Creates notification for next signatory
  - Includes report details
  - Returns notification ID
  
- CreateSignatureCompletedNotificationAsync(reportId, signatoryName, role) → void
  - Notifies previous signatories of completion
  - Notifies report creator
  
- CreateReportCompletedNotificationAsync(reportId) → void
  - Notifies all signatories of final completion
  - Marks workflow as complete

**Dependencies**:
- INotificationRepository
- IReportManagementService (Unit 2)
- IUserManagementService (Unit 1)

### 3.4 SignatureReminderService
**Responsibility**: Generate reminders for pending signatures

**Public Operations**:
- CheckPendingSignaturesAsync() → void
  - Background job method
  - Checks all reports with pending signatures
  - Creates reminders for signatures pending > 24 hours
  - Runs periodically (e.g., every 6 hours)
  
- CreateReminderNotificationAsync(reportId, userId) → int
  - Creates reminder notification
  - Checks if reminder already sent recently
  - Returns notification ID
  
- GetReportsNeedingRemindersAsync() → List<int> (reportIds)
  - Returns reports with signatures pending > 24 hours
  - Excludes reports with recent reminders

**Dependencies**:
- IWorkflowStateRepository
- INotificationRepository
- IReportManagementService (Unit 2)

### 3.5 SignatureValidationService
**Responsibility**: Validate signature data and workflow rules

**Public Operations**:
- ValidateSignatoryNameAsync(name) → ValidationResult
  - Validates name format (letters and spaces only)
  - Validates length (2-100 characters)
  - Returns validation result
  
- ValidateWorkflowSequenceAsync(reportId, signatureType) → ValidationResult
  - Validates signature is for correct step
  - Checks previous steps completed
  - Checks current step not already signed
  - Returns validation result
  
- ValidateReportReadyForSignatureAsync(reportId) → ValidationResult
  - Validates report completeness (for Step 1)
  - Checks all required fields filled
  - Checks checklist complete
  - Returns validation result

**Dependencies**:
- IWorkflowStateRepository
- IReportSignatureRepository
- IReportStatusService (Unit 2)

---

## 4. Data Models

### 4.1 ReportSignature Entity
**Maps to**: ReportSignatures table

**Properties**:
- SignatureId (int, primary key)
- ReportId (int, foreign key, required)
- SignatureType (string, required, max 50) - "PreparedBy", "BranchAck", "NISDack"
- SignatoryUserId (int, foreign key, required)
- SignatoryName (string, required, max 100)
- SignatureTimestamp (DateTime, required)
- DisclaimerAccepted (bool, required)
- IPAddress (string, nullable, max 50)
- WorkstationName (string, nullable, max 100)

**Relationships**:
- Report (Report from Unit 2) - navigation property
- SignatoryUser (User from Unit 1) - navigation property

**Validation Rules**:
- SignatureType: must be "PreparedBy", "BranchAck", or "NISDack"
- SignatoryName: 2-100 characters, letters and spaces only
- DisclaimerAccepted: must be true
- SignatureTimestamp: server-generated, not user-provided

### 4.2 WorkflowState Entity
**Maps to**: WorkflowState table

**Properties**:
- WorkflowId (int, primary key)
- ReportId (int, foreign key, required, unique)
- CurrentStep (int, required) - 1, 2, 3, or 4 (completed)
- Step1_Completed (bool, required, default false)
- Step1_CompletedDate (DateTime, nullable)
- Step2_Completed (bool, required, default false)
- Step2_CompletedDate (DateTime, nullable)
- Step3_Completed (bool, required, default false)
- Step3_CompletedDate (DateTime, nullable)
- WorkflowStatus (string, required, max 50) - "InProgress" or "Completed"
- CreatedDate (DateTime, required)
- CompletedDate (DateTime, nullable)

**Relationships**:
- Report (Report from Unit 2) - navigation property

**Business Logic**:
- CurrentStep advances as signatures are completed
- Step completion flags set when signature recorded
- WorkflowStatus changes to "Completed" when Step 3 done
- CompletedDate set when workflow finishes

### 4.3 Notification Entity
**Maps to**: Notifications table

**Properties**:
- NotificationId (int, primary key)
- UserId (int, foreign key, required)
- ReportId (int, foreign key, required)
- NotificationType (string, required, max 50)
- NotificationMessage (string, required, max 500)
- IsRead (bool, required, default false)
- CreatedDate (DateTime, required)
- ReadDate (DateTime, nullable)
- IsDismissed (bool, required, default false)
- DismissedDate (DateTime, nullable)

**Relationships**:
- User (User from Unit 1) - navigation property
- Report (Report from Unit 2) - navigation property

**Validation Rules**:
- NotificationType: must be valid type (SignatureRequired, SignatureCompleted, ReportCompleted, Reminder)
- NotificationMessage: required, max 500 characters

### 4.4 SignatureAttempt Entity
**Maps to**: SignatureAttempts table

**Properties**:
- AttemptId (int, primary key)
- ReportId (int, foreign key, required)
- UserId (int, foreign key, required)
- SignatureType (string, required, max 50)
- AttemptDate (DateTime, required)
- IsSuccessful (bool, required)
- FailureReason (string, nullable, max 255)

**Relationships**:
- Report (Report from Unit 2) - navigation property
- User (User from Unit 1) - navigation property

**Business Logic**:
- Records all signature attempts (success and failure)
- Used for audit trail
- Used for troubleshooting workflow issues

---

## 5. Repository Pattern

### 5.1 IReportSignatureRepository Interface
**Responsibility**: Data access for ReportSignature entity

**Operations**:
- GetByIdAsync(signatureId) → ReportSignature
- GetByReportIdAsync(reportId) → List<ReportSignature>
- GetByReportAndTypeAsync(reportId, signatureType) → ReportSignature
- CreateAsync(signature) → ReportSignature
- ExistsAsync(reportId, signatureType) → bool
- GetSignatureCountAsync(reportId) → int

**Implementation**: ReportSignatureRepository (EF Core)
- Uses DbContext for database operations
- Implements async operations
- Includes related entities (User, Report)

### 5.2 IWorkflowStateRepository Interface
**Responsibility**: Data access for WorkflowState entity

**Operations**:
- GetByReportIdAsync(reportId) → WorkflowState
- CreateAsync(workflowState) → WorkflowState
- UpdateAsync(workflowState) → WorkflowState
- GetInProgressWorkflowsAsync() → List<WorkflowState>
- GetPendingSignaturesAsync(olderThanHours) → List<WorkflowState>

**Implementation**: WorkflowStateRepository (EF Core)
- One-to-one relationship with Report
- Tracks workflow progress

### 5.3 INotificationRepository Interface
**Responsibility**: Data access for Notification entity

**Operations**:
- GetByIdAsync(notificationId) → Notification
- GetByUserIdAsync(userId, includeRead) → List<Notification>
- GetUnreadCountAsync(userId) → int
- CreateAsync(notification) → Notification
- UpdateAsync(notification) → Notification
- MarkAsReadAsync(notificationId) → bool
- MarkAllAsReadAsync(userId) → bool
- DismissAsync(notificationId) → bool
- GetRecentReminderAsync(reportId, userId, withinHours) → Notification

**Implementation**: NotificationRepository (EF Core)
- Manages notification lifecycle
- Supports filtering and sorting

### 5.4 ISignatureAttemptRepository Interface
**Responsibility**: Data access for SignatureAttempt entity

**Operations**:
- CreateAsync(attempt) → SignatureAttempt
- GetByReportIdAsync(reportId) → List<SignatureAttempt>
- GetByUserIdAsync(userId) → List<SignatureAttempt>
- GetFailedAttemptsAsync(reportId, userId) → List<SignatureAttempt>

**Implementation**: SignatureAttemptRepository (EF Core)
- Tracks all signature attempts
- Used for audit and troubleshooting

---

## 6. Service Interfaces (Exposed to Other Units)

### 6.1 ISignatureService Interface
**Exposed Operations**:
- SubmitSignatureAsync(reportId, userId, signatureType, signatoryName) → SignatureResult
- GetSignatureStatusAsync(reportId) → SignatureStatusDto
- GetReportSignaturesAsync(reportId) → List<SignatureDto>
- ValidateSignatureEligibilityAsync(reportId, userId, signatureType) → ValidationResult

**Data Contracts**:

**SignatureResult**:
- Success (bool)
- SignatureId (int)
- NewReportStatus (string)
- ErrorMessage (string)

**SignatureStatusDto**:
- ReportId (int)
- CurrentStep (int)
- Step1 (SignatureStepDto)
- Step2 (SignatureStepDto)
- Step3 (SignatureStepDto)
- WorkflowStatus (string)

**SignatureStepDto**:
- IsCompleted (bool)
- SignatoryName (string)
- SignatureTimestamp (DateTime?)
- IsCurrentStep (bool)

**SignatureDto**:
- SignatureId (int)
- SignatureType (string)
- SignatoryName (string)
- SignatureTimestamp (DateTime)

### 6.2 IWorkflowService Interface
**Exposed Operations**:
- InitializeWorkflowAsync(reportId) → bool
- GetWorkflowStateAsync(reportId) → WorkflowStateDto
- GetCurrentStepAsync(reportId) → int
- IsWorkflowCompleteAsync(reportId) → bool

**Data Contracts**:

**WorkflowStateDto**:
- ReportId (int)
- CurrentStep (int)
- Step1Completed (bool)
- Step1CompletedDate (DateTime?)
- Step2Completed (bool)
- Step2CompletedDate (DateTime?)
- Step3Completed (bool)
- Step3CompletedDate (DateTime?)
- WorkflowStatus (string)
- CompletedDate (DateTime?)

### 6.3 INotificationService Interface
**Exposed Operations**:
- CreateNotificationAsync(userId, reportId, notificationType, message) → int
- GetUserNotificationsAsync(userId, includeRead) → List<NotificationDto>
- GetUnreadNotificationCountAsync(userId) → int
- MarkNotificationAsReadAsync(notificationId) → bool
- DismissNotificationAsync(notificationId) → bool
- GetPendingSignatureReportsAsync(userId) → List<ReportSummaryDto>

**Data Contracts**:

**NotificationDto**:
- NotificationId (int)
- ReportId (int)
- ReportNumber (string)
- BranchName (string)
- NotificationType (string)
- NotificationMessage (string)
- IsRead (bool)
- CreatedDate (DateTime)
- ReadDate (DateTime?)

---

## 7. Dependencies

### 7.1 Internal Dependencies
- Entity Framework Core (data access)
- System.Threading.Timer (background reminders)
- Microsoft.Extensions.DependencyInjection (DI container)

### 7.2 External Unit Dependencies
**Depends On**:
- Unit 1 (User Management & Authentication):
  - IAuthenticationService for session validation
  - IUserManagementService for user details
  - IAuthorizationService for permission checks
- Unit 2 (Report Lifecycle Management):
  - IReportManagementService for report details
  - IReportStatusService for report validation and status updates
- Unit 4 (System Administration):
  - IAuditLogService for logging signature events
  - IConfigurationService for reminder settings

**Consumed By**:
- Unit 2 (Report Lifecycle Management): Signature information for display
- Unit 4 (System Administration): Signature events for audit logs

### 7.3 Database Dependencies
**Tables Owned**:
- ReportSignatures
- WorkflowState
- Notifications
- SignatureAttempts

**Foreign Key References**:
- ReportSignatures.ReportId → Reports.ReportId (Unit 2)
- ReportSignatures.SignatoryUserId → Users.UserId (Unit 1)
- WorkflowState.ReportId → Reports.ReportId (Unit 2)
- Notifications.UserId → Users.UserId (Unit 1)
- Notifications.ReportId → Reports.ReportId (Unit 2)
- SignatureAttempts.ReportId → Reports.ReportId (Unit 2)
- SignatureAttempts.UserId → Users.UserId (Unit 1)

---

## 8. Component Diagram

```
┌──────────────────────────────────────────────────────────────────┐
│              Unit 3: Signature Workflow Engine                    │
├──────────────────────────────────────────────────────────────────┤
│                                                                   │
│  ┌───────────────────────────────────────────────────────────┐  │
│  │              UI Layer (Windows Forms)                      │  │
│  │  ┌──────────┐  ┌──────────┐  ┌──────────┐  ┌──────────┐  │  │
│  │  │Signature │  │Signature │  │Notifica  │  │Pending   │  │  │
│  │  │Dialog    │  │Workflow  │  │tion      │  │Signature │  │  │
│  │  │          │  │Panel     │  │Panel     │  │Dashboard │  │  │
│  │  └────┬─────┘  └────┬─────┘  └────┬─────┘  └────┬─────┘  │  │
│  └───────┼─────────────┼─────────────┼─────────────┼─────────┘  │
│          │             │             │             │             │
│  ┌───────▼─────────────▼─────────────▼─────────────▼─────────┐  │
│  │              Business Services Layer                        │  │
│  │  ┌──────────────────┐  ┌──────────────────┐               │  │
│  │  │Signature         │  │Workflow          │               │  │
│  │  │Service           │  │Service           │               │  │
│  │  └────────┬─────────┘  └────────┬─────────┘               │  │
│  │           │                     │                          │  │
│  │  ┌────────▼─────────┐  ┌───────▼──────────┐              │  │
│  │  │Notification      │  │SignatureReminder │              │  │
│  │  │Service           │  │Service           │              │  │
│  │  └────────┬─────────┘  └───────┬──────────┘              │  │
│  │           │                     │                          │  │
│  │  ┌────────▼─────────────────────▼──────────┐              │  │
│  │  │SignatureValidationService                │              │  │
│  │  └──────────────────┬───────────────────────┘              │  │
│  └─────────────────────┼──────────────────────────────────────┘  │
│                        │                                          │
│  ┌─────────────────────▼──────────────────────────────────────┐  │
│  │              Repository Layer                               │  │
│  │  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐     │  │
│  │  │ReportSig     │  │WorkflowState │  │Notification  │     │  │
│  │  │Repository    │  │Repository    │  │Repository    │     │  │
│  │  └──────┬───────┘  └──────┬───────┘  └──────┬───────┘     │  │
│  │         │                 │                 │              │  │
│  │  ┌──────▼─────────────────▼─────────────────▼───────────┐ │  │
│  │  │SignatureAttemptRepository                            │ │  │
│  │  └──────────────────────┬───────────────────────────────┘ │  │
│  └─────────────────────────┼─────────────────────────────────┘  │
│                            │                                     │
│  ┌─────────────────────────▼─────────────────────────────────┐  │
│  │              Data Access Layer (EF Core)                   │  │
│  │  ┌────────────────────────────────────────────────────┐   │  │
│  │  │            DbContext (SQLite)                       │   │  │
│  │  │  - ReportSignatures                                 │   │  │
│  │  │  - WorkflowState                                    │   │  │
│  │  │  - Notifications                                    │   │  │
│  │  │  - SignatureAttempts                                │   │  │
│  │  └────────────────────────────────────────────────────┘   │  │
│  └────────────────────────────────────────────────────────────┘  │
│                                                                   │
│  ┌──────────────────────────────────────────────────────────┐   │
│  │         Service Interfaces (Exposed to Other Units)       │   │
│  │  - ISignatureService                                      │   │
│  │  - IWorkflowService                                       │   │
│  │  - INotificationService                                   │   │
│  └──────────────────────────────────────────────────────────┘   │
└──────────────────────────────────────────────────────────────────┘
```

---

## 9. Database Schema


### 9.1 ReportSignatures Table
```
Table: ReportSignatures
Primary Key: SignatureId (INT, IDENTITY)
Indexes:
  - IX_ReportSignatures_ReportId
  - IX_ReportSignatures_SignatoryUserId
  - IX_ReportSignatures_SignatureType
  - UQ_ReportSignatures_ReportId_SignatureType (UNIQUE)

Columns:
  SignatureId         INT             PRIMARY KEY IDENTITY
  ReportId            INT             NOT NULL FOREIGN KEY REFERENCES Reports(ReportId)
  SignatureType       NVARCHAR(50)    NOT NULL CHECK (SignatureType IN ('PreparedBy', 'BranchAck', 'NISDack'))
  SignatoryUserId     INT             NOT NULL FOREIGN KEY REFERENCES Users(UserId)
  SignatoryName       NVARCHAR(100)   NOT NULL
  SignatureTimestamp  DATETIME        NOT NULL DEFAULT CURRENT_TIMESTAMP
  DisclaimerAccepted  BIT             NOT NULL
  IPAddress           NVARCHAR(50)    NULL
  WorkstationName     NVARCHAR(100)   NULL
```

### 9.2 WorkflowState Table
```
Table: WorkflowState
Primary Key: WorkflowId (INT, IDENTITY)
Indexes:
  - IX_WorkflowState_ReportId (UNIQUE)
  - IX_WorkflowState_CurrentStep
  - IX_WorkflowState_WorkflowStatus

Columns:
  WorkflowId          INT             PRIMARY KEY IDENTITY
  ReportId            INT             NOT NULL UNIQUE FOREIGN KEY REFERENCES Reports(ReportId)
  CurrentStep         INT             NOT NULL CHECK (CurrentStep BETWEEN 1 AND 4)
  Step1_Completed     BIT             NOT NULL DEFAULT 0
  Step1_CompletedDate DATETIME        NULL
  Step2_Completed     BIT             NOT NULL DEFAULT 0
  Step2_CompletedDate DATETIME        NULL
  Step3_Completed     BIT             NOT NULL DEFAULT 0
  Step3_CompletedDate DATETIME        NULL
  WorkflowStatus      NVARCHAR(50)    NOT NULL CHECK (WorkflowStatus IN ('InProgress', 'Completed'))
  CreatedDate         DATETIME        NOT NULL DEFAULT CURRENT_TIMESTAMP
  CompletedDate       DATETIME        NULL
```

### 9.3 Notifications Table
```
Table: Notifications
Primary Key: NotificationId (INT, IDENTITY)
Indexes:
  - IX_Notifications_UserId
  - IX_Notifications_ReportId
  - IX_Notifications_IsRead
  - IX_Notifications_CreatedDate

Columns:
  NotificationId      INT             PRIMARY KEY IDENTITY
  UserId              INT             NOT NULL FOREIGN KEY REFERENCES Users(UserId)
  ReportId            INT             NOT NULL FOREIGN KEY REFERENCES Reports(ReportId)
  NotificationType    NVARCHAR(50)    NOT NULL CHECK (NotificationType IN ('SignatureRequired', 'SignatureCompleted', 'ReportCompleted', 'Reminder'))
  NotificationMessage NVARCHAR(500)   NOT NULL
  IsRead              BIT             NOT NULL DEFAULT 0
  CreatedDate         DATETIME        NOT NULL DEFAULT CURRENT_TIMESTAMP
  ReadDate            DATETIME        NULL
  IsDismissed         BIT             NOT NULL DEFAULT 0
  DismissedDate       DATETIME        NULL
```

### 9.4 SignatureAttempts Table
```
Table: SignatureAttempts
Primary Key: AttemptId (INT, IDENTITY)
Indexes:
  - IX_SignatureAttempts_ReportId
  - IX_SignatureAttempts_UserId
  - IX_SignatureAttempts_AttemptDate

Columns:
  AttemptId           INT             PRIMARY KEY IDENTITY
  ReportId            INT             NOT NULL FOREIGN KEY REFERENCES Reports(ReportId)
  UserId              INT             NOT NULL FOREIGN KEY REFERENCES Users(UserId)
  SignatureType       NVARCHAR(50)    NOT NULL
  AttemptDate         DATETIME        NOT NULL DEFAULT CURRENT_TIMESTAMP
  IsSuccessful        BIT             NOT NULL
  FailureReason       NVARCHAR(255)   NULL
```

### 9.5 Relationships
- ReportSignatures.ReportId → Reports.ReportId (many-to-one)
- ReportSignatures.SignatoryUserId → Users.UserId (many-to-one)
- WorkflowState.ReportId → Reports.ReportId (one-to-one)
- Notifications.UserId → Users.UserId (many-to-one)
- Notifications.ReportId → Reports.ReportId (many-to-one)
- SignatureAttempts.ReportId → Reports.ReportId (many-to-one)
- SignatureAttempts.UserId → Users.UserId (many-to-one)

---

## 10. Implementation Notes

### 10.1 Entity Framework Core Configuration
- Use Code-First approach with migrations
- Configure one-to-one relationship (Report-WorkflowState)
- Configure unique constraint on ReportId + SignatureType
- Use async operations throughout
- Configure indexes for query performance
- Set up cascade delete rules (restrict for user references)

### 10.2 Workflow State Management
- Initialize workflow when report submitted for signature
- CurrentStep starts at 1
- Advance CurrentStep only when signature successfully recorded
- Update step completion flags and dates atomically
- Use database transactions for workflow state changes
- Validate workflow state before allowing signature
- Lock workflow state during signature submission (prevent concurrent modifications)

### 10.3 Signature Timestamp
- Always use server timestamp (DateTime.UtcNow or DateTime.Now)
- Never accept timestamp from client
- Store in consistent timezone (UTC recommended)
- Display in local timezone in UI
- Timestamp is immutable once recorded

### 10.4 Signature Validation
- Validate workflow sequence before accepting signature
- Check previous steps completed
- Check current step not already signed
- Validate signatory name format
- Validate disclaimer acceptance
- Record all validation failures in SignatureAttempts table
- Return detailed validation errors to UI

### 10.5 Notification Strategy
- Create notifications immediately after signature
- Notification types:
  - SignatureRequired: When report advances to next step
  - SignatureCompleted: When a signature is recorded
  - ReportCompleted: When all signatures complete
  - Reminder: Daily reminder for pending signatures
- Notifications are in-app only (no email)
- Unread notifications shown with badge count
- Notifications can be dismissed but remain in history

### 10.6 Reminder System
- Background job runs every 6 hours
- Checks for signatures pending > 24 hours
- Creates reminder notification if no recent reminder
- Reminder frequency: once per 24 hours
- User can dismiss reminders
- Dismissed reminders reappear after 24 hours if still pending
- Use System.Threading.Timer for background job

### 10.7 Workflow State Transitions
**State Diagram**:
```
Draft (Report Status)
  ↓ [Submit for Signature]
Step 1: Prepared By (CurrentStep = 1)
  ↓ [Sign as Prepared By]
Step 2: Branch Acknowledgement (CurrentStep = 2)
  ↓ [Sign as Branch Ack]
Step 3: NISD Acknowledgement (CurrentStep = 3)
  ↓ [Sign as NISD Ack]
Completed (CurrentStep = 4, WorkflowStatus = Completed)
```

**Transition Rules**:
1. Step 1 → Step 2: After PreparedBy signature
2. Step 2 → Step 3: After BranchAck signature
3. Step 3 → Completed: After NISDack signature
4. No skipping steps allowed
5. No backward transitions
6. Each step can only be signed once

### 10.8 Concurrent Signature Handling
- Use database transactions for signature submission
- Lock workflow state row during signature
- Check workflow state again inside transaction
- Handle optimistic concurrency conflicts
- Return appropriate error if concurrent modification detected
- Retry logic for transient failures

### 10.9 Error Handling
- Use try-catch blocks in service methods
- Return meaningful error messages to UI
- Log all errors to audit log (Unit 4)
- Record failed attempts in SignatureAttempts table
- Handle database errors (constraint violations, deadlocks)
- Handle workflow state conflicts
- Display user-friendly error messages

### 10.10 Dependency Injection Setup
- Register services with appropriate lifetime:
  - Scoped: Services, Repositories, DbContext
  - Singleton: SignatureReminderService (background job)
  - Transient: Validators
- Use constructor injection throughout
- Register interfaces with implementations
- Configure in Program.cs or Startup class

### 10.11 Testing Considerations
- Unit test services with mocked repositories
- Integration test repositories with in-memory SQLite
- Test workflow state transitions
- Test signature validation rules
- Test concurrent signature attempts
- Test notification creation
- Test reminder generation
- Mock external unit dependencies (Unit 1, 2, 4)

### 10.12 Performance Considerations
- Use async/await throughout
- Implement connection pooling (EF Core default)
- Add indexes on frequently queried columns
- Use Include() for eager loading related entities
- Cache workflow state during signature submission
- Optimize notification queries (pagination)
- Background job runs at low-priority intervals

### 10.13 Security Best Practices
- Validate all inputs
- Use parameterized queries (EF Core default)
- Validate user permissions before signature
- Log all signature attempts
- Capture IP address and workstation name
- Prevent signature tampering (immutable once recorded)
- Validate session before signature submission

---

## 11. Integration Points

### 11.1 With Unit 1 (User Management & Authentication)
- Call IAuthenticationService.ValidateSessionAsync() before signature operations
- Call IAuthorizationService.CheckPermissionAsync() for signature permissions
- Call IUserManagementService.GetUserByIdAsync() for signatory details
- Receive user context (userId) from authenticated session

### 11.2 With Unit 2 (Report Lifecycle Management)
- Call IReportStatusService.ValidateReportCompleteAsync() before Step 1 signature
- Call IReportStatusService.UpdateReportStatusAsync() after each signature
- Call IReportManagementService.GetReportByIdAsync() for report details in notifications
- Receive workflow initialization request when report submitted

### 11.3 With Unit 4 (System Administration)
- Call IAuditLogService.LogActionAsync() for:
  - Signature submissions (success/failure)
  - Workflow state changes
  - Notification creation
  - Reminder generation
- Call IConfigurationService.GetConfigurationAsync() for:
  - Reminder frequency
  - Reminder threshold (24 hours)
  - Notification settings

### 11.4 Application Startup
- Initialize DbContext with SQLite connection
- Run EF Core migrations for Workflow tables
- Register all services in DI container
- Start SignatureReminderService background job
- Configure reminder job interval (6 hours)

---

## 12. Build Order and Dependencies

### 12.1 Build Order
1. Data Models (Entities)
2. Repository Interfaces
3. Repository Implementations
4. Service Interfaces
5. Validation Services
6. Workflow Service
7. Signature Service
8. Notification Service
9. Reminder Service (background job)
10. UI Components
11. Integration Testing

### 12.2 Critical Path
- Build after Unit 1 (User Management) and Unit 2 (Report Lifecycle)
- Depends on both units for core functionality
- Orchestrates the signature workflow process
- Required for report completion

---

## 13. Workflow Scenarios

### 13.1 Happy Path: Complete Workflow
1. User creates report (Unit 2)
2. User submits report for signature
3. Unit 2 calls IWorkflowService.InitializeWorkflowAsync()
4. Workflow created with CurrentStep = 1
5. User signs as "Prepared By"
6. Signature validated and recorded
7. Workflow advances to Step 2
8. Report status updated to "Pending Branch"
9. Notification created for Branch Acknowledger
10. Branch user receives notification
11. Branch user signs as "Branch Acknowledgement"
12. Workflow advances to Step 3
13. Report status updated to "Pending NISD"
14. Notification created for NISD Acknowledger
15. NISD user receives notification
16. NISD user signs as "NISD Acknowledgement"
17. Workflow completes (CurrentStep = 4)
18. Report status updated to "Completed"
19. Completion notifications sent to all signatories

### 13.2 Validation Failure Scenario
1. User attempts to sign report
2. Validation checks workflow state
3. Previous step not completed
4. Validation fails with error message
5. Attempt recorded in SignatureAttempts table
6. Error displayed to user
7. Signature not recorded
8. Workflow state unchanged

### 13.3 Reminder Scenario
1. Report submitted for signature (Step 2)
2. 24 hours pass without signature
3. Background job runs
4. Detects pending signature > 24 hours
5. Creates reminder notification
6. User sees reminder in notification panel
7. User clicks notification link
8. Navigates to report
9. User signs report
10. Reminder dismissed automatically

### 13.4 Concurrent Signature Attempt
1. Two users attempt to sign same step simultaneously
2. First user's transaction locks workflow state
3. First user's signature recorded successfully
4. Workflow state updated
5. Transaction commits
6. Second user's transaction detects state change
7. Validation fails (step already signed)
8. Second user receives error message
9. Second user's attempt recorded as failed

---

## 14. Future Enhancements

### 14.1 Potential Improvements
- Email notifications (in addition to in-app)
- SMS notifications for urgent signatures
- Signature delegation (assign to another user)
- Signature escalation (auto-escalate after X days)
- Parallel signature workflows (multiple approvers at same step)
- Conditional workflows (different paths based on criteria)
- Signature comments/notes
- Signature rejection with reason
- Workflow templates for different report types
- Signature analytics (average time per step)
- Mobile app for signature on-the-go
- Digital signature with certificate
- Biometric signature capture
- Signature preview before submission

### 14.2 Scalability Considerations
- Move to message queue for notifications (RabbitMQ, Azure Service Bus)
- Implement event-driven architecture
- Use distributed cache for workflow state (Redis)
- Implement CQRS for read/write separation
- Consider workflow engine (Elsa, Workflow Core)
- Implement webhook support for external integrations

---

**Document Version**: 1.0  
**Date**: December 5, 2025  
**Status**: Ready for Implementation  
**Technology**: C# .NET 8, Windows Forms, Entity Framework Core, SQLite
