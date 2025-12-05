# Technical Stack

## Platform

- **Application Type**: Windows Desktop Application (C# .NET 8 Windows Forms)
- **Deployment**: Intranet only (no internet access required)
- **Database**: SQLite (embedded database file)
- **Architecture**: Modular Monolith with 4 business units
- **Framework**: .NET 8.0
- **UI Framework**: Windows Forms

## Architecture Units

1. **User Management & Authentication**: Account management, login, sessions, RBAC
2. **Report Lifecycle Management**: Report CRUD, search/filter, image storage, export
3. **Signature Workflow Engine**: Sequential signature workflow, validation, in-app notifications
4. **System Administration**: Audit logs, configuration, backup/restore

## Database

- Single SQLite database file shared across units
- Each unit owns specific tables
- Referential integrity enforced at database level with foreign keys
- Soft delete pattern for user accounts and reports
- Data Access: Repository pattern with ADO.NET or Entity Framework Core
- Database File: Local file (e.g., TravelReports.db)
- Migration Path: Design with comments for future MSSQL migration if needed

## Security

- Password hashing: bcrypt or Argon2
- Session management with 30-minute timeout
- Role-based access control (Admin/User)
- HTTPS/TLS for data in transit
- Encryption at rest for sensitive data
- Account lockout after 5 failed login attempts

## Data Validation

- Multi-layer validation: UI, business logic, database constraints
- Real-time field validation with inline error messages
- Required field enforcement before report submission
- File type and size validation for image uploads

## Common Patterns

- **Soft Delete**: Archive records instead of permanent deletion
- **Audit Trail**: Log all critical operations with timestamp and user
- **Sequential Workflow**: Enforce step-by-step signature process
- **Repository Pattern**: Data access abstraction for each unit
- **Service Interfaces**: Well-defined boundaries between units

## File Handling

- **Image Formats**: JPG, JPEG, PNG, GIF, BMP, WEBP
- **Max File Size**: 15MB per image
- **Max Images**: 10 per report
- **Storage**: File system or database (implementation decision)

## Export Capabilities

- PDF export for individual reports
- CSV export for report lists
- Filtered result exports

## Development Workflow

The project follows a structured development process documented in `/Prompts/`:

1. **Step 1.1**: Create user stories with acceptance criteria
2. **Step 1.2**: Group user stories into business units
3. **Step 2.1**: Design architecture for each unit
4. **Step 2.2**: Create logical design
5. **Step 2.3**: Implement source code
6. **Step 2.4**: Debug source code
7. **Step 2.5**: Create tests

## Key Constraints

- Support 50+ concurrent users
- Page load time under 3 seconds
- Session timeout: 30 minutes of inactivity
- Password history: Last 3 passwords
- Backup retention: 30 days
- Audit log retention: Minimum 1 year
