# Unit 3: Signature Workflow Engine

## Unit Overview

**Business Capability**: Manage the sequential three-step signature workflow for Travel Accomplishment Reports.

**Responsibility**: This unit enforces the sequential signature process, validates signatures, manages workflow state transitions, and provides simple in-app notifications for signature status.

**Scope**:
- Sequential signature workflow enforcement (3 steps)
- Signature disclaimer and entry
- Signature validation and timestamp capture
- Workflow state management
- Simple in-app notifications for signature requirements
- Signature status tracking

---

## User Stories

### US-3.1: Signature Disclaimer and Entry
**As a** User acting as a signatory  
**I want to** see a clear disclaimer before signing  
**So that** I understand the legal implications of my signature

**Acceptance Criteria:**
- Disclaimer text displayed prominently: "By entering your name below, you acknowledge this as your official signature and confirm the accuracy of the information in this report."
- Checkbox to acknowledge disclaimer (required before signature entry)
- Name entry field (text input, required)
- Signature timestamp automatically captured upon submission
- Signature cannot be modified once submitted
- Visual confirmation of successful signature

---

### US-3.2: Prepared By Signature (Step 1)
**As a** User who created the report  
**I want to** sign as "Prepared By"  
**So that** I can certify the report is complete and accurate

**Acceptance Criteria:**
- "Prepared By" signature section is the first signature required
- Signature button/section only enabled when all required fields are completed
- System validates all required data before allowing signature
- Validation errors displayed clearly if data incomplete
- Upon signature:
  - Name entered by user is recorded
  - Timestamp automatically captured (format: YYYY-MM-DD HH:MM:SS)
  - Report status changes to "Pending Branch Acknowledgement"
  - In-app notification created for Branch Acknowledger
- Signature details displayed in report (Name and Timestamp)
- "Prepared By" section becomes read-only after signature
- Report creator can no longer edit report data after signing (only Admin can edit)

---

### US-3.3: Branch Acknowledgement Signature (Step 2)
**As a** Branch representative  
**I want to** sign as "Branch Acknowledgement"  
**So that** I can confirm the branch's review and acceptance of the report

**Acceptance Criteria:**
- "Branch Acknowledgement" signature section only enabled after "Prepared By" is completed
- System enforces sequential workflow (cannot skip to Step 3)
- Branch Acknowledger receives in-app notification when report is ready for their signature
- Signature disclaimer displayed before entry
- Upon signature:
  - Name entered by Branch Acknowledger is recorded
  - Timestamp automatically captured (format: YYYY-MM-DD HH:MM:SS)
  - Report status changes to "Pending NISD Acknowledgement"
  - In-app notification created for NISD Acknowledger
- Signature details displayed in report (Name and Timestamp)
- "Branch Acknowledgement" section becomes read-only after signature
- Previous signatures remain visible and unmodifiable

---

### US-3.4: NISD Acknowledgement Signature (Step 3)
**As a** NISD representative  
**I want to** sign as "NISD Acknowledgement"  
**So that** I can provide final approval and complete the report

**Acceptance Criteria:**
- "NISD Acknowledgement" signature section only enabled after "Branch Acknowledgement" is completed
- System enforces sequential workflow (requires Steps 1 and 2)
- NISD Acknowledger receives in-app notification when report is ready for their signature
- Signature disclaimer displayed before entry
- Upon signature:
  - Name entered by NISD Acknowledger is recorded
  - Timestamp automatically captured (format: YYYY-MM-DD HH:MM:SS)
  - Report status changes to "Completed"
  - Report is finalized and locked
  - In-app notification created for all signatories (completion notification)
- Signature details displayed in report (Name and Timestamp)
- All three signatures visible with timestamps
- Report marked as finalized in the system

---

### US-3.5: Signature Workflow Enforcement
**As the** System  
**I want to** enforce the sequential signature workflow  
**So that** reports follow the proper approval chain

**Acceptance Criteria:**
- Step 2 (Branch Acknowledgement) is disabled until Step 1 (Prepared By) is complete
- Step 3 (NISD Acknowledgement) is disabled until Step 2 (Branch Acknowledgement) is complete
- Visual indicators show which step is current (e.g., highlighting, numbering)
- Completed steps show checkmark or completion indicator
- Pending steps show waiting/locked indicator
- Users cannot bypass or skip signature steps
- System prevents tampering with signature sequence
- Audit log records all signature attempts and completions

---

