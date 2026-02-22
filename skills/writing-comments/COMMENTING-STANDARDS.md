# Commenting Standards by Element Type

## Contents
- [Functions](#functions)
- [Structs](#structs)
- [Variables and Constants](#variables-and-constants)
- [Inline Comments (Inside Functions)](#inline-comments-inside-functions)
- [Type Definitions and Interfaces](#type-definitions-and-interfaces)

---

## Functions

### Exception: Simple Getters/Setters

ONLY these trivial patterns can use single-line comments without sections:

```go
// Name returns the client's name.
func (c *Client) Name() string {
    return c.name
}

// SetName sets the client's name.
func (c *Client) SetName(name string) {
    c.name = name
}
```

**All other functions MUST use the structured format below.**

### Standard Functions (Multi-Line with Sections)

This format is REQUIRED for all functions except simple getters/setters:

```go
// CreateTask inserts a task into the database with retry handling.
//
// This function validates inputs, marshals the payload to JSON,
// and writes the task to the database within a transaction. If a task
// with the same idempotency key exists, behavior depends on replaceExisting.
//
// Parameters:
//   - ctx: context for cancellation and timeouts
//   - db: database handle or transaction (implements DBTX interface)
//   - name: task name identifying which handler should process it
//   - idempotencyKey: unique key to prevent duplicate task insertion
//   - payload: task data, marshaled to JSON (must be JSON-serializable)
//   - replaceExisting: if true, replaces existing task with same idempotency key
//
// Returns:
//   - TaskKey containing the task name and idempotency key
//   - error if validation fails, marshaling fails, or database operation fails
//
// Errors:
//   - ErrInvalidInput: if name or idempotencyKey is empty
//   - ErrMarshalFailed: if payload cannot be marshaled to JSON
//   - ErrDuplicateKey: if idempotencyKey exists and replaceExisting is false
func CreateTask(ctx context.Context, db DBTX, name, idempotencyKey string, payload interface{}, replaceExisting bool) (TaskKey, error) {
```

**Simple function example (still requires sections):**

```go
// New creates a new Client with the given configuration.
//
// Parameters:
//   - config: configuration settings for the client
//
// Returns:
//   - pointer to initialized Client
func New(config Config) *Client {
```

### Required Sections

- High-level description (first 1-2 paragraphs) — ALWAYS REQUIRED
- Parameters section (list each with description) — REQUIRED if function has parameters
- Returns section (describe each return value) — REQUIRED if function has return values
- Errors section (document specific error constants/variables) — REQUIRED if function returns error type

### Critical: Error Documentation

Always document specific error constants or variables that a function might return:

```go
// GetUserByID retrieves a user record from the database.
//
// Parameters:
//   - ctx: request context
//   - db: database connection
//   - userID: UUID of the user to retrieve
//
// Returns:
//   - User struct populated with user data
//   - error if user not found or database error occurs
//
// Errors:
//   - ErrUserNotFound: if no user exists with the given ID
//   - ErrInvalidUUID: if userID is not a valid UUID
func GetUserByID(ctx context.Context, db *sql.DB, userID string) (User, error) {
```

This allows engineers to properly handle specific errors:
```go
user, err := GetUserByID(ctx, db, id)
if err != nil {
    if errors.Is(err, ErrUserNotFound) {
        // Handle missing user
    }
    return err
}
```

---

## Structs

### Struct Comment (Above Definition)

Provide a concise, high-level description:

```go
// Task represents a unit of work stored in the database.
type Task struct {
    ...
}

// Config holds configuration settings for the application.
type Config struct {
    ...
}
```

**Guidelines:**
- Start with struct name
- Use "represents", "provides", "manages", "holds", "wraps" verbs
- Keep to one sentence when possible

### Field Comments (Inline)

Add concise inline comments to the right of each field:

```go
// Task represents a unit of work stored in the database.
type Task struct {
    ID             int             // Database primary key (auto-incremented)
    Name           string          // Logical name identifying the handler for this task
    Payload        json.RawMessage // JSON-encoded task data passed to the handler
    IdempotencyKey string          // Unique key to prevent duplicate processing
    Created        time.Time       // Timestamp when the task was created
    Status         TaskStatus      // Current processing status (queued, processing, completed, failed)
}
```

**Guidelines:**
- Keep brief (ideally under 80 characters)
- Explain the purpose, not just the type
- For status fields, document possible values in parentheses
- For IDs/keys, explain what they identify
- For timestamps, explain what event they represent

---

## Variables and Constants

### Constants (Above-Line Format with Spacing)

For grouped constants, use above-line comments with blank lines between entries:

```go
// TaskStatus represents the current state of a task.
type TaskStatus string

const (
    // StatusQueued indicates a task is waiting to be processed.
    StatusQueued TaskStatus = "queued"

    // StatusProcessing indicates a task is currently being executed.
    StatusProcessing TaskStatus = "processing"

    // StatusCompleted indicates a task finished successfully.
    StatusCompleted TaskStatus = "completed"

    // StatusFailed indicates a task encountered an error.
    StatusFailed TaskStatus = "failed"
)
```

```go
const (
    // MaxRetries is the maximum number of retry attempts before giving up.
    MaxRetries = 3

    // DefaultTimeout is the default timeout for task execution.
    DefaultTimeout = 30 * time.Second

    // CleanupInterval defines how often to clean up old completed tasks.
    CleanupInterval = 1 * time.Hour
)
```

### Variables (Above-Line Format with Spacing)

For variables, place comment above with blank lines between entries:

```go
// ErrTaskNotFound indicates that a task with the given ID does not exist.
var ErrTaskNotFound = errors.New("task not found")

// ErrDuplicateKey indicates a task with the same idempotency key already exists.
var ErrDuplicateKey = errors.New("duplicate idempotency key")
```

```go
var (
    // defaultClient is the singleton HTTP client used across the application.
    defaultClient *http.Client

    // initOnce ensures the default client is initialized only once.
    initOnce sync.Once
)
```

**Guidelines:**
- Explain purpose and meaning, not just value
- For errors: Explain when/why this error occurs
- For all constants/variables (exported and unexported): Always include comment
- For configuration constants: Document units, ranges, or constraints
- For grouped entries: Use above-line comments with blank lines between

---

## Inline Comments (Inside Functions)

Add comments above logical groupings of code:

```go
func ProcessOrder(ctx context.Context, orderID string) error {
    // Validate the order ID format
    if !isValidUUID(orderID) {
        return ErrInvalidOrderID
    }

    // Fetch order details from database
    order, err := db.GetOrder(ctx, orderID)
    if err != nil {
        return fmt.Errorf("fetching order: %w", err)
    }

    // Calculate total including tax and shipping
    subtotal := calculateSubtotal(order.Items)
    tax := subtotal * order.TaxRate
    shipping := calculateShipping(order.Address)
    total := subtotal + tax + shipping

    // Start transaction to ensure atomicity
    tx, err := db.Begin(ctx)
    if err != nil {
        return fmt.Errorf("starting transaction: %w", err)
    }
    defer tx.Rollback()

    // Update order status and total
    if err := tx.UpdateOrder(ctx, orderID, total, StatusProcessing); err != nil {
        return fmt.Errorf("updating order: %w", err)
    }

    // Commit transaction
    if err := tx.Commit(); err != nil {
        return fmt.Errorf("committing transaction: %w", err)
    }

    return nil
}
```

**When to add inline comments:**
- Complex algorithms or calculations
- Business logic implementing specific requirements
- Workarounds or non-obvious solutions
- Operations with side effects (database writes, API calls)
- Error handling with specific recovery strategies
- Logical groupings (validation, data fetching, processing, persistence)

**When NOT to add inline comments:**
- Obvious operations (x := 1, return nil)
- Self-explanatory code
- Simple variable assignments
- Standard library calls with clear names

**Guidelines:**
- Place comment above the code block being explained
- Keep concise (1-2 lines maximum)
- Explain WHY, not WHAT (code shows what)
- Use blank line before comment if starting new logical section

---

## Type Definitions and Interfaces

### Type Aliases

```go
// TaskHandler is a function that processes a task and returns a result.
type TaskHandler func(context.Context, *Task) TaskResult

// DBTX is satisfied by both *sql.DB and *sql.Tx, allowing functions
// to work with either a database connection or transaction.
type DBTX interface {
    ExecContext(ctx context.Context, query string, args ...any) (sql.Result, error)
    QueryContext(ctx context.Context, query string, args ...any) (*sql.Rows, error)
    QueryRowContext(ctx context.Context, query string, args ...any) *sql.Row
}
```

### Interfaces

```go
// Storage defines the interface for persisting and retrieving tasks.
type Storage interface {
    // CreateTask inserts a new task into storage.
    CreateTask(ctx context.Context, task *Task) error

    // GetTask retrieves a task by ID, returning ErrTaskNotFound if not found.
    GetTask(ctx context.Context, id int) (*Task, error)

    // UpdateStatus changes the status of a task.
    UpdateStatus(ctx context.Context, id int, status TaskStatus) error
}
```

**Guidelines:**
- Document the interface purpose above the type definition
- Document each method within the interface
- Mention key errors methods might return
- Keep method comments brief (one line when possible)
