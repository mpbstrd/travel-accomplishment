# Travel Accomplishment Report System - User Stories

## Document Information
**Project**: Travel Accomplishment Report Repository System  
**Organization**: Development Bank of the Philippines - Network Infrastructure Services Department  
**Version**: 1.0  
**Date**: December 5, 2025

## System Overview
A digital repository system to capture, store, and manage Travel Accomplishment Reports based on the Branch Travel Checklist form. The system enforces a sequential signature workflow with three approval stages and provides role-based access control for NISD personnel.

## User Roles

### Admin
- Full system access
- Account management (create, edit, delete user accounts)
- Report editing capabilities at any stage
- System configuration and maintenance

### User (NISD Personnel)
- Create and submit travel accomplishment reports
- Input data and upload images
- Sign reports as designated signatory
- View own reports and reports requiring their signature

### Signatory Roles
1. **Prepared By**: Initial report creator/preparer
2. **Branch Acknowledgement**: Branch representative approval
3. **NISD Acknowledgement**: Final NISD approval

---

## Epic 1: User Account Management

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

### US-1.4: User Login
**As a** User  
**I want to** log into the system securely  
**So that** I can access my reports and perform my duties

**Acceptance Criteria:**
- Login page with username and password fields
- Authentication validates credentials against database
- Failed login attempts are tracked (max 5 attempts before temporary lockout)
- Successful login redirects to dashboard
- Session timeout after 30 minutes of inactivity
- "Remember Me" option for trusted devices
- Password visibility toggle available

---

## Epic 2: Report Creation and Data Entry

### US-2.1: Create New Travel Accomplishment Report
**As a** User  
**I want to** create a new travel accomplishment report  
**So that** I can document branch visit activities

**Acceptance Criteria:**
- User can access "Create New Report" button from dashboard
- System generates unique report ID automatically
- Report creation timestamp recorded
- Creator's name auto-populated as "Prepared By"
- Report status set to "Draft" upon creation
- User can save draft and continue later
- All form sections are accessible and organized logically

### US-2.2: Enter Basic Travel Information
**As a** User  
**I want to** enter basic travel details  
**So that** the report captures essential visit information

**Acceptance Criteria:**
- Field: Branch/BLU/LC (text input, required, max 100 characters)
- Field: Date of Travel (date picker, required, cannot be future date)
- Field: Time Started (time picker, required, format HH:MM)
- Field: Time Ended (time picker, required, must be after Time Started)
- Field: Purpose of Travel (text area, required, max 500 characters)
- All fields have clear labels and validation messages
- Data is auto-saved every 2 minutes
- Manual save button available

### US-2.3: Enter Network Equipment Details
**As a** User  
**I want to** record network equipment information  
**So that** the report documents the branch's technical infrastructure

**Acceptance Criteria:**
- **Telco 1 Section:**
  - Telco 1 name (text input, optional, max 50 characters)
  - Router Brand (text input, optional, max 50 characters)
  - Modem Brand (text input, optional, max 50 characters)
- **Telco 2 Section:**
  - Telco 2 name (text input, optional, max 50 characters)
  - Router Brand (text input, optional, max 50 characters)
  - Modem Brand (text input, optional, max 50 characters)
- **Switch 1 Section:**
  - Brand (text input, optional, max 50 characters)
  - Model (text input, optional, max 50 characters)
  - Ports (number input, optional, range 1-999)
  - PN/Part Number (text input, optional, max 50 characters)
- **Switch 2 Section:**
  - Brand (text input, optional, max 50 characters)
  - Model (text input, optional, max 50 characters)
  - Ports (number input, optional, range 1-999)
  - PN/Part Number (text input, optional, max 50 characters)
- **SD-WAN Section:**
  - Lights Status (text input, optional, max 200 characters)
- All fields properly labeled and grouped
- Clear section headers for organization

### US-2.4: Complete Activity Checklist
**As a** User  
**I want to** complete the activity checklist items  
**So that** I can document all required verification steps

**Acceptance Criteria:**
- **Checklist Item 1: Capture image of data rack before the activity**
  - Choice: Yes/No (radio buttons, required)
  - Comment/Remarks field (text area, optional, max 500 characters)