### US-4.1: Signature Required Notifications (In-App)
**As a** User  
**I want to** receive in-app notifications when a report requires my signature  
**So that** I can promptly review and sign reports

**Acceptance Criteria:**
- In-app notification created when report reaches user's signature step
- Notification includes:
  - Report ID
  - Branch/BLU/LC name
  - Date of travel
  - Link to report
- Notification displayed:
  - In notification panel/badge
  - Counter showing unread notifications
- Notification marked as read when user views the report
- Unread notification counter displayed on dashboard

---

### US-4.2: Signature Completion Notifications (In-App)
**As a** User  
**I want to** receive in-app notifications when signatures are completed  
**So that** I can track report progress

**Acceptance Criteria:**
- In-app notification sent to report creator when each signature is completed
- In-app notification sent to all previous signatories when subsequent signatures are completed
- Notification includes:
  - Report ID
  - Signatory name and role
  - Signature timestamp
  - Current report status
  - Link to report
- Final completion notification sent to all signatories when report is fully approved
- Notification history accessible in user profile

---

### US-4.3: Incomplete Report Reminders (In-App)
**As a** User  
**I want to** receive reminders about incomplete reports  
**So that** I don't forget to complete pending tasks

**Acceptance Criteria:**
- Reminder shown for draft reports not submitted after 48 hours
- Reminder shown for reports awaiting signature after 24 hours
- Reminder displayed on dashboard/notification panel
- Reminder includes report details and direct link
- User can dismiss reminders
- Dismissed reminders reappear after 24 hours if still pending

---

## Database Tables Owned by This Unit

### ReportSignatures Table
| Column | Type | Constraints | Description |
|--------|------|-------------|-------------|
| SignatureID | INT | PRIMARY KEY, IDENTITY | Unique signature ID |
| ReportID | INT | FOREIGN KEY (Reports.ReportID), NOT NULL | Associated report |
| SignatureType | NVARCHAR(50) | NOT NULL | PreparedBy, BranchAck, NISDack |
| SignatoryUserID | INT | FOREIGN KEY (Users.UserID), NOT NULL | User who signed |
| SignatoryName | NVARCHAR(100) | NOT NULL | Name entered as signature |
| SignatureTimestamp | DATETIME | NOT NULL, DEFAULT GETDATE() | When signature was made |
| DisclaimerAccepted | BIT | NOT NULL | Disclaimer checkbox status |
| IPAddress | NVARCHAR(50) | NULL | IP address of signatory |
| WorkstationName | NVARCHAR(100) | NULL | Computer name |

### WorkflowState Table
| Column | Type | Constraints | Description |
|--------|------|-------------|-------------|
| WorkflowID | INT | PRIMARY KEY, IDENTITY | Unique workflow ID |
| ReportID | INT | FOREIGN KEY (Reports.ReportID), NOT NULL, UNIQUE | Associated report |
| CurrentStep | INT | NOT NULL | Current step (1, 2, 3, or 4 for completed) |
| Step1_Completed | BIT | NOT NULL, DEFAULT 0 | Prepared By completed |
| Step1_CompletedDate | DATETIME | NULL | Step 1 completion date |
| Step2_Completed | BIT | NOT NULL, DEFAULT 0 | Branch Ack completed |
| Step2_CompletedDate | DATETIME | NULL | Step 2 completion date |
| Step3_Completed | BIT | NOT NULL, DEFAULT 0 | NISD Ack completed |
| Step3_CompletedDate | DATETIME | NULL | Step 3 completion date |
| WorkflowStatus | NVARCHAR(50) | NOT NULL | InProgress, Completed |
| CreatedDate | DATETIME | NOT NULL, DEFAULT GETDATE() | Workflow creation date |
| CompletedDate | DATETIME | NULL | Workflow completion date |

### Notifications Table
| Column | Type | Constraints | Description |
|--------|------|-------------|-------------|
| NotificationID | INT | PRIMARY KEY, IDENTITY | Unique notification ID |
| UserID | INT | FOREIGN KEY (Users.UserID), NOT NULL | Recipient user |
| ReportID | INT | FOREIGN KEY (Reports.ReportID), NOT NULL | Associated report |
| NotificationType | NVARCHAR(50) | NOT NULL | SignatureRequired, SignatureCompleted, ReportCompleted, Reminder |
| NotificationMessage | NVARCHAR(500) | NOT NULL | Notification text |
| IsRead | BIT | NOT NULL, DEFAULT 0 | Read status |
| CreatedDate | DATETIME | NOT NULL, DEFAULT GETDATE() | Notification creation |
| ReadDate | DATETIME | NULL | When notification was read |
| IsDismissed | BIT | NOT NULL, DEFAULT 0 | User dismissed flag |
| DismissedDate | DATETIME | NULL | When dismissed |

