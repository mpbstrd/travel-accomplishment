# Project Structure

## Root Directory

```
/
├── .git/                    # Version control
├── .kiro/                   # Kiro configuration and steering rules
│   └── steering/            # AI assistant steering documents
├── context/                 # Reference documents
│   └── Branch Travel Checklist v3.docx  # Source form specification
├── inception/               # Requirements and design phase
│   ├── README.md            # Inception phase overview
│   ├── overview_user_stories.md  # Complete user stories (50+ stories, 10 epics)
│   └── units/               # Business unit specifications
│       ├── unit1_user_management_authentication.md
│       ├── unit2_report_lifecycle_management.md
│       ├── unit3_signature_workflow_engine.md
│       └── integration_contract.md  # Service interfaces between units
├── Prompts/                 # Development workflow prompts
│   ├── Step 1.1_ Create User Stories
│   ├── Step 1.2_ Grouping User Stories into Units
│   ├── Step 2.1_ Architecture Design
│   ├── Step 2.2_ Create Logical Design
│   ├── Step 2.3_ Implement Source Code
│   ├── Step 2.4_ Debugging Source Code
│   └── Step 2.5_ Create Tests
└── plan.md                  # Current phase execution plan
```

## Key Directories

### `/inception/`
Contains all requirements documentation:
- **overview_user_stories.md**: Master document with all user stories, acceptance criteria, data dictionary, and traceability matrix
- **units/**: Individual business unit specifications with user stories, database schemas, and service interfaces
- **README.md**: Summary of inception phase deliverables

### `/context/`
Reference materials and source documents:
- Original Branch Travel Checklist form (DOCX)
- Any additional business context documents

### `/Prompts/`
Step-by-step development workflow instructions:
- Each file contains role definition, planning requirements, and task specifications
- Follow sequential order for structured development
- Each step requires plan creation, approval, and execution

### `/construction/` (Future)
Will contain architecture design and implementation for each unit:
- `/construction/{unit_name}/architecture_design.md`
- Source code organized by business unit

## Documentation Standards

### User Stories Format
```
**As a** [role]
**I want to** [action]
**So that** [benefit]

**Acceptance Criteria:**
- Criterion 1
- Criterion 2
```

### Unit Documentation Structure
Each unit document includes:
1. Unit Overview (business capability, responsibility, scope)
2. User Stories with acceptance criteria
3. Database Tables Owned
4. Service Interfaces Exposed
5. Dependencies on Other Units
6. Business Rules
7. UI Components
8. Validation Rules
9. Error Handling
10. Implementation Notes

### Planning Documents
- Use markdown with checkboxes for task tracking
- Include critical decisions section requiring confirmation
- Mark completed steps with [x]
- Document assumptions and constraints

## Naming Conventions

- **Files**: Use underscores for spaces (e.g., `user_management_authentication.md`)
- **User Stories**: Format as `US-X.Y` (Epic.Story)
- **Database Tables**: PascalCase (e.g., `Users`, `TravelReports`)
- **Columns**: PascalCase (e.g., `UserID`, `CreatedDate`)

## Development Approach

1. **Feature-Based Architecture**: Organize by business capability
2. **Service-Oriented Patterns**: Clear interfaces between units
3. **Modular Monolith**: Independent units within single application
4. **Build Order**: User Management → Report Lifecycle → Signature Workflow → System Administration

## Cross-Cutting Concerns

- **Validation**: Each unit handles its own domain validation
- **Security**: Split between User Management (authentication) and System Admin (authorization)
- **UI/UX**: Distributed across relevant units (desktop app)
- **Audit Logging**: Centralized in System Administration unit
- **Error Handling**: Consistent patterns across all units
