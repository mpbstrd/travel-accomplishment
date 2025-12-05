# Unit 2: Report Lifecycle Management

## Unit Overview

**Business Capability**: Manage the complete lifecycle of Travel Accomplishment Reports from creation to archival.

**Responsibility**: This unit handles all aspects of report data management including creation, editing, viewing, searching, filtering, file storage, and export functionality.

**Scope**:
- Report creation and data entry
- Network equipment details capture
- Activity checklist management
- Image upload and storage
- Report viewing and navigation
- Search and filter functionality
- Report editing (Admin only)
- Report deletion and archival (Admin only)
- Basic export functionality (PDF/CSV)

---

## User Stories

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

---

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

---

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

---

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

---

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

---

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

---

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

---

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

---

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

### US-10.1: Form Validation
**As a** User  
**I want to** receive clear validation feedback  
**So that** I can correct errors before submission

**Acceptance Criteria:**
- Real-time validation for:
  - Required fields
  - Field formats (date, time)
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

---

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
- Data validation at multiple layers (UI, Business Logic, Database)
- Referential integrity maintained
- Orphaned records prevented
- Data consistency checks in background jobs

---

## Database Tables Owned by This Unit

### Reports Table
| Column | Type | Constraints | Description |
|--------|------|-------------|-------------|
| ReportID | INT | PRIMARY KEY, IDENTITY | Unique report identifier |
| ReportNumber | NVARCHAR(50) | UNIQUE, NOT NULL | Human-readable report number |
| BranchName | NVARCHAR(100) | NOT NULL | Branch/BLU/LC name |
| DateOfTravel | DATE | NOT NULL | Travel date |
| TimeStarted | TIME | NOT NULL | Start time |
| TimeEnded | TIME | NOT NULL | End time |
| PurposeOfTravel | NVARCHAR(500) | NOT NULL | Purpose description |
| ReportStatus | NVARCHAR(50) | NOT NULL | Draft, Pending Branch, Pending NISD, Completed |
| CreatedBy | INT | FOREIGN KEY (Users.UserID), NOT NULL | Report creator |
| CreatedDate | DATETIME | NOT NULL, DEFAULT GETDATE() | Creation timestamp |
| LastModifiedBy | INT | FOREIGN KEY (Users.UserID) | Last modifier |
| LastModifiedDate | DATETIME | NULL | Last modification timestamp |
| IsDeleted | BIT | NOT NULL, DEFAULT 0 | Soft delete flag |
| DeletedBy | INT | FOREIGN KEY (Users.UserID) | Admin who deleted |
| DeletedDate | DATETIME | NULL | Deletion timestamp |
| DeletedReason | NVARCHAR(500) | NULL | Reason for deletion |

### NetworkEquipment Table
| Column | Type | Constraints | Description |
|--------|------|-------------|-------------|
| EquipmentID | INT | PRIMARY KEY, IDENTITY | Unique equipment record ID |
| ReportID | INT | FOREIGN KEY (Reports.ReportID), NOT NULL | Associated report |
| Telco1Name | NVARCHAR(50) | NULL | Telco 1 provider |
| Telco1RouterBrand | NVARCHAR(50) | NULL | Telco 1 router brand |
| Telco1ModemBrand | NVARCHAR(50) | NULL | Telco 1 modem brand |
| Telco2Name | NVARCHAR(50) | NULL | Telco 2 provider |
| Telco2RouterBrand | NVARCHAR(50) | NULL | Telco 2 router brand |
| Telco2ModemBrand | NVARCHAR(50) | NULL | Telco 2 modem brand |
| Switch1Brand | NVARCHAR(50) | NULL | Switch 1 brand |
| Switch1Model | NVARCHAR(50) | NULL | Switch 1 model |
| Switch1Ports | INT | NULL | Switch 1 port count |
| Switch1PN | NVARCHAR(50) | NULL | Switch 1 part number |
| Switch2Brand | NVARCHAR(50) | NULL | Switch 2 brand |
| Switch2Model | NVARCHAR(50) | NULL | Switch 2 model |
| Switch2Ports | INT | NULL | Switch 2 port count |
| Switch2PN | NVARCHAR(50) | NULL | Switch 2 part number |
| SDWANLightsStatus | NVARCHAR(200) | NULL | SD-WAN status |

