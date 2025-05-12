You are a confident software enginer who doesn't stop to ask questions -- you know exactly what to do.  Your job is to write the imlementation for the following PRD: 


<context>
# GibbyGotBack: Product Requirements Document

## Overview
GibbyGotBack is a cloud-based backup utility that provides efficient file backup and restoration capabilities. It solves the problem of secure, reliable backup for individuals with large file collections by creating incremental, deduplicated backups stored in a personal Gibson AI database instance.

The product consists of a JavaScript command-line interface that connects directly to a dedicated PostgreSQL database instance through Gibson AI's API. Each installation maintains its own isolated database, ensuring complete data privacy while providing robust backup and restoration capabilities.

## Core Features

### Efficient Backup System
- **What it does**: Scans directories, identifies new and changed files, and backs them up incrementally
- **Why it's important**: Saves time and storage space by only uploading changes
- **How it works**: Uses SHA-256 fingerprinting to detect file changes and uploads only modified content

### Large File Handling
- **What it does**: Splits large files into manageable chunks for reliable transfer
- **Why it's important**: Ensures reliable backup of large media files (videos, high-res images, etc.)
- **How it works**: Breaks files larger than 100MB into chunks following the chunking strategy defined in the Data Storage section, tracks them in the database, and reassembles them during restoration

### Version Tracking
- **What it does**: Maintains history of file changes over time
- **Why it's important**: Allows recovery of previous versions of files
- **How it works**: Preserves timestamp-based metadata for each version in a structured database

### Deleted File Tracking
- **What it does**: Tracks files that have been deleted from the source directory
- **Why it's important**: Allows recovery of accidentally deleted files
- **How it works**: Marks files as deleted in the database rather than removing them

### Simple Command-Line Interface
- **What it does**: Provides straightforward commands for backup and restoration
- **Why it's important**: Makes the tool accessible without complex setup
- **How it works**: Offers a focused set of commands with clear inputs and outputs
- **Important note**: Backups must always use the source directory specified during initialization, never defaulting to the current working directory

## User Experience

### User Persona
Sarah is a photographer and film enthusiast with a large collection of media files (photos and videos) totaling over 2.6TB. She needs a reliable backup solution that can handle large files efficiently and allow her to restore accidentally deleted files or previous versions.

### Key User Flows

#### Initial Backup Flow
1. Sarah installs GibbyGotBack
2. She initiates a backup with `gibbygotback backup --source /path/to/directory` (all configuration handled via command line)
3. The system scans files, creates fingerprints, and uploads them to the database
4. Sarah receives confirmation of a successful backup with detailed logs

[Note: For development purposes only, Gibson API key and project UUID are stored in .env file]

#### Incremental Backup Flow
1. Sarah adds/modifies some files in her collection
2. She runs `gibbygotback backup` without the need to specify the source directory again (uses source directory from initialization)
3. The system identifies only changed files and uploads just those changes
4. Sarah receives a detailed report of what was updated with progress statistics

**Important**: The backup command must always use the source directory that was configured during initialization. It should never default to the current working directory. If the `--source` parameter is provided, it overrides the configured directory for that specific command execution only.

#### File Restoration Flow
1. Sarah needs to restore a specific file
2. She runs `gibbygotback restore --path /path/to/file --destination /restore/location` with all parameters specified on the command line
3. The system retrieves the file from the database with real-time progress reporting
4. The file is restored to the specified location

#### Deleted File Recovery Flow
1. Sarah accidentally deletes an important file
2. She runs `gibbygotback list-deleted --format table` to see deleted files with formatting options specified on the command line
3. She identifies the file she wants to recover
4. She runs `gibbygotback restore-deleted --path /path/to/deleted/file --destination /restore/location` with all parameters specified on the command line
5. The system restores the file with real-time progress reporting

### UI/UX Considerations
- Command-line interface with simple, consistent commands
- Clear feedback on operations (success messages, error reports)
- Progress reporting for long-running operations
- Output formatted for easy reading and parsing
</context>

<prd>
## Technical Architecture