### SignatureAttempts Table
| Column | Type | Constraints | Description |
|--------|------|-------------|-------------|
| AttemptID | INT | PRIMARY KEY, IDENTITY | Unique attempt ID |
| ReportID | INT | FOREIGN KEY (Reports.ReportID), NOT NULL | Associated report |
| UserID | INT | FOREIGN KEY (Users.UserID), NOT NULL | User who attempted |
| SignatureType | NVARCHAR(50) | NOT NULL | Which signature step |
| AttemptDate | DATETIME | NOT NULL, DEFAULT GETDATE() | Attempt timestamp |
| IsSuccessful | BIT | NOT NULL | Success/failure flag |
| FailureReason | NVARCHAR(255) | NULL | Why attempt failed |

---

## Service Interfaces Exposed

This unit exposes the following operations to other units:

### Signature Operations
- **SubmitSignature**(reportId, userId, signatureType, signatoryName) → Success
  - Records signature and advances workflow
  - Validates workflow sequence
  - Creates notifications
  
- **GetSignatureStatus**(reportId) → SignatureStatusDetails
  - Returns status of all three signature steps
  
- **GetReportSignatures**(reportId) → SignatureList
  - Retrieves all signatures for a report
  
- **ValidateSignatureEligibility**(reportId, userId, signatureType) → Boolean
  - Checks if user can sign at current step

### Workflow Operations
- **InitializeWorkflow**(reportId) → Success
  - Creates workflow state for new report
  
- **GetWorkflowState**(reportId) → WorkflowState
  - Returns current workflow state
  
- **GetCurrentStep**(reportId) → StepNumber
  - Returns current step (1, 2, 3, or 4 if completed)
  
- **IsWorkflowComplete**(reportId) → Boolean
  - Checks if all signatures are complete

### Notification Operations
- **CreateNotification**(userId, reportId, notificationType, message) → NotificationId
  - Creates in-app notification
  
- **GetUserNotifications**(userId) → NotificationList
  - Retrieves all notifications for user
  
- **GetUnreadNotificationCount**(userId) → Count
  - Returns count of unread notifications
  
- **MarkNotificationAsRead**(notificationId) → Success
  - Marks notification as read
  
- **DismissNotification**(notificationId) → Success
  - Dismisses notification
  
- **GetPendingSignatureReports**(userId) → ReportList
  - Returns reports awaiting user's signature

---

## Dependencies on Other Units

### Depends On:
- **User Management & Authentication Unit**: 
  - User validation
  - User details for notifications
  
- **Report Lifecycle Management Unit**:
  - Report data validation
  - Report status updates
  - Report details for notifications

### Consumed By:
- **Report Lifecycle Management Unit**: Signature information for display
- **System Administration Unit**: Signature events for audit logs

---

## Business Rules

1. **Sequential Workflow**: Signatures must be completed in order (Step 1 → Step 2 → Step 3)
2. **No Skipping**: Cannot skip to Step 2 without completing Step 1
3. **No Bypassing**: Cannot skip to Step 3 without completing Steps 1 and 2
4. **Immutable Signatures**: Once submitted, signatures cannot be modified or deleted
5. **Disclaimer Required**: Users must accept disclaimer before signing
6. **Timestamp Capture**: System automatically captures timestamp (not user-entered)
7. **Name Entry**: Signatory must enter their name (not auto-populated)
8. **Report Validation**: Report must be complete before first signature
9. **Status Updates**: Report status automatically updates with each signature
10. **Notification Creation**: Notifications automatically created for next signatory
11. **Completion Notification**: All signatories notified when workflow completes
12. **Reminder Timing**: Reminders shown after 24 hours for pending signatures
13. **One Signature Per Step**: Each step can only be signed once
14. **User Assignment**: System determines who can sign based on role and report state
15. **Audit Trail**: All signature attempts logged for audit purposes

---

## UI Components (Desktop App)

### Screens/Forms:
1. **Signature Panel** (within Report View)
   - Three-step progress indicator
   - Step 1: Prepared By section
   - Step 2: Branch Acknowledgement section
   - Step 3: NISD Acknowledgement section
   - Visual indicators (completed, current, pending)
   - Signature button (enabled only for current step)

