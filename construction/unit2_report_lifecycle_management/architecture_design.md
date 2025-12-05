# Unit 2: Report Lifecycle Management - Architecture Design

## 1. Unit Overview

**Business Capability**: Manage the complete lifecycle of Travel Accomplishment Reports from creation to archival.

**Responsibilities**:
- Report creation and data entry (travel details, network equipment, activity checklist)
- Image upload and storage management
- Report viewing, searching, and filtering
- Report editing (Admin only) and deletion (Admin only)
- Export functionality (PDF, CSV)
- Report status management

**Scope**:
- Core business functionality of the system
- Manages all report data and associated files
- Provides search and filter capabilities
- Handles report exports
- Integrates with signature workflow for status updates

**Technology Stack**:
- Platform: C# .NET 8 Windows Forms
- Data Access: Entity Framework Core with SQLite
- File Storage: File system with paths in database
- PDF Generation: iTextSharp or QuestPDF
- CSV Export: CsvHelper library
- Dependency Injection: Microsoft.Extensions.DependencyInjection

---

## 2. Feature Components

### 2.1 Report Creation Feature
**Responsibility**: Create new travel accomplishment reports with all required data

**UI Components**:
- ReportEditorForm (Windows Form)
  - Basic Information section (Branch, Date, Time, Purpose)
  - Network Equipment section (Telco 1/2, Switch 1/2, SD-WAN)
  - Activity Checklist section (7 checklist items with Yes/No and comments)
  - Image Upload section (drag-drop, browse, preview)
  - Save Draft button
  - Submit for Signature button
  - Cancel button
  - Auto-save indicator

**Feature Operations**:
- Create new report in Draft status
- Auto-populate creator as "Prepared By"
- Generate unique report number
- Validate required fields
- Auto-save every 2 minutes
- Manual save draft
- Submit for signature workflow

### 2.2 Report Viewing Feature
**Responsibility**: Display complete report details in read-only format

**UI Components**:
- ReportViewForm (Windows Form)
  - Report header (ID, Branch, Date, Status)
  - Basic Information display
  - Network Equipment display
  - Activity Checklist display
  - Image gallery with zoom capability
  - Signature section with status
  - Export to PDF button
  - Print button
  - Edit button (Admin only)
  - Delete button (Admin only)
  - Previous/Next navigation buttons

**Feature Operations**:
- Display all report sections
- Show images in gallery view
- Display signature status
- Navigate between reports
- Export to PDF
- Print report

### 2.3 Report Dashboard Feature
**Responsibility**: Display categorized list of reports with quick access

**UI Components**:
- ReportDashboardForm (Windows Form)
  - Tab control with categories:
    - My Drafts
    - Pending My Signature
    - Completed Reports
    - All Reports (Admin only)
  - Report cards/grid with key information
  - Report count badges per tab
  - Create New Report button
  - Quick action buttons per report
  - Pagination controls

**Feature Operations**:
- List reports by category
- Display report counts
- Quick navigation to report details
- Pagination for large result sets (20 per page)
- Sort by date, status, branch

### 2.4 Search and Filter Feature
**Responsibility**: Find reports based on various criteria

**UI Components**:
- SearchFilterPanel (User Control)
  - Search text box with real-time search
  - Date range picker (From/To)
  - Status filter dropdown
  - Branch filter dropdown
  - Activity Type filter dropdown
  - Created By filter dropdown
  - Apply Filters button
  - Clear All Filters button
  - Results count label
  - Export Filtered Results button

**Feature Operations**:
- Real-time search across multiple fields
- Apply multiple filters simultaneously
- Persist filter selections during session
- Export filtered results
- Display result count

### 2.5 Image Management Feature
**Responsibility**: Upload, store, and manage report images

**UI Components**:
- ImageUploadControl (User Control)
  - Drag-and-drop area
  - Browse button
  - Image preview thumbnails
  - Delete image buttons
  - Upload progress bars
  - Image count indicator (X/10)
  - File size display per image

**Feature Operations**:
- Upload images (JPG, JPEG, PNG, GIF, BMP, WEBP)
- Validate file type and size (max 15MB)
- Limit to 10 images per report
- Store files in file system
- Associate images with checklist items
- Delete uploaded images
- Display thumbnails with zoom

### 2.6 Report Export Feature
**Responsibility**: Export reports in various formats

**UI Components**:
- ExportDialog (Modal Dialog)
  - Export format selection (PDF, CSV)
  - Single/Multiple report selection
  - Progress indicator
  - Save location selector
  - Export button
  - Cancel button

**Feature Operations**:
- Export single report to PDF
- Export multiple reports to ZIP
- Export report list to CSV
- Generate formatted PDF with images and signatures
- Track export history

### 2.7 Admin Report Management Feature
**Responsibility**: Admin-only editing and deletion of reports

**UI Components**:
- AdminReportEditorForm (Windows Form)
  - Same as ReportEditorForm but accessible for any report
  - Edit history viewer
  - Reason for edit text box
  - Save Changes button with confirmation

**Feature Operations**:
- Edit any report regardless of status
- Track all changes in edit history
- Log admin edits with reason
- Notify report creator of changes
- Soft delete reports with reason
- Restore archived reports

---

## 3. Business Services

### 3.1 ReportManagementService
**Responsibility**: Core report CRUD operations