- **Checklist Item 2: Check physical connection of network equipment**
  - Sub-items:
    - UPS available (Yes/No)
    - If Yes, all power cord connected to UPS (Yes/No)
    - If No UPS, all power cord connected to safe power outlet (Yes/No)
  - Choice: Yes/No (radio buttons, required)
  - Comment/Remarks field (text area, optional, max 500 characters)
- **Checklist Item 3: Check all equipment working before activity**
  - Equipment list: Modem, Router, Switch, SD-WAN
  - Choice: Yes/No (radio buttons, required)
  - Comment/Remarks field (text area, optional, max 500 characters)
- **Checklist Item 4: Specify the activity**
  - Choices: Re-grooming/cabling, Telco Related, Others (specify)
  - Radio buttons with text input for "Others"
  - Required field
  - Comment/Remarks field (text area, optional, max 500 characters)
- **Checklist Item 5: Verify equipment working after activity**
  - Equipment list: Modem, Router, Switch, SD-WAN
  - Choice: Yes/No (radio buttons, required)
  - Comment/Remarks field (text area, optional, max 500 characters)
- **Checklist Item 6: Cables properly tagged/labeled**
  - Choice: Yes/No (radio buttons, required)
  - Comment/Remarks field (text area, optional, max 500 characters)
- **Checklist Item 7: Capture image of data rack after activity**
  - Choice: Yes/No (radio buttons, required)
  - Comment/Remarks field (text area, optional, max 500 characters)
- All checklist items must be answered to submit report
- Visual indication of incomplete items

### US-2.5: Upload Images
**As a** User  
**I want to** upload images of the data rack and equipment  
**So that** I can provide visual documentation of the site visit

**Acceptance Criteria:**
- Image upload interface with drag-and-drop support
- Supported formats: JPG, JPEG, PNG, GIF, BMP, WEBP
- Maximum file size: 15MB per image
- Maximum 10 images per report
- Image preview after upload
- Ability to delete uploaded images before submission
- File name and size displayed for each image
- Upload progress indicator
- Error messages for invalid files or size limits exceeded
- Images are associated with specific checklist items when applicable
- Thumbnail view of all uploaded images

---

## Epic 3: Sequential Signature Workflow

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
  - Notification sent to Branch Acknowledger
- Signature details displayed in report (Name and Timestamp)
- "Prepared By" section becomes read-only after signature
- Report creator can no longer edit report data after signing (only Admin can edit)

### US-3.3: Branch Acknowledgement Signature (Step 2)
**As a** Branch representative  
**I want to** sign as "Branch Acknowledgement"  
**So that** I can confirm the branch's review and acceptance of the report

**Acceptance Criteria:**
- "Branch Acknowledgement" signature section only enabled after "Prepared By" is completed
- System enforces sequential workflow (cannot skip to Step 3)
- Branch Acknowledger receives notification when report is ready for their signature
- Signature disclaimer displayed before entry
- Upon signature:
  - Name entered by Branch Acknowledger is recorded
  - Timestamp automatically captured (format: YYYY-MM-DD HH:MM:SS)
  - Report status changes to "Pending NISD Acknowledgement"
  - Notification sent to NISD Acknowledger
- Signature details displayed in report (Name and Timestamp)
- "Branch Acknowledgement" section becomes read-only after signature
- Previous signatures remain visible and unmodifiable

### US-3.4: NISD Acknowledgement Signature (Step 3)
**As a** NISD representative  
**I want to** sign as "NISD Acknowledgement"  
**So that** I can provide final approval and complete the report

**Acceptance Criteria:**
- "NISD Acknowledgement" signature section only enabled after "Branch Acknowledgement" is completed
- System enforces sequential workflow (requires Steps 1 and 2)
- NISD Acknowledger receives notification when report is ready for their signature
- Signature disclaimer displayed before entry
- Upon signature:
  - Name entered by NISD Acknowledger is recorded
  - Timestamp automatically captured (format: YYYY-MM-DD HH:MM:SS)
  - Report status changes to "Completed"
  - Report is finalized and locked
  - Completion notification sent to all signatories