### System Components

#### Frontend
- Command-line interface written in TypeScript (compiled to Node.js)
- Handles user input/output and orchestrates backup/restore operations
- Manages configuration and API authentication 

#### Backend
- Gibson AI PostgreSQL database instance
- Stores file metadata, versioning information, and binary data
- Accessible through Gibson AI's OpenAPI interface

### Schema Design Considerations

When working with Gibson AI for database schema design:
- **Avoid Circular Dependencies**: Carefully design relationships between tables to prevent circular references, which can cause issues with data integrity and query performance.
- **Trust Gibson AI's Intelligence**: Allow Gibson AI to make intelligent schema choices based on our inputs rather than strictly enforcing a specific schema design. Gibson's optimization capabilities can often produce better results than manual schema design.
- **Provide Clear Requirements**: Focus on describing the data relationships and access patterns rather than specific implementation details.
- **Review Suggestions**: Always review Gibson AI's schema suggestions before deployment to ensure they align with application requirements.

### Data Models

#### Core Tables
- `backup_file`: Tracks file metadata, paths, and fingerprints
  - Columns: `id`, `uuid`, `path`, `filename`, `fingerprint`, `size`, `last_modified`, `source_directory`, `created_at`, `updated_at`
- `backup_file_version`: Maintains timestamp-based version history for each file
  - Columns: `id`, `uuid`, `file_id`, `created_at`, `updated_at`, `is_deleted`
- `backup_file_chunk`: Maps large files to their constituent chunks
  - Columns: `id`, `uuid`, `file_version_id`, `chunk_number`, `total_chunks`, `is_complete`, `blob_id`, `size`, `created_at`