**Public Operations**:
- CreateReportAsync(userId, reportData) → int (reportId)
  - Generates unique report number
  - Sets status to Draft
  - Sets creator as Prepared By
  - Initializes workflow
  - Returns report ID
  
- UpdateReportAsync(reportId, reportData, userId) → bool
  - Validates report is Draft or user is Admin
  - Updates report data
  - Records edit history if Admin
  - Returns success status
  
- GetReportByIdAsync(reportId) → ReportDetailsDto
  - Retrieves complete report with all sections
  - Includes images, signatures, equipment, checklist
  - Returns null if not found or deleted
  
- GetReportsByUserAsync(userId, includeDeleted) → List<ReportSummaryDto>
  - Returns all reports created by user
  - Optionally includes deleted reports
  
- SearchReportsAsync(searchCriteria) → PagedResult<ReportSummaryDto>
  - Searches by multiple criteria
  - Returns paginated results
  - Supports sorting
  
- DeleteReportAsync(reportId, userId, reason) → bool
  - Admin only (soft delete)
  - Sets IsDeleted flag
  - Records deletion reason
  - Logs to audit trail
  
- RestoreReportAsync(reportId) → bool
  - Admin only
  - Restores archived report
  - Clears IsDeleted flag

**Dependencies**:
- IReportRepository
- INetworkEquipmentRepository
- IActivityChecklistRepository
- IWorkflowService (Unit 3)
- IAuditLogService (Unit 4)
- IAuthorizationService (Unit 1)

### 3.2 ReportStatusService
**Responsibility**: Manage report status transitions

**Public Operations**:
- GetReportStatusAsync(reportId) → string
  - Returns current status
  - Statuses: Draft, Pending Branch, Pending NISD, Completed
  
- UpdateReportStatusAsync(reportId, newStatus) → bool
  - Updates report status
  - Called by Signature Workflow unit
  - Validates status transition
  
- ValidateReportCompleteAsync(reportId) → ValidationResult
  - Checks all required fields filled
  - Validates checklist completion
  - Returns validation result with errors
  
- CanSubmitForSignatureAsync(reportId) → bool
  - Checks if report ready for signature workflow
  - Validates all required data present

**Dependencies**:
- IReportRepository
- IActivityChecklistRepository

### 3.3 ImageManagementService
**Responsibility**: Handle image upload, storage, and retrieval

**Public Operations**:
- UploadImageAsync(reportId, imageFile, checklistItem, userId) → int (imageId)
  - Validates file type and size
  - Checks image count limit (10)
  - Generates unique filename
  - Stores file in file system
  - Records metadata in database
  - Returns image ID
  
- GetReportImagesAsync(reportId) → List<ReportImageDto>
  - Returns all images for report
  - Includes metadata (filename, size, upload date)
  
- DeleteImageAsync(imageId, userId) → bool
  - Removes image from report
  - Deletes file from file system
  - Soft deletes database record
  
- GetImageFilePathAsync(imageId) → string
  - Returns full file path for image
  - Used for display and export
  
- ValidateImageFileAsync(file) → ValidationResult
  - Validates file type
  - Validates file size
  - Returns validation result

**Dependencies**:
- IReportImageRepository
- IFileStorageService
- IReportRepository

### 3.4 ReportExportService
**Responsibility**: Generate report exports in various formats

**Public Operations**:
- ExportReportToPdfAsync(reportId) → byte[] (PDF file)
  - Generates formatted PDF
  - Includes all report data
  - Embeds images
  - Shows signatures with timestamps
  - Returns PDF as byte array
  
- ExportReportsToZipAsync(reportIds) → byte[] (ZIP file)
  - Generates individual PDFs for each report
  - Packages into ZIP file
  - Returns ZIP as byte array
  
- ExportReportsToCsvAsync(reportIds) → byte[] (CSV file)
  - Exports report list to CSV
  - Includes all filterable fields
  - Returns CSV as byte array
  
- GenerateReportNumberAsync() → string
  - Generates unique report number
  - Format: TAR-YYYYMMDD-XXXX

**Dependencies**:
- IReportRepository
- IReportImageRepository
- ISignatureService (Unit 3)
- PDF generation library (iTextSharp/QuestPDF)
- CsvHelper library

### 3.5 FileStorageService
**Responsibility**: Manage file system operations for images

**Public Operations**:
- SaveFileAsync(fileData, fileName) → string (filePath)
  - Saves file to file system
  - Organizes by date (YYYY/MM/DD structure)
  - Returns relative file path
  
- DeleteFileAsync(filePath) → bool
  - Deletes file from file system
  - Returns success status
  
- GetFileAsync(filePath) → byte[]
  - Reads file from file system
  - Returns file as byte array
  
- FileExistsAsync(filePath) → bool
  - Checks if file exists
  
- GetFileInfoAsync(filePath) → FileInfo
  - Returns file metadata

**Dependencies**:
- System.IO
- Configuration for storage path

### 3.6 ReportValidationService
**Responsibility**: Validate report data at various stages

**Public Operations**:
- ValidateBasicInformationAsync(reportData) → ValidationResult
  - Validates required fields
  - Validates date/time logic
  - Returns validation errors
  
- ValidateNetworkEquipmentAsync(equipmentData) → ValidationResult
  - Validates equipment data formats
  - Validates numeric ranges
  
- ValidateActivityChecklistAsync(checklistData) → ValidationResult
  - Validates all items answered
  - Validates activity type selection
  
