# README Section Definitions

## Contents
- [Section 1: Service Name & High-Level Description](#section-1-service-name--high-level-description)
- [Section 2: Overview](#section-2-overview)
- [Section 3: Getting Started](#section-3-getting-started)
- [Section 4: Environment Variables](#section-4-environment-variables)
- [Section 5: API Endpoints](#section-5-api-endpoints)
- [Section 6: Pub/Sub Topics](#section-6-pubsub-topics)
- [Section 7: Pub/Sub Subscriptions](#section-7-pubsub-subscriptions)
- [Section 8: Cloud Tasks](#section-8-cloud-tasks)
- [Section 9: Cronjobs](#section-9-cronjobs)
- [Section 10: Outbox Tasks](#section-10-outbox-tasks)
- [Section 11: Database Tables](#section-11-database-tables)
- [Code Path Mapping Rules](#code-path-mapping-rules)

---

## Section 1: Service Name & High-Level Description

```markdown
# {Service Name}

{Two-sentence, non-technical description of what this service does, designed for humans new to the codebase.}
```

---

## Section 2: Overview

```markdown
## Overview

{Two-paragraph overview explaining what the service does, important things to know about it, and why it's important to the platform.}
```

---

## Section 3: Getting Started

```markdown
## Getting Started

Before running the service for the first time, make sure you have the **Google Cloud CLI (gcloud)** installed. If you haven't installed it yet, follow the instructions [here](https://cloud.google.com/sdk/docs/install).

Once installed, authenticate your account with Google Cloud and configure the project by running the following commands:

\```shell
gcloud auth login                             # Sign in to the Google Cloud SDK
gcloud auth application-default login         # Sign in to the Google Auth Library
gcloud config set project {YOUR_GCP_PROJECT_ID}  # Set the active Google Cloud project
GOPRIVATE=github.com/{YOUR_GITHUB_ORG} go mod tidy      # Manage Go dependencies
\```

## Running
> **NOTE**: _The first time you run the service locally, you'll be walked through a short environment variable setup process._

To run the service locally, execute:
\```shell
go run .
\```
```

If this section already exists in the README, preserve it as-is.

---

## Section 4: Environment Variables

**Table Format**: 3 columns — NAME, DEFAULT, DESCRIPTION
**Sort**: Alphabetically by NAME

**How to Extract**:
1. Read `main.go` and find `type EnvVars struct`
2. Extract field names and their `default:` tag values
3. Generate descriptions by analyzing usage context in the code

```markdown
## Environment Variables

| NAME | DEFAULT | DESCRIPTION |
|:-----|:--------|:------------|
| CLOUD_SQL_CONNECTION | {YOUR_GCP_PROJECT_ID}:us-east1:shared | Cloud SQL instance connection string |
| GCP_PROJECT_ID | {YOUR_GCP_PROJECT_ID} | Google Cloud Platform project identifier |
| HOST | :8080 | Port for the HTTP server to listen on |
```

If no environment variables exist, show `None`.

---

## Section 5: API Endpoints

**Table Format**: 5 columns — METHOD, PATH, PERMISSIONS, DESCRIPTION, CODEBASE
**Sort**: Alphabetically by PATH

**How to Extract**:
1. Read `main.go` and find all `s.AddPublicEndpoint()` and `s.AddAuthenticatedEndpoint()` calls
2. Extract: HTTP method, path, permission scope (authenticated only — leave empty for public), handler function name
3. Map handler to file: `endpoints/{method}__{path}.go` (lowercase, underscores replace slashes/hyphens)
4. Read the handler file and generate a concise description

```markdown
## API Endpoints

| METHOD | PATH | PERMISSIONS | DESCRIPTION | CODEBASE |
|:-------|:-----|:------------|:------------|:---------|
| GET | / | | Returns basic information about the service. | [View Code](endpoints/get__root.go) |
| POST | /archives/create | zip.archive.creator | Creates a ZIP archive from specified GCS files. | [View Code](endpoints/post__archives_create.go) |
```

---

## Section 6: Pub/Sub Topics

**Table Format**: 3 columns — TOPIC, DESCRIPTION, CODEBASE
**Sort**: Alphabetically by TOPIC

**How to Extract**:
1. Check if `pkg/pubsub/types.go` exists
2. Find `const` blocks containing topic definitions (format: `{service}.{event-name}`)
3. Generate descriptions from constant name, associated message struct, and code comments

```markdown
## Pub/Sub Topics

| TOPIC | DESCRIPTION | CODEBASE |
|:------|:------------|:---------|
| orders.manager-assigned | Published when a manager is assigned to an order. | [View Code](pkg/pubsub/types.go) |
| orders.order-created | Published when a new order is created. | [View Code](pkg/pubsub/types.go) |
```

If none, show `None`.

---

## Section 7: Pub/Sub Subscriptions

**Table Format**: 3 columns — TOPIC, DESCRIPTION, CODEBASE
**Sort**: Alphabetically by TOPIC

**How to Extract**:
1. Read `main.go` and find `s.AddPubSubEndpoint()` calls
2. Extract topic from the path: `/_pubsub/{topic-name}`
3. Map to handler file: `endpoints/pubsub__{topic-name}.go`
4. Read handler and generate description

```markdown
## Pub/Sub Subscriptions

| TOPIC | DESCRIPTION | CODEBASE |
|:------|:------------|:---------|
| billing.transaction-created | Handles new transaction events from the billing service. | [View Code](endpoints/pubsub__billing_transaction-created.go) |
```

If none, show `None`.

---

## Section 8: Cloud Tasks

**Table Format**: 3 columns — NAME, DESCRIPTION, CODEBASE
**Sort**: Alphabetically by NAME

**How to Extract**:
1. Read `main.go` and find `s.AddCloudTaskEndpoint()` calls
2. Extract task name from path: `/_tasks/{task-name}`
3. Map to handler file: `endpoints/task__{task-name}.go`

```markdown
## Cloud Tasks

| NAME | DESCRIPTION | CODEBASE |
|:-----|:------------|:---------|
| evaluate-timetable | Advances an order's phase if timetable criteria are met. | [View Code](endpoints/task__evaluate-timetable.go) |
```

If none, show `None`.

---

## Section 9: Cronjobs

**Table Format**: 3 columns — PATH, DESCRIPTION, CODEBASE
**Sort**: Alphabetically by PATH

**How to Extract**:
1. Read `main.go` and find `s.AddCloudSchedulerEndpoint()` calls
2. Extract cron path (format: `/_cron/{cron-name}`)
3. Map to handler file: `endpoints/cron__{cron-name}.go`

```markdown
## Cronjobs

| PATH | DESCRIPTION | CODEBASE |
|:-----|:------------|:---------|
| /_cron/clean-database | Periodically removes expired or obsolete records. | [View Code](endpoints/cron__clean-database.go) |
```

If none, show `None`.

---

## Section 10: Outbox Tasks

**Table Format**: 3 columns — NAME, DESCRIPTION, CODEBASE
**Sort**: Alphabetically by NAME

**How to Extract**:
1. Check if `outboxes/definitions/definitions.go` exists
2. Find `const {TASK_NAME} = "{task-identifier}"` definitions
3. Find corresponding handler in `outboxes/{task-identifier}.go`
4. Generate description from payload struct and handler logic

```markdown
## Outbox Tasks

| NAME | DESCRIPTION | CODEBASE |
|:-----|:------------|:---------|
| assign-manager | Assigns a manager to an order from the roster service. | [View Code](outboxes/assign-manager.go) |
```

If none, show `None`.

---

## Section 11: Database Tables

**Table Format**: 3 columns — TABLE, DESCRIPTION, CODEBASE
**Sort**: Alphabetically by TABLE

**How to Extract**:
1. Check if `internal/database/` directory exists
2. List all `.go` files EXCEPT `database.go` (shared utilities)
3. File name maps to table name (e.g., `user_preferences.go` -> `user_preferences`)
4. Generate description from struct definitions, function signatures, and comments

```markdown
## Database Tables

| TABLE | DESCRIPTION | CODEBASE |
|:------|:------------|:---------|
| orders | Stores currently open or ongoing orders. | [View Code](internal/database/orders.go) |
| participants | Stores all participants assigned to orders. | [View Code](internal/database/participants.go) |
```

If none, show `None`.

---

## Code Path Mapping Rules

### Endpoint Handlers
- Registration: `s.AddPublicEndpoint("POST", "/orders/create", ...)`
- File: `endpoints/post__orders_create.go`
- Pattern: `{method}__{path_with_underscores}.go`

### Pub/Sub Handlers
- Registration: `s.AddPubSubEndpoint("/_pubsub/billing.transaction-created", ...)`
- File: `endpoints/pubsub__billing_transaction-created.go`
- Pattern: `pubsub__{topic_with_hyphens}.go`

### Task Handlers
- Registration: `s.AddCloudTaskEndpoint("/_tasks/evaluate-timetable", ...)`
- File: `endpoints/task__evaluate-timetable.go`
- Pattern: `task__{task_name}.go`

### Cron Handlers
- Registration: `s.AddCloudSchedulerEndpoint("/_cron/process-outbox", ...)`
- File: `endpoints/cron__process-outbox.go`
- Pattern: `cron__{cron_name}.go`

### Outbox Handlers
- Definition: `const PUBLISH_ORDER_CREATED = "publish-order-created"`
- File: `outboxes/publish-order-created.go`
- Pattern: Matches the constant value exactly