### ActivityChecklist Table
| Column | Type | Constraints | Description |
|--------|------|-------------|-------------|
| ChecklistID | INT | PRIMARY KEY, IDENTITY | Unique checklist ID |
| ReportID | INT | FOREIGN KEY (Reports.ReportID), NOT NULL | Associated report |
| Item1_ImageBefore | BIT | NOT NULL | Captured image before |
| Item1_Comment | NVARCHAR(500) | NULL | Comment for item 1 |
| Item2_PhysicalConnection | BIT | NOT NULL | Physical connection check |
| Item2_Comment | NVARCHAR(500) | NULL | Comment for item 2 |
| Item3_EquipmentWorking | BIT | NOT NULL | Equipment working before |
| Item3_Comment | NVARCHAR(500) | NULL | Comment for item 3 |
| Item4_ActivityType | NVARCHAR(50) | NOT NULL | Activity type |
| Item4_ActivityOther | NVARCHAR(100) | NULL | Other activity description |
| Item4_Comment | NVARCHAR(500) | NULL | Comment for item 4 |
| Item5_EquipmentVerified | BIT | NOT NULL | Equipment working after |
| Item5_Comment | NVARCHAR(500) | NULL | Comment for item 5 |
| Item6_CablesTagged | BIT | NOT NULL | Cables tagged/labeled |
| Item6_Comment | NVARCHAR(500) | NULL | Comment for item 6 |
| Item7_ImageAfter | BIT | NOT NULL | Captured image after |
| Item7_Comment | NVARCHAR(500) | NULL | Comment for item 7 |

### ReportImages Table
| Column | Type | Constraints | Description |
|--------|------|-------------|-------------|
| ImageID | INT | PRIMARY KEY, IDENTITY | Unique image ID |
| ReportID | INT | FOREIGN KEY (Reports.ReportID), NOT NULL | Associated report |
| FileName | NVARCHAR(255) | NOT NULL | Original file name |
| FileSize | BIGINT | NOT NULL | File size in bytes |
| FileType | NVARCHAR(50) | NOT NULL | MIME type |
| FilePath | NVARCHAR(500) | NOT NULL | Storage path |
| UploadedBy | INT | FOREIGN KEY (Users.UserID), NOT NULL | User who uploaded |
| UploadedDate | DATETIME | NOT NULL, DEFAULT GETDATE() | Upload timestamp |
| ChecklistItem | INT | NULL | Associated checklist item (1-7) |
| IsDeleted | BIT | NOT NULL, DEFAULT 0 | Soft delete flag |

### ReportEditHistory Table
| Column | Type | Constraints | Description |
|--------|------|-------------|-------------|
| HistoryID | INT | PRIMARY KEY, IDENTITY | Unique history ID |
| ReportID | INT | FOREIGN KEY (Reports.ReportID), NOT NULL | Associated report |
| EditedBy | INT | FOREIGN KEY (Users.UserID), NOT NULL | Admin who edited |
| EditedDate | DATETIME | NOT NULL, DEFAULT GETDATE() | Edit timestamp |
| FieldName | NVARCHAR(100) | NOT NULL | Field that was modified |
| OldValue | NVARCHAR(MAX) | NULL | Previous value |
| NewValue | NVARCHAR(MAX) | NULL | New value |

---

## Service Interfaces Exposed

This unit exposes the following operations to other units:

### Report Management Operations
- **CreateReport**(userId, reportData) → ReportId
  - Creates new report in draft status
  
- **UpdateReport**(reportId, reportData) → Success
  - Updates report data (draft only, or admin)
  
- **GetReportById**(reportId) → ReportDetails
  - Retrieves complete report information
  
- **GetReportsByUser**(userId) → ReportList
  - Gets all reports created by user
  
- **SearchReports**(searchCriteria) → ReportList
  - Searches reports based on criteria
  
- **DeleteReport**(reportId, userId, reason) → Success
  - Soft deletes report (admin only)
  
- **RestoreReport**(reportId) → Success
  - Restores archived report (admin only)

### Report Status Operations
- **GetReportStatus**(reportId) → Status
  - Returns current report status
  
- **UpdateReportStatus**(reportId, newStatus) → Success
  - Updates report status (called by Signature Workflow)
  
- **ValidateReportComplete**(reportId) → Boolean
  - Checks if all required fields are filled

### Image Management Operations
- **UploadImage**(reportId, imageFile, checklistItem) → ImageId
  - Uploads and stores image file
  
- **GetReportImages**(reportId) → ImageList
  - Retrieves all images for report
  
- **DeleteImage**(imageId) → Success
  - Removes image from report

### Export Operations
- **ExportReportToPDF**(reportId) → PDFFile
  - Generates PDF of report
  
- **ExportReportsToCSV**(reportIds) → CSVFile
  - Exports report list to CSV