- ValidateForSubmissionAsync(reportId) → ValidationResult
  - Comprehensive validation before signature
  - Checks all sections complete
  - Returns detailed validation result

**Dependencies**:
- IReportRepository
- IActivityChecklistRepository
- IReportImageRepository

---

## 4. Data Models

### 4.1 Report Entity
**Maps to**: Reports table

**Properties**:
- ReportId (int, primary key)
- ReportNumber (string, unique, required, max 50)
- BranchName (string, required, max 100)
- DateOfTravel (DateTime, required)
- TimeStarted (TimeSpan, required)
- TimeEnded (TimeSpan, required)
- PurposeOfTravel (string, required, max 500)
- ReportStatus (string, required, max 50)
- CreatedBy (int, foreign key to Users, required)
- CreatedDate (DateTime, required)
- LastModifiedBy (int, foreign key to Users, nullable)
- LastModifiedDate (DateTime, nullable)
- IsDeleted (bool, required, default false)
- DeletedBy (int, foreign key to Users, nullable)
- DeletedDate (DateTime, nullable)
- DeletedReason (string, nullable, max 500)

**Relationships**:
- CreatedByUser (User) - navigation property
- LastModifiedByUser (User) - navigation property
- DeletedByUser (User) - navigation property
- NetworkEquipment (NetworkEquipment) - one-to-one
- ActivityChecklist (ActivityChecklist) - one-to-one
- Images (collection of ReportImage) - one-to-many
- EditHistory (collection of ReportEditHistory) - one-to-many

**Validation Rules**:
- BranchName: required, max 100 characters
- DateOfTravel: required, cannot be future date
- TimeStarted: required
- TimeEnded: required, must be after TimeStarted
- PurposeOfTravel: required, max 500 characters
- ReportStatus: must be valid status value

### 4.2 NetworkEquipment Entity
**Maps to**: NetworkEquipment table

**Properties**:
- EquipmentId (int, primary key)
- ReportId (int, foreign key, required, unique)
- Telco1Name (string, nullable, max 50)
- Telco1RouterBrand (string, nullable, max 50)
- Telco1ModemBrand (string, nullable, max 50)
- Telco2Name (string, nullable, max 50)
- Telco2RouterBrand (string, nullable, max 50)
- Telco2ModemBrand (string, nullable, max 50)
- Switch1Brand (string, nullable, max 50)
- Switch1Model (string, nullable, max 50)
- Switch1Ports (int, nullable)
- Switch1PN (string, nullable, max 50)
- Switch2Brand (string, nullable, max 50)
- Switch2Model (string, nullable, max 50)
- Switch2Ports (int, nullable)
- Switch2PN (string, nullable, max 50)
- SDWANLightsStatus (string, nullable, max 200)

**Relationships**:
- Report (Report) - navigation property

**Validation Rules**:
- All fields optional
- String fields: max length as specified
- Switch ports: range 1-999 if provided

### 4.3 ActivityChecklist Entity
**Maps to**: ActivityChecklist table

**Properties**:
- ChecklistId (int, primary key)
- ReportId (int, foreign key, required, unique)
- Item1_ImageBefore (bool, required)
- Item1_Comment (string, nullable, max 500)
- Item2_PhysicalConnection (bool, required)
- Item2_Comment (string, nullable, max 500)
- Item3_EquipmentWorking (bool, required)
- Item3_Comment (string, nullable, max 500)
- Item4_ActivityType (string, required, max 50)
- Item4_ActivityOther (string, nullable, max 100)
- Item4_Comment (string, nullable, max 500)
- Item5_EquipmentVerified (bool, required)
- Item5_Comment (string, nullable, max 500)
- Item6_CablesTagged (bool, required)
- Item6_Comment (string, nullable, max 500)
- Item7_ImageAfter (bool, required)
- Item7_Comment (string, nullable, max 500)

**Relationships**:
- Report (Report) - navigation property

**Validation Rules**:
- All item responses (bool) required
- Item4_ActivityType: required, must be valid type
- Item4_ActivityOther: required if ActivityType is "Others"
- Comments: optional, max 500 characters

### 4.4 ReportImage Entity
**Maps to**: ReportImages table

**Properties**:
- ImageId (int, primary key)
- ReportId (int, foreign key, required)
- FileName (string, required, max 255)
- FileSize (long, required)
- FileType (string, required, max 50)
- FilePath (string, required, max 500)
- UploadedBy (int, foreign key to Users, required)
- UploadedDate (DateTime, required)
- ChecklistItem (int, nullable)
- IsDeleted (bool, required, default false)

**Relationships**:
- Report (Report) - navigation property
- UploadedByUser (User) - navigation property

**Validation Rules**:
- FileName: required, max 255 characters
- FileSize: required, max 15MB (15728640 bytes)
- FileType: must be valid image MIME type
- FilePath: required, max 500 characters
- ChecklistItem: range 1-7 if provided

### 4.5 ReportEditHistory Entity
**Maps to**: ReportEditHistory table

**Properties**:
- HistoryId (int, primary key)
- ReportId (int, foreign key, required)
- EditedBy (int, foreign key to Users, required)
- EditedDate (DateTime, required)
- FieldName (string, required, max 100)
- OldValue (string, nullable)
- NewValue (string, nullable)

**Relationships**:
- Report (Report) - navigation property
- EditedByUser (User) - navigation property

**Business Logic**:
- Records all admin edits
- Tracks field-level changes
- Used for audit trail

