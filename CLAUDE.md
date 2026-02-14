# Platform Development Guidelines

## CRITICAL: Follow the CLAUDE.md Hierarchy

**You MUST read and follow ALL `CLAUDE.md` files in the directory chain from your current working directory up to the repository root.**

### How the Hierarchy Works

When working in any directory, follow instructions in this order:

1. **Current directory** - `./CLAUDE.md` (most specific)
2. **Parent directory** - `../CLAUDE.md`
3. **Grandparent directory** - `../../CLAUDE.md`
4. **Root directory** - `/CLAUDE.md` (most general, this file)

### Example

If you're working in `/services/example/`:
1. Read `/services/example/CLAUDE.md` (service-specific context)
2. Read `/services/CLAUDE.md` (ALL service workflows and patterns)
3. Read `/CLAUDE.md` (this file - platform-wide guidelines)

**All instructions in all files must be followed.**

---

## Agent Reference Guide

**Invoke these agents at the appropriate times when working on this codebase:**

### comment-writer
**Invoke after:** Writing or modifying Go code (functions, structs, constants, variables)
**Purpose:** Adds godoc-compliant comments to Go code with proper formatting, parameter documentation, and error handling notes
**Always required for:** New Go code, exported functions/types, database implementations

### database-implementer
**Invoke when:** Creating or refactoring MySQL database tables with CRUD operations
**Purpose:** Implements complete database table interactions with 100% test coverage using go-sqlmock, strict validation, and proper error handling
**Requirements:** Service name, table name, schema (columns, types, indexes), operations to implement

### default-readme-writer
**Invoke after:** Creating or modifying non-service directories (libraries, scripts, tools)
**Purpose:** Generates educational README.md files following the 4-section format (What/Why/How/Further Reading)
**Do NOT use for:** Service directories - use service-readme-writer instead

### service-readme-writer
**Invoke after:** Creating/modifying service endpoints, database tables, pub/sub topics, or other service components
**Purpose:** Generates comprehensive service README.md with all 11 required sections (endpoints, database tables, pub/sub, tasks, etc.)
**Automatically extracts:** Environment variables, API endpoints, database tables, pub/sub topics, outbox tasks, cron jobs

### openapi-writer
**Invoke after:** Adding or modifying service API endpoints
**Purpose:** Generates/updates OpenAPI 3.0.3 specification from code, validates schemas match current implementation
**Validates:** Request/response schemas, authentication, realistic examples, proper status codes

### pubsub-publish-implementer
**Invoke when:** Creating new pub/sub topics that a service will publish
**Purpose:** Implements pub/sub topic definitions with proper naming conventions (kebab-case), message structs, and camelCase JSON tags
**Enforces:** Service ownership (topics must match service name), alphabetical ordering, strict naming validation

---