- Signature details displayed in report (Name and Timestamp)
- All three signatures visible with timestamps
- Report marked as finalized in the system

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

## Epic 4: Notifications

### US-4.1: Signature Required Notifications
**As a** User  
**I want to** receive notifications when a report requires my signature  
**So that** I can promptly review and sign reports

**Acceptance Criteria:**
- Notification sent when report reaches user's signature step
- Notification includes:
  - Report ID
  - Branch/BLU/LC name
  - Date of travel
  - Link to report
  - Deadline (if applicable)
- Notification delivered via:
  - In-app notification (badge/counter)
  - Email notification
- Notification marked as read when user views the report
- Unread notification counter displayed on dashboard

### US-4.2: Signature Completion Notifications
**As a** User  
**I want to** receive notifications when signatures are completed  
**So that** I can track report progress

**Acceptance Criteria:**
- Notification sent to report creator when each signature is completed
- Notification sent to all previous signatories when subsequent signatures are completed
- Notification includes:
  - Report ID
  - Signatory name and role
  - Signature timestamp
  - Current report status
  - Link to report
- Final completion notification sent to all signatories when report is fully approved
- Notification history accessible in user profile

### US-4.3: Incomplete Report Reminders
**As a** User  
**I want to** receive reminders about incomplete reports  
**So that** I don't forget to complete pending tasks

**Acceptance Criteria:**
- Reminder sent for draft reports not submitted after 48 hours
- Reminder sent for reports awaiting signature after 24 hours
- Reminder frequency: Daily until action taken
- Reminder includes report details and direct link
- User can snooze reminders (options: 1 day, 3 days, 1 week)
- User can disable reminders for specific reports

---

## Epic 5: Report Management and Viewing

### US-5.1: View Report Dashboard
**As a** User  
**I want to** see a dashboard of all my reports  
**So that** I can quickly access and manage them

**Acceptance Criteria:**
- Dashboard displays reports in categorized tabs:
  - My Drafts
  - Pending My Signature
  - Completed Reports
  - All Reports (Admin only)
- Each report card shows:
  - Report ID
  - Branch/BLU/LC
  - Date of Travel
  - Current Status
  - Last Updated
  - Quick action buttons
- Report count displayed for each category
- Default view shows most recent reports first
- Pagination for large result sets (20 reports per page)

### US-5.2: Search and Filter Reports
**As a** User  
**I want to** search and filter reports  
**So that** I can quickly find specific reports

**Acceptance Criteria:**
- Search bar with real-time search capability
- Searchable fields:
  - Report ID
  - Branch/BLU/LC
  - Purpose of Travel
  - Signatory names
- Filter options:
  - Date Range (Date of Travel)
  - Report Status (Draft, Pending, Completed)
  - Branch/BLU/LC (dropdown)
  - Telco 1/Telco 2 (dropdown)
  - Router Brand (dropdown)
  - Activity Type (Re-grooming/cabling, Telco Related, Others)
  - Created By (user dropdown)
  - Signature Status (Unsigned, Partially Signed, Fully Signed)
- Multiple filters can be applied simultaneously
- Filter selections persist during session
- "Clear All Filters" button available
- Export filtered results option (CSV/PDF)
- Filter results count displayed

### US-5.3: View Report Details
**As a** User  
**I want to** view complete report details  
**So that** I can review all information and signatures

**Acceptance Criteria:**
- Report displayed in organized, readable format
- All sections clearly labeled and grouped:
  - Basic Information
  - Network Equipment Details
  - Activity Checklist
  - Images
  - Signatures
- Read-only view for completed reports
- Images displayed in gallery view with zoom capability
- Signature section shows:
  - Signatory name
  - Signature timestamp
  - Status indicator (Completed/Pending)
- Print-friendly view available
- Export to PDF option
- Report metadata displayed (Created Date, Last Modified, Report ID)
- Navigation buttons (Previous/Next report)

### US-5.4: Admin Edit Report
**As an** Admin  
**I want to** edit any report at any stage  
**So that** I can correct errors or update information