---

## 5. Repository Pattern


### 5.1 IReportRepository Interface
**Responsibility**: Data access for Report entity

**Operations**:
- GetByIdAsync(reportId, includeDeleted) → Report
- GetByReportNumberAsync(reportNumber) → Report
- GetByUserAsync(userId, includeDeleted) → List<Report>
- GetAllAsync(includeDeleted) → List<Report>
- SearchAsync(searchCriteria) → PagedResult<Report>
- CreateAsync(report) → Report
- UpdateAsync(report) → Report
- DeleteAsync(reportId, deletedBy, reason) → bool
- RestoreAsync(reportId) → bool
- GetReportCountByStatusAsync(userId, status) → int

**Implementation**: ReportRepository (EF Core)
- Uses DbContext for database operations
- Implements async operations
- Handles soft delete filtering
- Includes related entities (NetworkEquipment, ActivityChecklist, Images)
- Supports complex search queries
- Implements pagination

### 5.2 INetworkEquipmentRepository Interface
**Responsibility**: Data access for NetworkEquipment entity

**Operations**:
- GetByReportIdAsync(reportId) → NetworkEquipment
- CreateAsync(equipment) → NetworkEquipment
- UpdateAsync(equipment) → NetworkEquipment
- DeleteAsync(equipmentId) → bool

**Implementation**: NetworkEquipmentRepository (EF Core)
- One-to-one relationship with Report
- Created/updated with report

### 5.3 IActivityChecklistRepository Interface
**Responsibility**: Data access for ActivityChecklist entity

**Operations**:
- GetByReportIdAsync(reportId) → ActivityChecklist
- CreateAsync(checklist) → ActivityChecklist
- UpdateAsync(checklist) → ActivityChecklist
- DeleteAsync(checklistId) → bool
- IsCompleteAsync(reportId) → bool

**Implementation**: ActivityChecklistRepository (EF Core)
- One-to-one relationship with Report
- Validates all items answered

### 5.4 IReportImageRepository Interface
**Responsibility**: Data access for ReportImage entity

**Operations**:
- GetByIdAsync(imageId) → ReportImage
- GetByReportIdAsync(reportId, includeDeleted) → List<ReportImage>
- GetImageCountAsync(reportId) → int
- CreateAsync(image) → ReportImage
- DeleteAsync(imageId) → bool
- GetByChecklistItemAsync(reportId, checklistItem) → List<ReportImage>

**Implementation**: ReportImageRepository (EF Core)
- Manages image metadata
- Supports soft delete
- Enforces 10 image limit per report

### 5.5 IReportEditHistoryRepository Interface
**Responsibility**: Data access for ReportEditHistory entity

**Operations**:
- GetByReportIdAsync(reportId) → List<ReportEditHistory>
- CreateAsync(history) → ReportEditHistory
- GetRecentEditsAsync(reportId, count) → List<ReportEditHistory>

**Implementation**: ReportEditHistoryRepository (EF Core)
- Tracks all admin edits
- Provides audit trail

---

## 6. Service Interfaces (Exposed to Other Units)

### 6.1 IReportManagementService Interface
**Exposed Operations**:
- CreateReportAsync(userId, reportData) → int
- UpdateReportAsync(reportId, reportData, userId) → bool
- GetReportByIdAsync(reportId) → ReportDetailsDto
- GetReportsByUserAsync(userId) → List<ReportSummaryDto>
- SearchReportsAsync(searchCriteria) → PagedResult<ReportSummaryDto>
- DeleteReportAsync(reportId, userId, reason) → bool
- RestoreReportAsync(reportId) → bool

**Data Contracts**:

**ReportDetailsDto**:
- ReportId (int)
- ReportNumber (string)
- BranchName (string)
- DateOfTravel (DateTime)
- TimeStarted (TimeSpan)
- TimeEnded (TimeSpan)
- PurposeOfTravel (string)
- ReportStatus (string)
- CreatedBy (int)
- CreatedByName (string)
- CreatedDate (DateTime)
- NetworkEquipment (NetworkEquipmentDto)
- ActivityChecklist (ActivityChecklistDto)
- Images (List<ReportImageDto>)
- Signatures (List<SignatureDto>)

**ReportSummaryDto**:
- ReportId (int)
- ReportNumber (string)
- BranchName (string)
- DateOfTravel (DateTime)
- ReportStatus (string)
- CreatedByName (string)
- LastModifiedDate (DateTime?)

**ReportCreateRequest**:
- BranchName (string)
- DateOfTravel (DateTime)
- TimeStarted (TimeSpan)
- TimeEnded (TimeSpan)
- PurposeOfTravel (string)
- NetworkEquipment (NetworkEquipmentDto)
- ActivityChecklist (ActivityChecklistDto)

### 6.2 IReportStatusService Interface
**Exposed Operations**:
- GetReportStatusAsync(reportId) → string
- UpdateReportStatusAsync(reportId, newStatus) → bool
- ValidateReportCompleteAsync(reportId) → ValidationResult
- CanSubmitForSignatureAsync(reportId) → bool

**Data Contracts**:

**ValidationResult**:
- IsValid (bool)
- Errors (List<string>)

### 6.3 IImageManagementService Interface
**Exposed Operations**:
- UploadImageAsync(reportId, imageFile, checklistItem, userId) → int
- GetReportImagesAsync(reportId) → List<ReportImageDto>
- DeleteImageAsync(imageId, userId) → bool
- GetImageFilePathAsync(imageId) → string

