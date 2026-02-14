# Agents

## What this does?

This repository contains specialized Claude Code agent definitions designed for building production-grade microservice platforms in Go. Each agent is a markdown file that codifies best practices for specific development tasks, from implementing database tables with 100% test coverage to generating OpenAPI specifications that stay in sync with your code.

## Why we use it?

Building a microservice platform requires consistency across dozens of services. Every service needs godoc-compliant documentation, database operations with comprehensive tests, properly structured README files, validated pub/sub topics, and OpenAPI specifications. When multiple developers work across multiple services, maintaining these standards manually becomes impossible. Patterns diverge, documentation falls out of sync, and quality varies dramatically.

These agents solve this problem by encoding Go and microservice best practices into enforceable, repeatable workflows. Each agent represents years of hard-won knowledge about what makes production code maintainable: strict naming conventions, 100% test coverage requirements, alphabetical ordering for readability, validation of schemas against code, and comprehensive documentation generation. When you invoke an agent, you're not just automating a task. You're ensuring every service in your platform follows the same battle-tested patterns, making your entire codebase more consistent, maintainable, and production-ready.

## How we use it?

These agents are designed to work with Claude Code as part of your development workflow. Each agent enforces specific architectural decisions and coding standards that make microservice platforms scalable and maintainable:

**Architecture Enforcement:**
- Service ownership boundaries (services only publish to their own pub/sub topics)
- Strict database schema validation with index requirements
- OpenAPI specs that match actual code, not outdated documentation
- Consistent JSON naming conventions (camelCase for new implementations)

**Go Best Practices:**
- Godoc-compliant comments on all exported code
- 100% test coverage using go-sqlmock for database operations
- Proper error handling with specific error types
- Struct validation with comprehensive test cases

**Development Workflow:**
- Agents are invoked automatically when relevant files change
- Documentation stays in sync with code through automated generation
- Quality checks happen before code review, not after
- Platform-wide conventions are enforced at creation time, not fixed later

The agents work together as a system: when you implement a database table, the `database-implementer` creates the code and tests, then automatically invokes `comment-writer` for documentation and `default-readme-writer` for package-level explanations. This cascading workflow ensures nothing is forgotten and every piece of your platform meets the same high standard.

## Further reading

- **Microservice Architecture** - A design pattern where applications are structured as collections of loosely coupled, independently deployable services that communicate through well-defined APIs
- **Godoc Documentation Standards** - Go's built-in documentation system that generates API documentation from specially formatted comments, making code self-documenting
- **Test-Driven Development (TDD)** - A practice where tests are written before code, ensuring 100% coverage and catching bugs early in the development cycle
- **Convention Over Configuration** - A software design paradigm that reduces decisions by establishing sensible defaults and patterns, critical for maintaining consistency across microservices
- **OpenAPI Specification** - A standard for describing REST APIs that enables automatic validation, client generation, and documentation that stays synchronized with code
- **Pub/Sub Messaging Patterns** - Event-driven communication where services publish events to topics and other services subscribe to relevant events, enabling loose coupling in distributed systems
