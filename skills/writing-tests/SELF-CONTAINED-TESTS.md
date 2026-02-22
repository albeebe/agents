# Self-Contained Test Patterns

## Contents
- [Test Function Template](#test-function-template)
- [No Global Variables](#no-global-variables)
- [No External Helper Functions](#no-external-helper-functions)
- [Embedded Helpers Pattern](#embedded-helpers-pattern)
- [Coverage Documentation](#coverage-documentation)
- [Subtest Comment Requirements](#subtest-comment-requirements)
- [Logical Block Comments](#logical-block-comments)
- [Test Data Rules](#test-data-rules)

---

## Test Function Template

**CRITICAL: All tests for a function in ONE test function with subtests.**

```go
// TestFunctionName tests the FunctionName function with 100% coverage.
//
// Coverage: 100% (6 test cases covering all branches)
//
// Test cases:
//   - Success: Happy path with valid inputs
//   - ValidationError: Missing required fields
//   - DuplicateEntry: MySQL duplicate key error
//   - DatabaseError: Generic database failure
//   - NotFound: No rows returned
//   - EdgeCase: Boundary condition handling
func TestFunctionName(t *testing.T) {
    // Verify that inserting a valid row succeeds without errors
    t.Run("Success", func(t *testing.T) {
        // Setup mocks
        db, mock, err := sqlmock.New()
        if err != nil {
            t.Fatalf("failed to create mock: %v", err)
        }
        defer db.Close()

        // Define test data
        testData := &SomeRow{
            ID:   uuid.NewString(),
            Name: "Test Name",
        }

        // Set expectations
        mock.ExpectExec("INSERT INTO table_name").
            WithArgs(testData.ID, testData.Name).
            WillReturnResult(sqlmock.NewResult(1, 1))

        // Execute function
        err = FunctionName(context.Background(), db, testData)

        // Verify results
        if err != nil {
            t.Errorf("expected no error, got: %v", err)
        }

        // Verify mock expectations met
        if err := mock.ExpectationsWereMet(); err != nil {
            t.Errorf("unfulfilled expectations: %v", err)
        }
    })

    // Verify that missing required fields returns a validation error
    t.Run("ValidationError", func(t *testing.T) {
        // ... similar structure
    })

    // ... additional test cases
}
```

---

## No Global Variables

```go
// BAD - Global mock
var globalDB *sql.DB
var globalMock sqlmock.Sqlmock

func TestCreateTask(t *testing.T) {
    // Uses global state — tests can interfere with each other
}

// GOOD - Local mock
func TestCreateTask(t *testing.T) {
    db, mock, err := sqlmock.New()
    if err != nil {
        t.Fatalf("failed to create mock: %v", err)
    }
    defer db.Close()
    // Each test creates its own isolated mock
}
```

---

## No External Helper Functions

```go
// BAD - External helper
func setupMockDB(t *testing.T) (*sql.DB, sqlmock.Sqlmock) {
    // Helper outside test function — creates hidden dependencies
}

func TestCreateTask(t *testing.T) {
    db, mock := setupMockDB(t)
    // ...
}

// GOOD - Embedded helper
func TestCreateTask(t *testing.T) {
    setupMock := func() (*sql.DB, sqlmock.Sqlmock) {
        db, mock, err := sqlmock.New()
        if err != nil {
            t.Fatalf("failed to create mock: %v", err)
        }
        return db, mock
    }

    t.Run("Success", func(t *testing.T) {
        db, mock := setupMock()
        defer db.Close()
        // ...
    })
}
```

---

## Embedded Helpers Pattern

Embedded helpers inside test functions are encouraged when they improve clarity:

```go
func TestComplexFunction(t *testing.T) {
    // Embedded helper for setting up test data
    createTestUser := func(id, email string) *User {
        return &User{
            ID:      id,
            Email:   email,
            Created: time.Now(),
        }
    }

    // Embedded helper for asserting results
    assertNoError := func(t *testing.T, err error) {
        t.Helper()
        if err != nil {
            t.Fatalf("unexpected error: %v", err)
        }
    }

    t.Run("Success", func(t *testing.T) {
        user := createTestUser(uuid.NewString(), "test@example.com")
        err := SaveUser(user)
        assertNoError(t, err)
    })
}
```

---

## Coverage Documentation

Each test function MUST have a comment including:

```go
// TestFunctionName tests FunctionName with 100% coverage.
//
// Coverage: 100% (X test cases covering all Y branches)
//
// Test cases:
//   - CaseName1: Description
//   - CaseName2: Description
```

---

## Subtest Comment Requirements

**MANDATORY**: Every `t.Run(...)` block MUST have a concise, plain-English comment directly above it explaining what the test verifies. Write it as a simple, high-level sentence that anyone can understand.

**Format:** `// Verify that [simple description of what is being tested]`

### GOOD Subtest Comments

```go
// Verify that creating a task with valid data succeeds
t.Run("Success", func(t *testing.T) { ... })

// Verify that an empty task name returns a validation error
t.Run("ValidationError_MissingName", func(t *testing.T) { ... })

// Verify that inserting a duplicate task returns a duplicate entry error
t.Run("DuplicateEntry", func(t *testing.T) { ... })

// Verify that a database failure returns an error
t.Run("DatabaseError", func(t *testing.T) { ... })

// Verify that looking up a user that doesn't exist returns false
t.Run("NotFound", func(t *testing.T) { ... })
```

### BAD Subtest Comments

```go
// Too vague
// Test case 1: Success
t.Run("Success", func(t *testing.T) { ... })

// Too technical / implementation-focused
// Tests that ExecContext returns sql.Result with 1 affected row
t.Run("Success", func(t *testing.T) { ... })

// Just repeating the test name
// Validation error
t.Run("ValidationError", func(t *testing.T) { ... })

// Missing entirely
t.Run("Success", func(t *testing.T) { ... })
```

---

## Logical Block Comments

Add concise comments above logical sections within each subtest:

```go
func TestCreateTask(t *testing.T) {
    // Verify that creating a task with valid data succeeds
    t.Run("Success", func(t *testing.T) {
        // Setup mock database
        db, mock, err := sqlmock.New()
        if err != nil {
            t.Fatalf("failed to create mock: %v", err)
        }
        defer db.Close()

        // Create test data
        task := &Task{
            ID:   uuid.NewString(),
            Name: "Test Task",
        }

        // Set database expectations
        mock.ExpectExec("INSERT INTO tasks").
            WithArgs(task.ID, task.Name).
            WillReturnResult(sqlmock.NewResult(1, 1))

        // Execute function under test
        err = CreateTask(context.Background(), db, task)

        // Verify no error returned
        if err != nil {
            t.Errorf("expected no error, got: %v", err)
        }

        // Verify all mock expectations were met
        if err := mock.ExpectationsWereMet(); err != nil {
            t.Errorf("unfulfilled expectations: %v", err)
        }
    })
}
```

---

## Test Data Rules

Use generic, non-personal test data:

```go
// GOOD - Generic data
user := &User{
    ID:      uuid.NewString(),
    Name:    "Test User",
    Email:   "test@example.com",
    Company: "Example Corp",
}

// BAD - Personal/real info
user := &User{
    Name:    "John Smith",
    Email:   "john@acme.com",
    Company: "Acme Inc",
}
```

**Rules:**
- Use generic names: "Test User", "Example Corp", "Sample Product"
- Use UUIDs for IDs: `uuid.NewString()`
- Use example.com for emails/domains
- Use realistic but generic data