**Data Contracts**:

**ReportImageDto**:
- ImageId (int)
- ReportId (int)
- FileName (string)
- FileSize (long)
- FileType (string)
- UploadedByName (string)
- UploadedDate (DateTime)
- ChecklistItem (int?)

**ImageFileData**:
- FileName (string)
- FileData (byte[])
- FileType (string)
- FileSize (long)

### 6.4 IReportExportService Interface
**Exposed Operations**:
- ExportReportToPdfAsync(reportId) → byte[]
- ExportReportsToZipAsync(reportIds) → byte[]
- ExportReportsToCsvAsync(reportIds) → byte[]

---

## 7. Dependencies

### 7.1 Internal Dependencies
- Entity Framework Core (data access)
- iTextSharp or QuestPDF (PDF generation)
- CsvHelper (CSV export)
- System.IO.Compression (ZIP creation)
- Microsoft.Extensions.DependencyInjection (DI container)

### 7.2 External Unit Dependencies
**Depends On**:
- Unit 1 (User Management & Authentication):
  - IAuthenticationService for session validation
  - IAuthorizationService for permission checks
  - IUserManagementService for user details
- Unit 3 (Signature Workflow Engine):
  - IWorkflowService for workflow initialization
  - ISignatureService for signature information (display)
- Unit 4 (System Administration):
  - IAuditLogService for logging report operations
  - IConfigurationService for system settings

**Consumed By**:
- Unit 3 (Signature Workflow Engine): Report data and status
- Unit 4 (System Administration): Report data for statistics

### 7.3 Database Dependencies
**Tables Owned**:
- Reports
- NetworkEquipment
- ActivityChecklist
- ReportImages
- ReportEditHistory

**Foreign Key References**:
- Reports.CreatedBy → Users.UserId (Unit 1)
- Reports.LastModifiedBy → Users.UserId (Unit 1)
- Reports.DeletedBy → Users.UserId (Unit 1)
- NetworkEquipment.ReportId → Reports.ReportId
- ActivityChecklist.ReportId → Reports.ReportId
- ReportImages.ReportId → Reports.ReportId
- ReportImages.UploadedBy → Users.UserId (Unit 1)
- ReportEditHistory.ReportId → Reports.ReportId
- ReportEditHistory.EditedBy → Users.UserId (Unit 1)

---

## 8. Component Diagram

```
┌──────────────────────────────────────────────────────────────────┐
│              Unit 2: Report Lifecycle Management                  │
├──────────────────────────────────────────────────────────────────┤
│                                                                   │
│  ┌───────────────────────────────────────────────────────────┐  │
│  │              UI Layer (Windows Forms)                      │  │
│  │  ┌──────────┐  ┌──────────┐  ┌──────────┐  ┌──────────┐  │  │
│  │  │Report    │  │Report    │  │Report    │  │Search/   │  │  │
│  │  │Editor    │  │View      │  │Dashboard │  │Filter    │  │  │
│  │  │Form      │  │Form      │  │Form      │  │Panel     │  │  │
│  │  └────┬─────┘  └────┬─────┘  └────┬─────┘  └────┬─────┘  │  │
│  │       │             │             │             │         │  │
│  │  ┌────▼─────┐  ┌───▼──────┐  ┌───▼──────┐  ┌───▼──────┐  │  │
│  │  │Image     │  │Export    │  │Admin     │  │          │  │  │
│  │  │Upload    │  │Dialog    │  │Report    │  │          │  │  │
│  │  │Control   │  │          │  │Editor    │  │          │  │  │
│  │  └────┬─────┘  └────┬─────┘  └────┬─────┘  └──────────┘  │  │
│  └───────┼─────────────┼─────────────┼────────────────────────┘  │
│          │             │             │                            │
│  ┌───────▼─────────────▼─────────────▼────────────────────────┐  │
│  │              Business Services Layer                        │  │
│  │  ┌──────────────────┐  ┌──────────────────┐               │  │
│  │  │ReportManagement  │  │ReportStatus      │               │  │
│  │  │Service           │  │Service           │               │  │
│  │  └────────┬─────────┘  └────────┬─────────┘               │  │
│  │           │                     │                          │  │
│  │  ┌────────▼─────────┐  ┌───────▼──────────┐              │  │
│  │  │ImageManagement   │  │ReportExport      │              │  │
│  │  │Service           │  │Service           │              │  │
│  │  └────────┬─────────┘  └───────┬──────────┘              │  │
│  │           │                     │                          │  │
│  │  ┌────────▼─────────┐  ┌───────▼──────────┐              │  │
│  │  │FileStorage       │  │ReportValidation  │              │  │
│  │  │Service           │  │Service           │              │  │
│  │  └────────┬─────────┘  └───────┬──────────┘              │  │
│  └───────────┼─────────────────────┼──────────────────────────┘  │
│              │                     │                              │
│  ┌───────────▼─────────────────────▼──────────────────────────┐  │
│  │              Repository Layer                               │  │
│  │  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐     │  │
│  │  │Report        │  │NetworkEquip  │  │ActivityCheck │     │  │
│  │  │Repository    │  │Repository    │  │Repository    │     │  │
│  │  └──────┬───────┘  └──────┬───────┘  └──────┬───────┘     │  │
│  │         │                 │                 │              │  │
│  │  ┌──────▼───────┐  ┌──────▼───────────────────────────┐   │  │
│  │  │ReportImage   │  │ReportEditHistory                 │   │  │
│  │  │Repository    │  │Repository                        │   │  │
│  │  └──────┬───────┘  └──────┬───────────────────────────┘   │  │
│  └─────────┼──────────────────┼─────────────────────────────┘  │
│            │                  │                                 │
│  ┌─────────▼──────────────────▼─────────────────────────────┐  │
│  │              Data Access Layer (EF Core)                  │  │
│  │  ┌────────────────────────────────────────────────────┐  │  │
│  │  │            DbContext (SQLite)                       │  │  │
│  │  │  - Reports                                          │  │  │
│  │  │  - NetworkEquipment                                 │  │  │
│  │  │  - ActivityChecklist                                │  │  │
│  │  │  - ReportImages                                     │  │  │
│  │  │  - ReportEditHistory                                │  │  │
│  │  └────────────────────────────────────────────────────┘  │  │
│  └───────────────────────────────────────────────────────────┘  │
│                                                                  │
│  ┌──────────────────────────────────────────────────────────┐  │
│  │         Service Interfaces (Exposed to Other Units)       │  │
│  │  - IReportManagementService                               │  │
│  │  - IReportStatusService                                   │  │
│  │  - IImageManagementService                                │  │
│  │  - IReportExportService                                   │  │
│  └──────────────────────────────────────────────────────────┘  │
│                                                                  │
│  ┌──────────────────────────────────────────────────────────┐  │
│  │         File System Storage                               │  │
│  │  - Images stored in: /AppData/Images/YYYY/MM/DD/         │  │
│  │  - Organized by upload date                              │  │
│  └──────────────────────────────────────────────────────────┘  │
└──────────────────────────────────────────────────────────────────┘
```

