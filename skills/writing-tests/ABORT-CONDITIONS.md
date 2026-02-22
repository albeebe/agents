# Abort Conditions

## Contents
- [Condition 1: Hard-Coded Timeouts or Sleeps](#condition-1-hard-coded-timeouts-or-sleeps)
- [Condition 2: Cannot Achieve 100% Coverage](#condition-2-cannot-achieve-100-coverage)
- [Condition 3: Package Mock Required](#condition-3-package-mock-required)
- [Condition 4: Code Too Tightly Coupled](#condition-4-code-too-tightly-coupled)
- [Coverage Gap Identified](#coverage-gap-identified)
- [Coverage Below 100% After Writing Tests](#coverage-below-100-after-writing-tests)

---

## Condition 1: Hard-Coded Timeouts or Sleeps

If the code contains hard-coded sleeps or timeouts that cannot be overridden, STOP IMMEDIATELY.

**Abort template:**
```
TESTS WOULD REQUIRE WAITING

Function: RetryWithBackoff
Issue: Hard-coded sleeps and timeouts that cannot be overridden

Lines requiring wait time:
  - Line 15: time.Sleep(5 * time.Second) - 5 second wait
  - Line 23: time.Sleep(10 * time.Second) - 10 second wait
  - Line 34: context.WithTimeout(ctx, 30 * time.Second) - 30 second timeout

Testing all error paths would require waiting:
  - Success path: 0 seconds
  - First retry: 5 seconds
  - Second retry: 15 seconds (5 + 10)
  - Timeout test: 30 seconds
  - Total test time: ~50+ seconds just for this function

Unit tests should run in milliseconds, not minutes.

To make this testable, refactor to accept configurable timeouts:
  func RetryWithBackoff(ctx context.Context, retryDelays []time.Duration, operation func() error) error {
      for _, delay := range retryDelays {
          if err := operation(); err == nil {
              return nil
          }
          time.Sleep(delay)
      }
      return errors.New("all retries failed")
  }

Then in tests:
  // Use millisecond delays instead of seconds
  delays := []time.Duration{1*time.Millisecond, 2*time.Millisecond}
  err := RetryWithBackoff(ctx, delays, mockOperation)

Or better, accept a Timer interface:
  type Timer interface {
      Sleep(time.Duration)
      After(time.Duration) <-chan time.Time
  }

  func RetryWithBackoff(timer Timer, delays []time.Duration, operation func() error) error {
      for _, delay := range delays {
          if err := operation(); err == nil {
              return nil
          }
          timer.Sleep(delay)
      }
      return errors.New("all retries failed")
  }

Then use a mock timer in tests that doesn't actually wait.

I cannot write tests that require waiting for hard-coded time periods.
Please refactor to make timeouts and delays configurable.
```

---

## Condition 2: Cannot Achieve 100% Coverage

If the code has untestable paths due to hard-coded dependencies, STOP IMMEDIATELY.

**Abort template — hard-coded external dependency:**
```
CANNOT ACHIEVE 100% COVERAGE

Function: ProcessPayment
Issue: Hard-coded call to external payment API with no interface

Lines that cannot be tested:
  - Line 45-52: Direct HTTP call to payment.stripe.com
  - Line 67: time.Now() used for timestamp (non-deterministic)

To achieve 100% coverage, this code needs refactoring:
  1. Extract payment API calls to an interface (PaymentService)
  2. Accept PaymentService as parameter or struct field
  3. Accept time function as parameter (or use clock interface)

Suggested refactoring:
  type PaymentService interface {
      Charge(amount int, token string) error
  }

  func ProcessPayment(svc PaymentService, now func() time.Time, amount int) error {
      // ... testable code
  }

I cannot write tests for this code in its current state. Please refactor first.
```

**Abort template — non-deterministic behavior:**
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

**Abort template — hard-coded timeouts (variant):**
```
CANNOT ACHIEVE 100% COVERAGE

Function: WaitForResponse
Issue: Hard-coded timeout that cannot be overridden

Line 15: ctx, cancel := context.WithTimeout(context.Background(), 5*time.Second)
Line 23: time.Sleep(2 * time.Second)

Tests would be forced to wait for these hard-coded time periods:
  - 5 second timeout on line 15
  - 2 second sleep on line 23

This makes tests slow and unreliable.

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

For sleep statements, refactor to use a timer interface:
  type Timer interface {
      Sleep(duration time.Duration)
  }

  func WaitForResponse(timer Timer, retryDelay time.Duration) error {
      timer.Sleep(retryDelay)
      // ...
  }

Then in tests:
  mockTimer := &MockTimer{} // Implements Sleep as no-op
  err := WaitForResponse(mockTimer, 1*time.Millisecond)

I cannot write tests that would force waiting for hard-coded time periods.
Please refactor to make timeouts configurable.
```

---

## Condition 3: Package Mock Required

If the function calls other functions in the same package that have complex logic and would benefit from mocking, STOP IMMEDIATELY.

**Abort template:**
```
PACKAGE MOCK REQUIRED

Function: ValidateUser
Issue: Calls other functions in the same package that have complex logic

The function calls:
  - checkEmailFormat() (line 23)
  - verifyDomain() (line 28)
  - lookupMXRecords() (line 35)

Testing this function properly requires mocking these internal package functions,
which would require creating a mock of the package itself.

This indicates the code should be refactored to better separate concerns:
  1. Extract email validation to separate package
  2. Create EmailValidator interface
  3. Accept validator as dependency

I cannot write effective tests without either:
  a) Mocking the package (not recommended)
  b) Refactoring the code structure

Please refactor or confirm you want tests with mocked package dependencies.
```

---

## Condition 4: Code Too Tightly Coupled

If the function directly opens connections, writes files, or uses global state, STOP IMMEDIATELY.

**Abort template:**
```
CODE REFACTORING RECOMMENDED

Function: SaveOrder
Issue: Tightly coupled to database connection and filesystem

The function:
  - Opens database connection directly (line 15)
  - Writes files directly to disk (line 34)
  - Logs to global logger (line 45)

For proper testing, this should be refactored to:
  1. Accept DBTX interface instead of opening connection
  2. Accept io.Writer or FileSystem interface
  3. Accept logger interface

Current structure makes 100% coverage very difficult without:
  - Real database
  - Real filesystem
  - Integration tests (not unit tests)

Recommended: Refactor to inject dependencies before writing tests.
```

---

## Coverage Gap Identified

When coverage analysis during Step 3 shows that some paths cannot be covered:

**Abort template:**
```
COVERAGE GAP IDENTIFIED

Function: ProcessOrder
Analysis shows the following paths cannot be covered:
  - Line 78: Panic recovery (cannot be tested in unit tests)
  - Line 92: System exit call (os.Exit cannot be mocked)

Coverage achievable: 87%

I cannot write tests that achieve 100% coverage for this function.

Options:
  1. Refactor to remove panic recovery and os.Exit
  2. Accept <100% coverage (you must explicitly request this)
  3. Cancel operation

What would you like to do?
```

---

## Coverage Below 100% After Writing Tests

When tests are written but coverage verification shows <100%:

**Abort template:**
```
COVERAGE BELOW 100%

Target: {source_file}.go
Current coverage: {percentage}%

Missing coverage:
  - Function: FunctionName
    - Line 52-54: Error handling branch not covered
    - Line 67: Early return not covered

  - Function: OtherFunction
    - Line 89-91: Edge case not covered

I identified missing coverage after writing tests. To achieve 100%:
  1. I can add test cases for the missing branches
  2. You can review the coverage report: coverage.out
  3. Cancel if you want <100% coverage (must explicitly request)

What would you like to do?
```