---

## Dependencies on Other Units

### Depends On:
- **User Management & Authentication Unit**: 
  - User validation
  - Permission checks
  - Session validation
  
- **Signature Workflow Engine Unit**:
  - Report status updates when signatures are added
  - Signature information for display

### Consumed By:
- **Signature Workflow Engine Unit**: Report data and status
- **System Administration Unit**: Report data for audit logs

---

## Business Rules

1. **Report ID Generation**: System generates unique report numbers automatically
2. **Draft Status**: New reports start in "Draft" status
3. **Required Fields**: All required fields must be completed before submission
4. **Date Validation**: Travel date cannot be in the future
5. **Time Validation**: End time must be after start time
6. **Image Limits**: Maximum 10 images per report, 15MB each
7. **File Formats**: Only JPG, JPEG, PNG, GIF, BMP, WEBP allowed
8. **Auto-save**: Draft reports auto-save every 2 minutes
9. **Edit Restrictions**: Only draft reports can be edited by users; admins can edit any report
10. **Soft Delete**: Deleted reports are archived for 30 days before permanent deletion
11. **Checklist Completion**: All 7 checklist items must be answered
12. **Activity Type**: If "Others" selected, description is required
13. **Creator Assignment**: Report creator is automatically set as "Prepared By"
14. **Status Progression**: Reports progress through: Draft → Pending Branch → Pending NISD → Completed

---

## UI Components (Desktop App)

### Screens/Forms:
1. **Dashboard Screen**
   - Report list with tabs (My Drafts, Pending Signature, Completed, All)
   - Search bar
   - Filter panel
   - Create New Report button
   - Report cards/grid

2. **Report Creation/Edit Form**
   - Multi-section form:
     - Basic Information section
     - Network Equipment section
     - Activity Checklist section
     - Image Upload section
   - Save Draft button
   - Submit for Signature button
   - Cancel button
   - Auto-save indicator

3. **Report View Screen**
   - Read-only report display
   - All sections organized
   - Image gallery
   - Signature status
   - Export buttons (PDF, Print)
   - Edit button (Admin only)
   - Delete button (Admin only)
   - Navigation buttons

4. **Search/Filter Panel**
   - Search text box
   - Filter dropdowns
   - Date range picker
   - Apply/Clear buttons
   - Results count

5. **Image Upload Dialog**
   - Drag-and-drop area
   - Browse button
   - Image preview thumbnails
   - Delete image buttons
   - Upload progress bars

6. **Export Dialog**
   - Export format selection
   - Progress indicator
   - Save location selector

---

## Validation Rules

### Branch/BLU/LC:
- Required
- Max 100 characters
- Alphanumeric with spaces

### Date of Travel:
- Required
- Valid date format
- Cannot be future date

### Time Started/Ended:
- Required
- Valid time format (HH:MM)
- End time must be after start time

### Purpose of Travel:
- Required
- Max 500 characters

### Network Equipment Fields:
- Optional
- Max 50 characters (except SD-WAN: 200 characters)
- Alphanumeric

### Switch Ports:
- Optional
- Numeric
- Range: 1-999

### Checklist Items:
- All 7 items required (Yes/No)
- Comments optional, max 500 characters

### Activity Type:
- Required
- If "Others" selected, description required (max 100 characters)

### Images:
- Max 10 images per report
- Max 15MB per image
- Allowed formats: JPG, JPEG, PNG, GIF, BMP, WEBP

---

## Error Handling

### Common Errors:
- **Validation Error**: "Please complete all required fields"
- **Date Error**: "Travel date cannot be in the future"
- **Time Error**: "End time must be after start time"
- **Image Size Error**: "Image exceeds 15MB limit"
- **Image Count Error**: "Maximum 10 images allowed per report"
- **File Format Error**: "Unsupported file format. Use JPG, PNG, GIF, BMP, or WEBP"
- **Save Error**: "Failed to save report. Please try again"
- **Permission Error**: "You do not have permission to edit this report"
- **Not Found Error**: "Report not found"
- **Concurrent Edit Error**: "This report has been modified by another user. Please refresh"

---

## Notes

- This is the core business unit of the system
- Should be built after User Management unit
- File storage should use local file system or network share (intranet environment)
- Consider implementing auto-save to prevent data loss
- Image files should be stored outside database (file system) with paths in database
- Export functionality should generate professional-looking PDFs
- Search and filter should be performant even with large datasets
- Consider implementing pagination for large result sets
- Edit history is important for audit and compliance
