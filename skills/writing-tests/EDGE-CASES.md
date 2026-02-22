# Edge Cases and Error Handling

## Contents
- [Handle Files That Don't Exist](#handle-files-that-dont-exist)
- [Handle Non-Go Files](#handle-non-go-files)
- [Handle Existing Test Files](#handle-existing-test-files)
- [Handle Generated Code](#handle-generated-code)
- [Handle Code That Cannot Reach 100%](#handle-code-that-cannot-reach-100)
- [Handle Multiple Files Request](#handle-multiple-files-request)

---

## Handle Files That Don't Exist

```
FILE NOT FOUND

Target: {file_path}

The file you specified does not exist. Please check the path and try again.

Valid paths should be absolute or relative to repository root:
  - /services/api/handler.go
  - ./pkg/tasks/processor.go

I cannot write tests for a file that doesn't exist.
```

---

## Handle Non-Go Files

```
INVALID FILE TYPE

Target: {file_path}

This skill only writes tests for Go files (.go extension).
The target file is: {extension}

I can only test Go code. Please specify a .go file.
```

---

## Handle Existing Test Files

**If test file already exists:**

**CRITICAL: Do NOT assume existing tests are correct. Always verify and fix if needed.**

### Workflow

1. **Read existing test file**

2. **Verify current coverage**
   ```bash
   go test -coverprofile=coverage.out ./{test_file}.go ./{source_file}.go
   go tool cover -func=coverage.out
   ```

3. **Analyze existing tests for violations:**
   - [ ] Check for global variables (outside test functions)
   - [ ] Check for external helper functions (not embedded in test functions)
   - [ ] Check for multiple test functions per target function
   - [ ] Check for missing coverage documentation
   - [ ] Check for missing or vague subtest comments (every t.Run must have a plain-English comment)
   - [ ] Check for proper mocks (go-sqlmock, httptest)
   - [ ] Check for real database/server connections
   - [ ] Verify 100% coverage

4. **Report findings:**

   **If existing tests are compliant and have 100% coverage:**
   ```
   EXISTING TESTS ARE COMPLIANT

   File: {test_file_path}
   Current coverage: 100%

   Verification:
     Self-contained (no globals or external helpers)
     Proper mocks used
     Coverage documented
     100% line and branch coverage

   No changes needed. Existing tests meet all requirements.
   ```

   **If existing tests have violations or <100% coverage:**
   ```
   EXISTING TESTS NEED UPDATES

   File: {test_file_path}
   Current coverage: {percentage}%

   Issues found:
     Global variable 'mockDB' declared outside test functions (line 15)
     External helper 'setupMocks()' not embedded in test function (line 45)
     Multiple test functions for CreateTask (should be one with subtests)
     Missing coverage documentation in test comments
     Coverage below 100% - missing branches:
         - CreateTask line 67: duplicate entry error not tested
         - GetTask line 89: scan error not tested

   I will update the tests to fix these issues and achieve 100% coverage.
   ```

5. **Fix existing tests:**
   - Consolidate multiple test functions into single functions with subtests
   - Remove global variables and move into test functions
   - Convert external helpers to embedded helpers
   - Add missing test cases to reach 100% coverage
   - Add coverage documentation
   - Ensure proper mocks are used

6. **Report updates made:**
   ```
   TESTS UPDATED

   Changes made:
     - Consolidated 4 test functions into TestCreateTask with 6 subtests
     - Removed global variable mockDB, now created in each subtest
     - Embedded setupMocks() helper inside TestCreateTask
     - Added 2 missing test cases for error paths
     - Added coverage documentation (now 100%)

   Before:
     - 4 separate test functions
     - Global variables used
     - External helpers
     - Coverage: 78%

   After:
     - 1 test function with 6 subtests
     - Self-contained (no globals)
     - Embedded helpers
     - Coverage: 100%

   All violations fixed. Tests now meet all requirements.
   ```

---

## Handle Generated Code

```
GENERATED CODE DETECTED

File: {file_path}

This file contains a "// Code generated" comment, indicating it's auto-generated.

Testing generated code is typically not recommended because:
  - Tests may be overwritten when code is regenerated
  - Generated code is usually tested via the generator
  - Manual tests create maintenance burden

I cannot write tests for generated code unless you explicitly confirm you want this.
```

---

## Handle Code That Cannot Reach 100%

### Example 1: Hard-Coded External Dependency

```
CANNOT ACHIEVE 100% COVERAGE

Function: FetchUserData
Issue: Hard-coded HTTP client with no injection point

Line 45: resp, err := http.Get("https://api.example.com/users")

This direct call to http.Get cannot be mocked in unit tests.

To make this testable, refactor to:
  type HTTPClient interface {
      Get(url string) (*http.Response, error)
  }

  func FetchUserData(client HTTPClient, userID string) error {
      resp, err := client.Get(fmt.Sprintf("https://api.example.com/users/%s", userID))
      // ...
  }

Then use httptest to create a mock server in tests.

I cannot write tests that achieve 100% coverage without refactoring.
```

### Example 2: Non-Deterministic Behavior

```
CANNOT ACHIEVE 100% COVERAGE

Function: CreateTimestampedRecord
Issue: Uses time.Now() directly with no injection point

Line 23: record.Created = time.Now()

Direct calls to time.Now() make tests non-deterministic and prevent
testing specific timestamp values.

To make this testable, refactor to:
  func CreateTimestampedRecord(now func() time.Time, data string) error {
      record.Created = now()
      // ...
  }

Then inject a mock time function in tests:
  mockNow := func() time.Time {
      return time.Date(2025, 1, 1, 0, 0, 0, 0, time.UTC)
  }

I cannot write tests that achieve 100% coverage without refactoring.
```

### Example 3: Hard-Coded Timeouts

```
CANNOT ACHIEVE 100% COVERAGE

Function: WaitForResponse
Issue: Hard-coded timeout that cannot be overridden

Line 15: ctx, cancel := context.WithTimeout(context.Background(), 5*time.Second)
Line 23: time.Sleep(2 * time.Second)

Tests would be forced to wait for these hard-coded time periods:
  - 5 second timeout on line 15
  - 2 second sleep on line 23

This makes tests slow and unreliable. A full test suite covering all branches
would take multiple seconds to run, which is unacceptable for unit tests.

To make this testable, refactor to accept timeout as parameter:
  func WaitForResponse(timeout time.Duration) error {
      ctx, cancel := context.WithTimeout(context.Background(), timeout)
      defer cancel()
      // ...
  }

  // Or better: accept context from caller
  func WaitForResponse(ctx context.Context) error {
      // Caller controls timeout
  }

I cannot write tests that would force waiting for hard-coded time periods.
Please refactor to make timeouts configurable.
```

---

## Handle Multiple Files Request

```
MULTIPLE FILES REQUESTED

You've asked me to test {count} files:
  - {file1}
  - {file2}
  - {file3}
  ...

This will generate {count} test files with comprehensive coverage.
Estimated test cases: ~{estimate}

Confirm to proceed, or specify a single file to test first.
```

Wait for confirmation before proceeding.