**Acceptance Criteria:**
- Admin has "Edit" button on all reports regardless of status
- Edit mode allows modification of all data fields except:
  - Report ID
  - Creation timestamp
  - Existing signatures and timestamps
- Admin can add/remove images
- Admin can update checklist responses
- Changes are tracked in audit log with:
  - Admin username
  - Timestamp of change
  - Fields modified
  - Previous and new values
- "Save Changes" button with confirmation dialog
- Report status remains unchanged after edit
- Notification sent to report creator about admin edits
- Edit history viewable by Admin

### US-5.5: Delete Report (Admin Only)
**As an** Admin  
**I want to** delete reports  
**So that** I can remove erroneous or duplicate entries

**Acceptance Criteria:**
- Delete button available only to Admin users
- Confirmation dialog with warning message before deletion
- Soft delete implementation (report archived, not permanently removed)
- Deleted reports moved to "Archived Reports" section
- Archived reports can be restored by Admin within 30 days
- After 30 days, archived reports permanently deleted
- Deletion logged in audit trail with:
  - Admin username
  - Deletion timestamp
  - Reason for deletion (optional text field)
- Report creator notified of deletion

---

## Epic 6: Reporting and Analytics

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

### US-6.2: Export Reports
**As a** User  
**I want to** export reports in various formats  
**So that** I can share or archive them externally

**Acceptance Criteria:**
- Export options available:
  - Single report to PDF
  - Multiple reports to ZIP (containing individual PDFs)
  - Report list to CSV/Excel
- PDF export includes:
  - All report data formatted professionally
  - All uploaded images
  - All signatures with timestamps
  - Report metadata (ID, creation date, status)
- CSV export includes all filterable fields
- Export progress indicator for large exports
- Download link provided upon completion
- Export history tracked (last 10 exports per user)

---

## Epic 7: System Administration

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
  - Date range
  - User
  - Action type
  - Entity ID
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
  - Notification settings (enable/disable email)
  - Reminder frequency
  - Archive retention period
- Changes take effect immediately
- Configuration changes logged in audit trail
- Default values available for reset
- Validation for configuration values
- Confirmation required for critical changes

### US-7.3: Backup and Restore
**As an** Admin  
**I want to** backup and restore system data  
**So that** I can protect against data loss

**Acceptance Criteria:**
- Manual backup trigger available
- Automated daily backups scheduled
- Backup includes:
  - All report data
  - User accounts
  - Images and attachments
  - System configuration
- Backup stored securely with encryption
- Restore functionality with point-in-time selection
- Backup history showing:
  - Backup date/time
  - Backup size
  - Backup status
- Test restore capability
- Backup retention: 30 days

---

## Epic 8: Security and Access Control

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
  - API level (reject unauthorized requests)
  - Database level (row-level security where applicable)
- Unauthorized access attempts logged
- Clear error messages for permission denials

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
- Password reset via email for forgotten passwords
- Password reset link expires after 24 hours
- Account lockout after 5 failed login attempts (15-minute lockout)

### US-8.3: Session Management
**As the** System  
**I want to** manage user sessions securely  
**So that** unauthorized access is prevented

**Acceptance Criteria:**
- Session timeout after 30 minutes of inactivity
- Warning displayed 2 minutes before timeout
- Option to extend session before timeout
- Secure session tokens (JWT or similar)
- Session invalidated on logout
- Only one active session per user (optional: configurable)
- Session data encrypted
- Session hijacking protection (IP validation, user agent validation)
- "Remember Me" creates extended session (7 days) with secure cookie

### US-8.4: Data Encryption
**As the** System  
**I want to** encrypt sensitive data  
**So that** information is protected from unauthorized access

**Acceptance Criteria:**
- Data encrypted at rest:
  - User passwords (hashed)
  - Session tokens
  - Uploaded images
  - Sensitive report fields
- Data encrypted in transit (HTTPS/TLS)
- Encryption keys managed securely
- Regular key rotation schedule
- Compliance with data protection standards
- Encrypted backups

---