---

## 9. Database Schema

### 9.1 Reports Table
```
Table: Reports
Primary Key: ReportId (INT, IDENTITY)
Indexes:
  - IX_Reports_ReportNumber (UNIQUE)
  - IX_Reports_CreatedBy
  - IX_Reports_DateOfTravel
  - IX_Reports_ReportStatus
  - IX_Reports_IsDeleted
  - IX_Reports_BranchName

Columns:
  ReportId            INT             PRIMARY KEY IDENTITY
  ReportNumber        NVARCHAR(50)    NOT NULL UNIQUE
  BranchName          NVARCHAR(100)   NOT NULL
  DateOfTravel        DATE            NOT NULL
  TimeStarted         TIME            NOT NULL
  TimeEnded           TIME            NOT NULL
  PurposeOfTravel     NVARCHAR(500)   NOT NULL
  ReportStatus        NVARCHAR(50)    NOT NULL
  CreatedBy           INT             NOT NULL FOREIGN KEY REFERENCES Users(UserId)
  CreatedDate         DATETIME        NOT NULL DEFAULT CURRENT_TIMESTAMP
  LastModifiedBy      INT             NULL FOREIGN KEY REFERENCES Users(UserId)
  LastModifiedDate    DATETIME        NULL
  IsDeleted           BIT             NOT NULL DEFAULT 0
  DeletedBy           INT             NULL FOREIGN KEY REFERENCES Users(UserId)
  DeletedDate         DATETIME        NULL
  DeletedReason       NVARCHAR(500)   NULL
```

### 9.2 NetworkEquipment Table
```
Table: NetworkEquipment
Primary Key: EquipmentId (INT, IDENTITY)
Indexes:
  - IX_NetworkEquipment_ReportId (UNIQUE)

Columns:
  EquipmentId         INT             PRIMARY KEY IDENTITY
  ReportId            INT             NOT NULL UNIQUE FOREIGN KEY REFERENCES Reports(ReportId)
  Telco1Name          NVARCHAR(50)    NULL
  Telco1RouterBrand   NVARCHAR(50)    NULL
  Telco1ModemBrand    NVARCHAR(50)    NULL
  Telco2Name          NVARCHAR(50)    NULL
  Telco2RouterBrand   NVARCHAR(50)    NULL
  Telco2ModemBrand    NVARCHAR(50)    NULL
  Switch1Brand        NVARCHAR(50)    NULL
  Switch1Model        NVARCHAR(50)    NULL
  Switch1Ports        INT             NULL CHECK (Switch1Ports BETWEEN 1 AND 999)
  Switch1PN           NVARCHAR(50)    NULL
  Switch2Brand        NVARCHAR(50)    NULL
  Switch2Model        NVARCHAR(50)    NULL
  Switch2Ports        INT             NULL CHECK (Switch2Ports BETWEEN 1 AND 999)
  Switch2PN           NVARCHAR(50)    NULL
  SDWANLightsStatus   NVARCHAR(200)   NULL
```

