# Agent Definitions

## What this does?

This directory contains specialized agent instruction files that define how Claude Code should handle specific development tasks like writing documentation, implementing database operations, and generating OpenAPI specifications. Each markdown file provides detailed workflows, validation rules, and quality standards that Claude follows when performing these tasks.

## Why we use it?

When building software at scale, consistency and quality become critical challenges. Every developer writes documentation differently, implements database operations with varying patterns, and generates API specs using their own conventions. This inconsistency makes code harder to maintain, review, and understand. Without standardized processes, teams spend significant time in code review catching issues like missing validation, incomplete test coverage, or non-compliant documentation formats.

Agent definitions solve this by encoding best practices, organizational standards, and proven workflows into reusable instructions that Claude Code can execute. Instead of each developer reinventing the wheel or remembering dozens of style rules, agents ensure that database implementations always achieve 100% test coverage, README files follow the 4-section structure, and OpenAPI specs use realistic examples with proper authentication documentation. This shifts quality enforcement from manual code review to automated execution, freeing developers to focus on solving business problems rather than formatting documentation or remembering validation patterns.

## How we use it?

Agent definitions work with Claude Code to automate complex development tasks. Each agent is invoked by name and handles a specific type of work:

```markdown
# Example 1: Generate a README for a package
# The default-readme-writer agent analyzes code and creates documentation
User: "Use the default-readme-writer to create a README for /services/orders/internal/database"

# Example 2: Implement database operations with full test coverage
# The database-implementer agent creates table files, tests, and documentation
User: "Use database-implementer to add notifications table to alerts service:
      - id VARCHAR(36) PRIMARY KEY
      - user_id VARCHAR(36) NOT NULL INDEX idx_user
      - message TEXT NOT NULL
      Operations: CreateNotification, GetNotification, MarkAsRead"

# Example 3: Document API endpoints in OpenAPI format
# The openapi-writer agent extracts endpoint definitions from code
User: "Use openapi-writer to document all endpoints in the zip service"

# Example 4: Add comprehensive godoc comments to Go code
# The comment-writer agent adds structured documentation to functions and types
User: "Use comment-writer to add comments to services/orders/internal/database/orders.go"

# Example 5: Create a pub/sub topic definition
# The pubsub-publish-implementer agent creates topic constants and message structs
User: "Use pubsub-publish-implementer to add topic orders.manager-assigned with:
      - orderID (string) - Order identifier
      - managerID (string) - Manager user ID
      - assignedAt (time.Time) - Assignment timestamp"
```

Each agent follows strict workflows defined in its markdown file, ensuring consistent output quality, proper validation, and complete documentation every time.

## Further reading

- **Agent-Driven Development** - A paradigm where AI agents encode organizational knowledge and automate repetitive development tasks to maintain consistency across codebases
- **Workflow Automation** - The practice of defining repeatable processes that can be executed programmatically to reduce manual effort and human error
- **Code Generation Standards** - Conventions and rules that ensure generated code follows team practices for testing, documentation, and architecture
- **Documentation as Code** - Treating documentation generation as a programming task with inputs, validation, and outputs rather than manual writing
- **Quality Gates** - Automated checks and requirements (like 100% test coverage) that prevent substandard code from being accepted into a codebase
