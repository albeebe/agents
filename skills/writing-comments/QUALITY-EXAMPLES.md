# Quality Examples and Edge Cases

## Contents
- [Function Comments](#function-comments)
- [Struct Comments](#struct-comments)
- [Constant Comments](#constant-comments)
- [Inline Comments](#inline-comments)
- [Handling Incomplete Comments](#handling-incomplete-comments)
- [Edge Case Handling](#edge-case-handling)

---

## Function Comments

### Simple Getter (Rare Exception)

```go
// Name returns the client's name.
func (c *Client) Name() string {
    return c.name
}
```

### Standard Function with Sections

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

### Complex Function

```go
// ExecuteQuery runs a SQL query against the database with retry logic.
//
// The query is executed with exponential backoff retry on transient errors.
// Context cancellation is respected during execution and retries.
//
// Parameters:
//   - ctx: context for cancellation and timeout
//   - db: database connection or transaction
//   - query: SQL query string with placeholder parameters
//   - args: values to substitute for placeholders
//
// Returns:
//   - sql.Result containing rows affected and last insert ID
//   - error if query fails after all retries
//
// Errors:
//   - ErrQueryTimeout: if context deadline exceeded
//   - ErrConnectionFailed: if database connection cannot be established
func ExecuteQuery(ctx context.Context, db DBTX, query string, args ...interface{}) (sql.Result, error) {
```

### BAD: Missing Parameters and Returns

```go
// ExecuteQuery executes a query
func ExecuteQuery(ctx context.Context, db DBTX, query string, args ...interface{}) (sql.Result, error) {
```

### BAD: Single-Line When Should Have Sections

```go
// New initializes a Client with the provided configuration.
func New(config Config) *Client {
```

---

## Struct Comments

### GOOD

```go
// Request encapsulates an HTTP request with headers and body.
type Request struct {
    Method  string            // HTTP method (GET, POST, PUT, DELETE)
    URL     string            // Target URL for the request
    Headers map[string]string // HTTP headers to include
    Body    []byte            // Request body content
}
```

### BAD: Just Restating Names

```go
// Request is a request
type Request struct {
    Method  string            // Method
    URL     string            // URL
    Headers map[string]string // Headers
    Body    []byte            // Body
}
```

---

## Constant Comments

### GOOD: Above-Line with Spacing

```go
const (
    // MaxRetries is the maximum number of retry attempts before giving up.
    MaxRetries = 3

    // RetryDelay is the initial delay between retry attempts, doubled after each retry.
    RetryDelay = 100 * time.Millisecond
)
```

### GOOD: Status Enum

```go
const (
    // StatusPending indicates a request is waiting to be processed.
    StatusPending Status = "pending"

    // StatusProcessing indicates a request is currently being handled.
    StatusProcessing Status = "processing"

    // StatusCompleted indicates a request finished successfully.
    StatusCompleted Status = "completed"

    // StatusFailed indicates a request encountered an error.
    StatusFailed Status = "failed"
)
```

### BAD: Inline Comments for Constants

```go
const (
    MaxRetries = 3               // Max retries
    RetryDelay = 100 * time.Millisecond // Retry delay
)
```

### BAD: Missing Blank Lines Between Constants

```go
const (
    // MaxRetries is the maximum number of retry attempts.
    MaxRetries = 3
    // RetryDelay is the delay between retries.
    RetryDelay = 100 * time.Millisecond
)
```

---

## Inline Comments

### GOOD: Explains Business Logic

```go
func CalculateDiscount(price float64, customerType string) float64 {
    // Apply volume discount for wholesale customers
    if customerType == "wholesale" && price > 1000 {
        return price * 0.15
    }

    // Apply standard discount for retail customers
    if customerType == "retail" && price > 500 {
        return price * 0.10
    }

    // No discount for small purchases
    return 0
}
```

### BAD: States the Obvious

```go
func CalculateDiscount(price float64, customerType string) float64 {
    // Check if wholesale and price > 1000 (obvious from code)
    if customerType == "wholesale" && price > 1000 {
        return price * 0.15 // Return 15% discount (obvious from calculation)
    }

    return 0
}
```

---

## Handling Incomplete Comments

When existing comments are incomplete (e.g., missing error documentation), improve them:

**Before:**
```go
// GetUser retrieves a user from the database.
func GetUser(ctx context.Context, db *sql.DB, userID string) (User, error) {
```

**After (improved):**
```go
// GetUser retrieves a user from the database.
//
// Parameters:
//   - ctx: request context
//   - db: database connection
//   - userID: UUID of the user to retrieve
//
// Returns:
//   - User struct with user data
//   - error if user not found or database error occurs
//
// Errors:
//   - ErrUserNotFound: if no user exists with the given ID
func GetUser(ctx context.Context, db *sql.DB, userID string) (User, error) {
```

---

## Edge Case Handling

### Files That Don't Exist

- Abort immediately
- Report: *"File not found: {path}. Please check the path and try again."*

### Non-Go Files

- Abort immediately
- Report: *"This skill only comments Go files. The target '{path}' is not a Go file."*

### Already-Commented Code

- Preserve good comments
- Report which elements were preserved:
  - *"Element '{name}' already has a well-written comment. Preserving existing documentation."*
- Only modify if comment is clearly incomplete or non-compliant

### Generated Code

- If file has `// Code generated` comment at top, skip entirely
- Report: *"File '{path}' is generated code. Skipping to avoid modifying auto-generated files."*

### Test Files

- Test files (`_test.go`) can be commented
- Focus on:
  - Exported test helpers
  - Exported benchmark functions
  - Complex test setup functions
- Don't comment every individual test case unless specifically requested
- **Never create test files or add test functions**