2. **Signature Dialog**
   - Disclaimer text (prominent display)
   - Disclaimer acceptance checkbox
   - Name entry field
   - Sign button (enabled after disclaimer accepted)
   - Cancel button
   - Confirmation message after signing

3. **Notification Panel**
   - Notification list
   - Unread count badge
   - Notification items with:
     - Report ID
     - Message
     - Timestamp
     - Link to report
   - Mark as read option
   - Dismiss option
   - Clear all option

4. **Dashboard Notification Widget**
   - "Pending My Signature" count
   - Quick list of reports awaiting signature
   - Direct links to reports

5. **Signature Status Display** (within Report View)
   - All three signatures shown
   - For each signature:
     - Signatory name
     - Timestamp
     - Status icon (completed/pending)
   - Overall workflow status

---

## Validation Rules

### Signature Name:
- Required
- 2-100 characters
- Letters and spaces only
- Cannot be empty or whitespace only

### Disclaimer Acceptance:
- Required
- Must be checked before signature submission

### Workflow Sequence:
- Step 1 must be completed before Step 2 is enabled
- Step 2 must be completed before Step 3 is enabled
- Cannot sign out of sequence

### Report Completeness:
- All required report fields must be filled before Step 1 signature
- All checklist items must be answered

### User Eligibility:
- User must be authenticated
- User must have appropriate role/permissions
- User cannot sign the same step twice

---

## Error Handling

### Common Errors:
- **Incomplete Report**: "Cannot sign. Please complete all required fields first"
- **Disclaimer Not Accepted**: "Please accept the disclaimer before signing"
- **Invalid Name**: "Please enter a valid name (letters and spaces only)"
- **Out of Sequence**: "This signature step is not yet available. Previous steps must be completed first"
- **Already Signed**: "This step has already been signed"
- **Permission Denied**: "You do not have permission to sign this report"
- **Report Not Found**: "Report not found"
- **Workflow Error**: "Unable to process signature. Please try again"
- **Concurrent Modification**: "This report has been modified. Please refresh and try again"

---

## Workflow State Transitions

### State Diagram:
```
Draft → Pending Branch Acknowledgement → Pending NISD Acknowledgement → Completed
  ↑              ↑                              ↑                           ↑
Step 1      Step 2                         Step 3                    Workflow
(Prepared By) (Branch Ack)                (NISD Ack)                Complete
```

### Transition Rules:
1. **Draft → Pending Branch Acknowledgement**
   - Trigger: Step 1 signature submitted
   - Validation: Report complete, disclaimer accepted
   - Action: Create notification for Branch Acknowledger

2. **Pending Branch Acknowledgement → Pending NISD Acknowledgement**
   - Trigger: Step 2 signature submitted
   - Validation: Step 1 complete, disclaimer accepted
   - Action: Create notification for NISD Acknowledger

3. **Pending NISD Acknowledgement → Completed**
   - Trigger: Step 3 signature submitted
   - Validation: Steps 1 and 2 complete, disclaimer accepted
   - Action: Create completion notifications for all signatories, lock report

---

## Notification Types

### 1. Signature Required
- **Trigger**: Report advances to next signature step
- **Recipient**: Next signatory in sequence
- **Message**: "Report [ID] from [Branch] requires your signature"
- **Action**: Link to report

### 2. Signature Completed
- **Trigger**: A signature is submitted
- **Recipients**: Report creator and previous signatories
- **Message**: "[Signatory Name] has signed report [ID] as [Role]"
- **Action**: Link to report

### 3. Report Completed
- **Trigger**: Final signature (Step 3) is submitted
- **Recipients**: All three signatories
- **Message**: "Report [ID] from [Branch] has been completed with all signatures"
- **Action**: Link to report

### 4. Reminder
- **Trigger**: 24 hours after signature becomes available
- **Recipient**: User with pending signature
- **Message**: "Reminder: Report [ID] from [Branch] is awaiting your signature"
- **Action**: Link to report
- **Frequency**: Daily until signed or dismissed

---

## Notes

- This unit should be built after User Management and Report Lifecycle units
- Workflow enforcement is critical - must be thoroughly tested
- Signatures are legally binding - ensure immutability
- Timestamps should use server time (not client time) to prevent tampering
- Consider implementing signature verification/validation
- Notification system is simplified (in-app only, no email)
- Reminders should not be intrusive but should be visible
- Audit trail is important for compliance and dispute resolution
- Consider implementing signature delegation for future enhancement
- Workflow state should be transactional to prevent inconsistencies
- Test concurrent signature attempts thoroughly