## Epic 9: User Experience and Interface

### US-9.1: Responsive Design
**As a** User  
**I want to** access the system from various devices  
**So that** I can work from desktop, tablet, or mobile

**Acceptance Criteria:**
- Responsive layout adapts to screen sizes:
  - Desktop (1920x1080 and above)
  - Laptop (1366x768 and above)
  - Tablet (768x1024)
  - Mobile (375x667 and above)
- Touch-friendly interface for mobile/tablet
- Optimized image loading for mobile networks
- Navigation menu adapts to screen size (hamburger menu on mobile)
- Forms remain usable on small screens
- Tables scroll horizontally on small screens
- All features accessible on all devices

### US-9.2: User-Friendly Forms
**As a** User  
**I want to** complete forms easily  
**So that** data entry is efficient and error-free

**Acceptance Criteria:**
- Clear field labels and placeholders
- Inline validation with helpful error messages
- Required fields marked with asterisk (*)
- Field-level help text/tooltips where needed
- Auto-save functionality (every 2 minutes)
- Progress indicator for multi-section forms
- Tab navigation between fields
- Date/time pickers for temporal fields
- Dropdown suggestions for common values
- Character count for text areas with limits
- Undo/redo capability for text fields

### US-9.3: Accessibility Compliance
**As a** User with disabilities  
**I want to** use the system with assistive technologies  
**So that** I can perform my duties independently

**Acceptance Criteria:**
- WCAG 2.1 Level AA compliance
- Keyboard navigation for all features
- Screen reader compatibility
- Sufficient color contrast (minimum 4.5:1)
- Alt text for all images
- ARIA labels for interactive elements
- Focus indicators visible
- No time-based content without controls
- Resizable text without loss of functionality
- Error messages announced to screen readers

### US-9.4: Performance Optimization
**As a** User  
**I want to** experience fast page loads and interactions  
**So that** I can work efficiently

**Acceptance Criteria:**
- Page load time under 3 seconds on standard connection
- Image optimization (compression, lazy loading)
- Minimal JavaScript bundle size
- Database query optimization
- Caching strategy for static content
- Pagination for large data sets
- Asynchronous operations for heavy tasks
- Loading indicators for operations over 1 second
- Smooth animations and transitions (60fps)
- Efficient search with debouncing

---

## Epic 10: Data Validation and Error Handling

### US-10.1: Form Validation
**As a** User  
**I want to** receive clear validation feedback  
**So that** I can correct errors before submission

**Acceptance Criteria:**
- Real-time validation for:
  - Required fields
  - Field formats (email, date, time)
  - Field lengths (min/max characters)
  - Numeric ranges
  - File types and sizes
- Validation messages displayed:
  - Inline below field (for field-specific errors)
  - Summary at top of form (for form-level errors)
- Error messages are clear and actionable
- Fields with errors highlighted (red border)
- Validation occurs on:
  - Field blur (lose focus)
  - Form submission
- Valid fields show success indicator (green checkmark)
- Form submission disabled until all validations pass

### US-10.2: Error Handling
**As a** User  
**I want to** receive helpful error messages  
**So that** I understand what went wrong and how to fix it

**Acceptance Criteria:**
- User-friendly error messages (avoid technical jargon)
- Error messages include:
  - What went wrong
  - Why it happened (if relevant)
  - How to fix it
- Different error types handled:
  - Validation errors (field-level)
  - Network errors (connection issues)
  - Server errors (500, 503)
  - Authorization errors (401, 403)
  - Not found errors (404)
- Error logging for debugging (admin view)
- Retry mechanism for transient errors
- Graceful degradation when features unavailable
- Contact support option for critical errors

### US-10.3: Data Integrity
**As the** System  
**I want to** maintain data integrity  
**So that** information remains accurate and consistent

**Acceptance Criteria:**
- Database constraints enforced:
  - Foreign key relationships
  - Unique constraints
  - Not null constraints
  - Check constraints
- Transaction management for multi-step operations
- Rollback on failure
- Optimistic locking for concurrent edits
- Data validation at multiple layers (UI, API, Database)
- Referential integrity maintained
- Orphaned records prevented
- Data consistency checks in background jobs