### 9.3 ActivityChecklist Table
```
Table: ActivityChecklist
Primary Key: ChecklistId (INT, IDENTITY)
Indexes:
  - IX_ActivityChecklist_ReportId (UNIQUE)

Columns:
  ChecklistId                 INT             PRIMARY KEY IDENTITY
  ReportId                    INT             NOT NULL UNIQUE FOREIGN KEY REFERENCES Reports(ReportId)
  Item1_ImageBefore           BIT             NOT NULL
  Item1_Comment               NVARCHAR(500)   NULL
  Item2_PhysicalConnection    BIT             NOT NULL
  Item2_Comment               NVARCHAR(500)   NULL
  Item3_EquipmentWorking      BIT             NOT NULL
  Item3_Comment               NVARCHAR(500)   NULL
  Item4_ActivityType          NVARCHAR(50)    NOT NULL
  Item4_ActivityOther         NVARCHAR(100)   NULL
  Item4_Comment               NVARCHAR(500)   NULL
  Item5_EquipmentVerified     BIT             NOT NULL
  Item5_Comment               NVARCHAR(500)   NULL
  Item6_CablesTagged          BIT             NOT NULL
  Item6_Comment               NVARCHAR(500)   NULL
  Item7_ImageAfter            BIT             NOT NULL
  Item7_Comment               NVARCHAR(500)   NULL
```

### 9.4 ReportImages Table
```
Table: ReportImages
Primary Key: ImageId (INT, IDENTITY)
Indexes:
  - IX_ReportImages_ReportId
  - IX_ReportImages_UploadedBy
  - IX_ReportImages_IsDeleted

Columns:
  ImageId             INT             PRIMARY KEY IDENTITY
  ReportId            INT             NOT NULL FOREIGN KEY REFERENCES Reports(ReportId)
  FileName            NVARCHAR(255)   NOT NULL
  FileSize            BIGINT          NOT NULL CHECK (FileSize <= 15728640)
  FileType            NVARCHAR(50)    NOT NULL
  FilePath            NVARCHAR(500)   NOT NULL
  UploadedBy          INT             NOT NULL FOREIGN KEY REFERENCES Users(UserId)
  UploadedDate        DATETIME        NOT NULL DEFAULT CURRENT_TIMESTAMP
  ChecklistItem       INT             NULL CHECK (ChecklistItem BETWEEN 1 AND 7)
  IsDeleted           BIT             NOT NULL DEFAULT 0
```

### 9.5 ReportEditHistory Table
```
Table: ReportEditHistory
Primary Key: HistoryId (INT, IDENTITY)
Indexes:
  - IX_ReportEditHistory_ReportId
  - IX_ReportEditHistory_EditedBy
  - IX_ReportEditHistory_EditedDate

Columns:
  HistoryId           INT             PRIMARY KEY IDENTITY
  ReportId            INT             NOT NULL FOREIGN KEY REFERENCES Reports(ReportId)
  EditedBy            INT             NOT NULL FOREIGN KEY REFERENCES Users(UserId)
  EditedDate          DATETIME        NOT NULL DEFAULT CURRENT_TIMESTAMP
  FieldName           NVARCHAR(100)   NOT NULL
  OldValue            NVARCHAR(MAX)   NULL
  NewValue            NVARCHAR(MAX)   NULL
```

### 9.6 Relationships
- Reports.CreatedBy → Users.UserId
- Reports.LastModifiedBy → Users.UserId
- Reports.DeletedBy → Users.UserId
- NetworkEquipment.ReportId → Reports.ReportId (one-to-one)
- ActivityChecklist.ReportId → Reports.ReportId (one-to-one)
- ReportImages.ReportId → Reports.ReportId (one-to-many)
- ReportImages.UploadedBy → Users.UserId
- ReportEditHistory.ReportId → Reports.ReportId (one-to-many)
- ReportEditHistory.EditedBy → Users.UserId

---

## 10. Implementation Notes


### 10.1 Entity Framework Core Configuration
- Use Code-First approach with migrations
- Configure one-to-one relationships (Report-NetworkEquipment, Report-ActivityChecklist)
- Configure one-to-many relationships (Report-Images, Report-EditHistory)
- Implement soft delete global query filter for Reports and ReportImages
- Use async operations throughout
- Configure indexes for search performance
- Set up cascade delete rules (restrict for user references, cascade for child entities)

### 10.2 File Storage Strategy
- Store images in file system, not database
- Organize by date: /AppData/Images/YYYY/MM/DD/
- Generate unique filenames: {ReportId}_{Guid}.{extension}
- Store relative path in database
- Configure storage root path in appsettings
- Implement file cleanup for deleted images (background job)
- Validate file types using MIME type checking
- Implement file size validation before upload

### 10.3 Report Number Generation
- Format: TAR-YYYYMMDD-XXXX
- TAR = Travel Accomplishment Report
- YYYYMMDD = Date of creation
- XXXX = Sequential number for the day (0001, 0002, etc.)
- Ensure uniqueness with database constraint
- Handle concurrent creation with retry logic

### 10.4 Auto-Save Implementation
- Implement timer in ReportEditorForm (2-minute interval)
- Save draft automatically if changes detected
- Display auto-save indicator (last saved timestamp)
- Handle auto-save failures gracefully
- Don't interrupt user input during auto-save

### 10.5 Search and Filter Performance
- Create indexes on frequently searched columns
- Use EF Core query optimization
- Implement pagination to limit result sets
- Consider full-text search for large datasets
- Cache filter dropdown values (branches, activity types)
- Use async operations to keep UI responsive

### 10.6 PDF Export Implementation
- Use iTextSharp or QuestPDF library
- Create professional report template
- Include all report sections
- Embed images with appropriate sizing
- Show signatures with timestamps
- Add page numbers and headers/footers
- Generate table of contents for multi-page reports
- Optimize PDF size for file sharing