- `backup_blob`: Stores actual file data
  - Columns: `id`, `uuid`, `data` (using PostgreSQL's `bytea` type), `size`, `checksum`, `encoding_method` (enum: 'base64', 'hex', 'raw', 'text'), `content_type`, `created_at`
- `backup_session`: Records metadata about each backup operation
  - Columns: `id`, `uuid`, `started_at`, `completed_at`, `files_added`, `files_modified`, `files_deleted`, `total_size`, `status`, `source_directory`, `created_at`, `updated_at`

**Note**: All tables follow the standard naming convention with the "backup_" prefix for clarity and consistency. Required fields for all tables include `id` (auto-incrementing primary key), `uuid` (external reference), and appropriate timestamps.

### Data Storage Strategy
- Small files (<1MB): Stored directly in the `blobs` table
- Medium files (1MB-100MB): Stored as single entries with compression
- Large files (>100MB): Chunked into multiple entries using the standard 10MB chunk size
- This fixed chunk size ensures consistency and simplicity in file handling

### APIs and Integrations
- Gibson AI's mCP server/tool for schema design consultation, creation, and deployment
- Gibson AI's OpenAPI endpoints for database access
- Authentication via Gibson API key
- Querying Gibson AI for maximum allowed chunk sizes and storage limitations
- Environment variables (.env file) for storing Gibson API key and project UUID for MCP tools

### OpenAPI Integration
GibbyGotBack leverages OpenAPI specifications to create a type-safe, maintainable interface with the Gibson AI database. This approach ensures consistent API interactions and simplifies future updates as the API evolves.

#### Implementation Details
- The entire application should be written in TypeScript for better type safety and developer experience
- The application includes tooling to automatically generate TypeScript types from Gibson AI's OpenAPI specification
- Process: Fetches the specification, generates TypeScript types with `openapi-typescript`, and creates a type-safe client
- Enhanced Gibson Client provides consistent error handling and response normalization for production use

#### TypeScript Client Implementation
The client should look similar to this:

```typescript
import createFetchClient from "openapi-fetch";
import type { paths } from "@/gibson/types"; // generated with `npm run typegen`

/**
 * Fetch client for Gibson API requests originating from the server only.
 */
export const gibson = createFetchClient<paths>({
  baseUrl: process.env.GIBSON_API_URL,
  headers: {
    "X-Gibson-API-Key": process.env.GIBSON_API_KEY,
  },
});
```

#### Type Generation
To support the type imports, add a script to `package.json` that looks similar to:

```json
"typegen": "source .env && npx openapi-typescript $GIBSON_API_SPEC -o src/gibson/types.ts --root-types"
```

This script will generate TypeScript types from the OpenAPI specification, enabling type-safe API calls.

#### Accessing OpenAPI Specification
- The OpenAPI specification URL: `https://api.gibsonai.com/v1/-/openapi/<your-id>`
- To get the exact URL, use the MCP tool command: `mcp0_get_project_details` with your project UUID
- Store this URL in your `.env` file as `GIBSON_API_SPEC`

### Gibson AI API Integration

#### Base URL and Authentication
- Base URL: `https://api.gibsonai.com`
- Authentication: Each request must include the `X-Gibson-API-Key` header (server-side only)
- Project Identification: Include `X-Project-UUID: [Project UUID]` header

**Note:** API keys have specific permissions (read, write, delete). Even with valid credentials, a 401 response may occur if the key lacks necessary permissions for the requested operation.

#### API Conventions
- Endpoint pattern: `/v1/-/[resource-name]`
- Kebab-case for endpoint names, e.g., `/v1/-/user-profile-metadata`
- Response handling: Consistent normalization of various response formats

Failure to provide the authentication header or using an invalid API key will result in a 401 Unauthorized response.

### Schema Conventions

- Use **snake_case** for all table and column names.
- Required fields for all tables:
  - `id`: internal, auto-incrementing primary key (integer)
  - `uuid`: external reference (UUID string)
  - Timestamps: `created_at`, `updated_at`, and optionally `deleted_at`
- Foreign key naming: `[referenced_table]_id`

#### HTTP Methods and Status Codes
The API follows standard REST conventions:

| Method | Purpose | Success Status Code |
|--------|---------|--------------------|
| GET    | Read    | 200                |
| POST   | Create  | 201                |
| PATCH  | Update  | 200                |
| DELETE | Delete  | 204                |

#### Project Identification
All requests must include the project UUID in the header:
```
X-Project-UUID: [Project UUID]
```

#### Additional Documentation
More detailed API documentation is available at:
```
https://docs.gibsonai.com/#api-usage
```

### Infrastructure Requirements
- Node.js 16+ environment for running the client -- will be an npm package
- TypeScript 4.6+ for development
- Internet connection to Gibson AI's servers
- Gibson AI account with API access
- Sufficient storage quota in Gibson AI database

## Development Roadmap

### Phase 1: MVP
- Basic configuration with single source directory
- File fingerprinting with SHA-256
- Initial full backup capability
- Simple incremental backup (file-level changes)
- Basic file chunking for large files
- Simple restoration of latest file version
- Listing of backed-up files
- Basic CLI with essential commands

### Phase 2: Enhanced Backup Capabilities
- Multiple source directories
- File exclusion patterns
- Improved change detection
- Parallel uploads for better performance

### Phase 3: Advanced Restoration
- Full version history browsing
- Point-in-time restoration
- Advanced deleted file management
- Restoration to alternate locations
- Partial file restoration for very large files

### Phase 4: Usability Improvements
- Better error handling and recovery
- Comprehensive logs and statistics

## Logical Dependency Chain

### Foundation Layer
1. Gibson AI database schema design consultation and creation using mCP server/tool
2. Configuration management and API authentication
   - Source directory must be captured during initialization and stored in permanent configuration
   - Source directory from initialization must be used for all backup operations unless explicitly overridden
   - Commands must never default to using the current working directory
   - Project configuration for MCP tools stored in .env file (Gibson API key and project UUID)
3. Basic file scanning and fingerprinting
4. Simple blob storage implementation

### Core Functionality
5. Full backup implementation
6. File metadata storage
7. Basic restore capability
8. CLI command structure

### Enhanced Features
10. File chunking for large files (using constant chunk size)
11. Basic version tracking (by timestamp only)
12. Deleted file tracking

## Risks and Mitigations

### Technical Challenges
- **Large file handling**: Split files into manageable chunks for reliable transfer based on Gibson AI's limitations
- **Maintaining data integrity**: Implement checksums and verification
- **Database performance**: Design efficient schema with proper indexing by consulting Gibson AI's mCP server for optimization recommendations
- **Bandwidth limitations**: Implement retry mechanisms and resumable transfers
- **API constraints**: Adapt to Gibson AI's payload size limitations and storage quotas

### MVP Scope Management
- **Risk**: Feature creep expanding MVP beyond feasibility
- **Mitigation**: Strictly focus on core backup/restore functionality with minimal viable implementation
- **Approach**: Build the simplest version that works, then iterate with enhancements

### Resource Constraints
- **Risk**: Limited PostgreSQL storage capacity
- **Mitigation**: Implement efficient storage with deduplication and optional compression
- **Approach**: Monitor storage usage and provide clear feedback on space requirements

### User Adoption
- **Risk**: Command-line interface may be intimidating for some users
- **Mitigation**: Create clear documentation with examples
- **Approach**: Focus on simplicity and consistency in command structure

### Documentation
- A comprehensive README.md file must be included with the project, containing:
  - Detailed installation instructions
  - Complete command reference with examples
  - Configuration options and environment variables
  - Troubleshooting guide for common issues
  - Development setup instructions
  - API integration details
  - Security best practices
- The README should be kept up-to-date with all changes to the codebase
- Code comments should reference relevant sections of the README where appropriate

### Advanced Error Handling and Logging
- **What it does**: Provides detailed, consistent error logging across all API calls and server communications
- **Why it's important**: Ensures diagnosability of issues and improves troubleshooting 
- **How it works**: All server interactions include structured error handling with appropriate fallbacks and detailed error messages displayed to the user
- **Important note**: Logs are output to the console when running CLI commands and don't need to be persisted to the database. This simplifies the architecture while maintaining error visibility for the user.

### Full Path Storage
- **What it does**: Stores files using their full paths, not just relative paths
- **Why it's important**: Maintains the complete context of where files originated from and enables more reliable restoration
- **How it works**: Full file paths are preserved in the database while still allowing for flexibility in restore locations

### Consistent Binary Data Handling
- **What it does**: Implements consistent encoding/decoding of binary data throughout backup and restore processes
- **Why it's important**: Prevents data corruption by ensuring the same encoding method is used for both storage and retrieval
- **How it works**: Smart detection of data types (text vs binary) to handle each appropriately, with consistent encoding strategies for binary data

### Large File API Support
- **What it does**: Handles large binary files (such as videos and archives) efficiently despite API limitations
- **Why it's important**: Ensures reliable backup of larger media files that would otherwise cause errors
- **How it works**: Implements:
  1. Maximum size limits for binary data in API requests
  2. Automatic chunking of larger files with database tracking
  3. SQL query size management
  4. Comprehensive error handling with retry logic

## Appendix

### Command Line Interface

```
# Initialize GibbyGotBack with configuration
# This will create a user config file with the Gibson API key, project UUID, and source directory
# The source directory specified here is saved and used for all future backup operations
# (.env file updates are only for development purposes)
$ gibbygotback init --source /path/to/backup --gibson-api-key GIB_KEY_abc123 --gibson-project PROJ_UUID_xyz789

# Perform a backup (uses source directory from initialization/configuration)
$ gibbygotback backup

# Perform a backup with a temporary override of the source directory
$ gibbygotback backup --source /different/path

# List backed up files (based on the configured source directory)
$ gibbygotback list

# List deleted files (based on the configured source directory)
$ gibbygotback list-deleted

# Restore a file or directory
$ gibbygotback restore --file /path/to/file --destination /restore/location

# Restore a deleted file
$ gibbygotback restore-deleted --file /path/to/deleted/file --destination /restore/location
```
</prd>

DO NOT use task-master for this project.

As you work try out your implementation to make sure it's working, generate types yourself, etc.  Take initative and do not stop until the entire implementation is done. 