---

## Assumptions and Constraints

### Assumptions
1. All users have valid email addresses for notifications
2. Users have access to modern web browsers (Chrome, Firefox, Edge, Safari - latest 2 versions)
3. Network connectivity is generally stable for NISD personnel
4. Users have basic computer literacy
5. Branch names and equipment brands are not standardized (free text entry)
6. Images are primarily photos taken during site visits
7. System will be hosted on secure internal network or cloud infrastructure
8. Users will access system during business hours primarily
9. Report volume estimated at 50-200 reports per month
10. Signature workflow is linear (no parallel approvals or rejections)

### Constraints
1. Maximum 10 images per report (15MB each)
2. Session timeout: 30 minutes of inactivity
3. Password requirements: minimum 8 characters, alphanumeric
4. Account lockout: 5 failed login attempts
5. Backup retention: 30 days
6. Audit log retention: minimum 1 year
7. Archive retention: 30 days before permanent deletion
8. Only NISD personnel have system access
9. Sequential signature workflow cannot be bypassed
10. Admin is the only role that can edit completed reports
11. Signatures cannot be modified or deleted once submitted
12. Report cannot be finalized without all three signatures

### Technical Constraints
1. System must support Windows operating system
2. Must be accessible via web browser (no native app required)
3. Must support HTTPS/TLS encryption
4. Must comply with organizational security policies
5. Must integrate with existing email system for notifications
6. Database must support concurrent users (minimum 50 simultaneous users)
7. File storage must accommodate growth (estimated 10GB per year)

---

## Data Dictionary

### Report Fields
| Field Name | Type | Required | Max Length | Validation Rules |
|------------|------|----------|------------|------------------|
| Report ID | Auto-generated | Yes | 20 | Unique, system-generated |
| Branch/BLU/LC | Text | Yes | 100 | Alphanumeric |
| Date of Travel | Date | Yes | - | Cannot be future date |
| Time Started | Time | Yes | - | Format HH:MM |
| Time Ended | Time | Yes | - | Must be after Time Started |
| Purpose of Travel | Text Area | Yes | 500 | Alphanumeric with punctuation |
| Telco 1 | Text | No | 50 | Alphanumeric |
| Telco 1 Router Brand | Text | No | 50 | Alphanumeric |
| Telco 1 Modem Brand | Text | No | 50 | Alphanumeric |
| Telco 2 | Text | No | 50 | Alphanumeric |
| Telco 2 Router Brand | Text | No | 50 | Alphanumeric |
| Telco 2 Modem Brand | Text | No | 50 | Alphanumeric |
| Switch 1 Brand | Text | No | 50 | Alphanumeric |
| Switch 1 Model | Text | No | 50 | Alphanumeric |
| Switch 1 Ports | Number | No | - | 1-999 |
| Switch 1 PN | Text | No | 50 | Alphanumeric |
| Switch 2 Brand | Text | No | 50 | Alphanumeric |
| Switch 2 Model | Text | No | 50 | Alphanumeric |
| Switch 2 Ports | Number | No | - | 1-999 |
| Switch 2 PN | Text | No | 50 | Alphanumeric |
| SD-WAN Lights Status | Text | No | 200 | Alphanumeric |
| Checklist Item 1-7 | Yes/No | Yes | - | Boolean |
| Checklist Comments 1-7 | Text Area | No | 500 | Alphanumeric with punctuation |
| Activity Type | Selection | Yes | - | Re-grooming/cabling, Telco Related, Others |
| Activity Type Other | Text | Conditional | 100 | Required if "Others" selected |
| Images | File Upload | No | 15MB each | Max 10 images, JPG/PNG/GIF/BMP/WEBP |
| Prepared By Name | Text | Yes | 100 | Alphanumeric |
| Prepared By Timestamp | DateTime | Auto | - | System-generated |
| Branch Ack Name | Text | Yes | 100 | Alphanumeric |
| Branch Ack Timestamp | DateTime | Auto | - | System-generated |
| NISD Ack Name | Text | Yes | 100 | Alphanumeric |
| NISD Ack Timestamp | DateTime | Auto | - | System-generated |
| Report Status | Enum | Auto | - | Draft, Pending Branch, Pending NISD, Completed |
| Created By | User ID | Auto | - | System-generated |
| Created Date | DateTime | Auto | - | System-generated |
| Last Modified | DateTime | Auto | - | System-generated |