### 10.7 Image Handling
- Validate file type using MIME type (not just extension)
- Supported types: image/jpeg, image/png, image/gif, image/bmp, image/webp
- Maximum file size: 15MB (15,728,640 bytes)
- Maximum images per report: 10
- Display thumbnails in UI (generate on-the-fly or cache)
- Implement zoom functionality for image viewing
- Handle image rotation if needed (EXIF data)
- Compress large images before storage (optional)

### 10.8 Validation Strategy
- Multi-layer validation:
  - UI: Real-time validation with visual feedback
  - Service: Business rule validation
  - Entity: Data annotation attributes
  - Database: Constraints and check rules
- Use FluentValidation library for complex validation
- Return detailed validation results with field-specific errors
- Validate before database operations
- Implement custom validators for business rules

### 10.9 Error Handling
- Use try-catch blocks in service methods
- Return meaningful error messages to UI
- Log exceptions to audit log (Unit 4)
- Handle file system errors (disk full, permissions)
- Handle database errors (constraint violations, deadlocks)
- Implement retry logic for transient failures
- Display user-friendly error messages
- Never expose sensitive information in errors

### 10.10 Dependency Injection Setup
- Register services with appropriate lifetime:
  - Scoped: Services, Repositories, DbContext
  - Singleton: Configuration, FileStorageService
  - Transient: Validators, Export services
- Use constructor injection throughout
- Register interfaces with implementations
- Configure in Program.cs or Startup class

### 10.11 Testing Considerations
- Unit test services with mocked repositories
- Integration test repositories with in-memory SQLite
- Test file upload/download operations
- Test PDF generation
- Test search and filter logic
- Test validation rules
- Test soft delete behavior
- Mock external unit dependencies (Unit 1, 3, 4)

### 10.12 Performance Considerations
- Use async/await throughout
- Implement connection pooling (EF Core default)
- Add indexes on frequently queried columns
- Use pagination for large result sets
- Lazy load images (load on demand)
- Cache dropdown values (branches, activity types)
- Optimize PDF generation for large reports
- Implement background jobs for cleanup tasks
- Use Include() for eager loading related entities

### 10.13 Security Best Practices
- Validate all file uploads
- Sanitize file names before storage
- Check file content, not just extension
- Implement path traversal protection
- Validate user permissions before operations
- Use parameterized queries (EF Core default)
- Implement CSRF protection if web-based
- Secure file storage location
- Regular security audits

---

## 11. Integration Points

### 11.1 With Unit 1 (User Management & Authentication)
- Call IAuthenticationService.ValidateSessionAsync() before all operations
- Call IAuthorizationService.CheckPermissionAsync() for:
  - Report editing (own vs any)
  - Report deletion (admin only)
  - Admin features access
- Call IUserManagementService.GetUserByIdAsync() for:
  - Display creator name
  - Display modifier name
  - Display uploader name
- Receive user context (userId) from authenticated session

### 11.2 With Unit 3 (Signature Workflow Engine)
- Call IWorkflowService.InitializeWorkflowAsync() when report submitted
- Receive status updates from IReportStatusService.UpdateReportStatusAsync()
- Provide report data via IReportManagementService.GetReportByIdAsync()
- Validate report completeness via IReportStatusService.ValidateReportCompleteAsync()
- Display signature information in report view

### 11.3 With Unit 4 (System Administration)
- Call IAuditLogService.LogActionAsync() for:
  - Report creation
  - Report modification
  - Report deletion
  - Image upload/deletion
  - Admin edits
  - Export operations
- Call IConfigurationService.GetConfigurationAsync() for:
  - Auto-save interval
  - Maximum file size
  - Maximum images per report
  - File storage path
- Provide report data for statistics via IReportManagementService

### 11.4 Application Startup
- Initialize DbContext with SQLite connection
- Run EF Core migrations for Report tables
- Create file storage directories if not exist
- Register all services in DI container
- Configure file storage path from settings
- Start background job for file cleanup

---

## 12. Build Order and Dependencies

### 12.1 Build Order
1. Data Models (Entities)
2. Repository Interfaces
3. Repository Implementations
4. Service Interfaces
5. Validation Services
6. File Storage Service
7. Core Services (ReportManagement, ReportStatus)
8. Image Management Service
9. Export Service
10. UI Components
11. Integration Testing

### 12.2 Critical Path
- Build after Unit 1 (User Management & Authentication)
- Required by Unit 3 (Signature Workflow Engine)
- Core business functionality of the system
- Database schema must be stable before Unit 3

---

## 13. Future Enhancements

### 13.1 Potential Improvements
- Report templates for common scenarios
- Bulk report operations
- Advanced search with saved filters
- Report comparison feature
- Report versioning
- Collaborative editing
- Mobile app for field data entry
- Offline mode with sync
- OCR for equipment labels
- Barcode/QR code scanning for equipment
- Integration with network monitoring tools
- Automated report generation from monitoring data
- Report scheduling and reminders
- Custom report fields (configurable)
- Report analytics and insights

### 13.2 Scalability Considerations
- Move to cloud storage for images (Azure Blob, AWS S3)
- Implement caching layer (Redis) for frequently accessed reports
- Consider separate read/write databases (CQRS pattern)
- Implement full-text search engine (Elasticsearch)
- Add CDN for image delivery
- Implement report archival strategy
- Consider microservices architecture for large scale

---

**Document Version**: 1.0  
**Date**: December 5, 2025  
**Status**: Ready for Implementation  
**Technology**: C# .NET 8, Windows Forms, Entity Framework Core, SQLite