### User Account Fields
| Field Name | Type | Required | Max Length | Validation Rules |
|------------|------|----------|------------|------------------|
| User ID | Auto-generated | Yes | 20 | Unique, system-generated |
| Username | Text | Yes | 50 | Unique, alphanumeric, no spaces |
| Full Name | Text | Yes | 100 | Alphanumeric with spaces |
| Email | Email | Yes | 100 | Valid email format, unique |
| Password | Password | Yes | - | Min 8 chars, alphanumeric |
| Role | Enum | Yes | - | Admin, User |
| Account Status | Enum | Yes | - | Active, Inactive |
| Created Date | DateTime | Auto | - | System-generated |
| Last Login | DateTime | Auto | - | System-generated |

---

## Success Metrics

### User Adoption
- 90% of NISD personnel onboarded within first month
- 80% of reports submitted digitally within first quarter
- Average user satisfaction score of 4/5 or higher

### System Performance
- 99% system uptime
- Average page load time under 3 seconds
- Zero data loss incidents
- 95% of reports completed within 48 hours of creation

### Workflow Efficiency
- Average time from report creation to final signature: under 72 hours
- 90% of signatures completed within 24 hours of notification
- Reduction in paper-based reports by 95%

### Data Quality
- 95% of reports have complete data (all required fields)
- Less than 5% of reports require admin correction
- 100% of reports have all three signatures before finalization

---

## Future Enhancements (Out of Scope for v1.0)

1. **Mobile App**: Native iOS/Android applications
2. **Offline Mode**: Ability to create reports offline and sync later
3. **Advanced Analytics**: Predictive analytics, trend analysis, custom dashboards
4. **Integration**: Integration with other DBP systems (HR, Asset Management)
5. **Workflow Customization**: Configurable approval workflows per branch
6. **Digital Signature**: PKI-based digital signatures instead of name entry
7. **Document Templates**: Customizable report templates
8. **Bulk Operations**: Bulk report creation, bulk signature approval
9. **API Access**: RESTful API for third-party integrations
10. **Multi-language Support**: Support for multiple languages
11. **Advanced Search**: Full-text search, saved searches, search templates
12. **Collaboration**: Comments, annotations, internal messaging
13. **Version Control**: Track all changes to reports with version history
14. **Approval Delegation**: Ability to delegate signature authority
15. **Scheduled Reports**: Automated report generation and distribution

---

## Traceability Matrix

| Form Element | User Story | Epic |
|--------------|------------|------|
| Branch/BLU/LC | US-2.2 | Epic 2 |
| Date of Travel | US-2.2 | Epic 2 |
| Time Started/Ended | US-2.2 | Epic 2 |
| Purpose of Travel | US-2.2 | Epic 2 |
| Telco 1/2 Details | US-2.3 | Epic 2 |
| Router/Modem Brands | US-2.3 | Epic 2 |
| Switch Details | US-2.3 | Epic 2 |
| SD-WAN Status | US-2.3 | Epic 2 |
| Checklist Items 1-7 | US-2.4 | Epic 2 |
| Comments/Remarks | US-2.4 | Epic 2 |
| Image Upload | US-2.5 | Epic 2 |
| Prepared By Signature | US-3.2 | Epic 3 |
| Branch Acknowledgement | US-3.3 | Epic 3 |
| NISD Acknowledgement | US-3.4 | Epic 3 |
| Signature Disclaimer | US-3.1 | Epic 3 |
| Sequential Workflow | US-3.5 | Epic 3 |

---

## Document Approval

**Prepared By**: AI Product Manager  
**Date**: December 5, 2025  
**Version**: 1.0  

**Review Status**: Pending Stakeholder Approval

---

*End of User Stories Document*
