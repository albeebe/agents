# Mock Patterns

## Contents
- [Database Mocking (go-sqlmock)](#database-mocking-go-sqlmock)
- [HTTP Mocking (httptest)](#http-mocking-httptest)
- [WebSocket Mocking (httptest)](#websocket-mocking-httptest)
- [Good vs Bad Mock Usage](#good-vs-bad-mock-usage)

---

## Database Mocking (go-sqlmock)

### Success Case — INSERT

```go
db, mock, err := sqlmock.New()
if err != nil {
    t.Fatalf("failed to create mock: %v", err)
}
defer db.Close()

mock.ExpectExec("INSERT INTO table").
    WithArgs(arg1, arg2).
    WillReturnResult(sqlmock.NewResult(1, 1))
```

### Query with Rows — SELECT

```go
rows := sqlmock.NewRows([]string{"id", "name"}).
    AddRow("123", "Test")

mock.ExpectQuery("SELECT (.+) FROM table").
    WithArgs("123").
    WillReturnRows(rows)
```

### Generic Database Error

```go
mock.ExpectExec("INSERT INTO table").
    WillReturnError(errors.New("database error"))
```

### MySQL Duplicate Entry (Error 1062)

```go
mock.ExpectExec("INSERT INTO table").
    WillReturnError(&mysql.MySQLError{Number: 1062})
```

### Row Not Found

```go
mock.ExpectQuery("SELECT (.+) FROM table").
    WillReturnError(sql.ErrNoRows)
```

### Transaction Mocking

```go
mock.ExpectBegin()
mock.ExpectExec("INSERT INTO table").
    WithArgs(arg1, arg2).
    WillReturnResult(sqlmock.NewResult(1, 1))
mock.ExpectCommit()
```

### Transaction Rollback

```go
mock.ExpectBegin()
mock.ExpectExec("INSERT INTO table").
    WillReturnError(errors.New("insert failed"))
mock.ExpectRollback()
```

### Verifying All Expectations Met

```go
if err := mock.ExpectationsWereMet(); err != nil {
    t.Errorf("unfulfilled expectations: %v", err)
}
```

---

## HTTP Mocking (httptest)

### Basic HTTP Server

```go
server := httptest.NewServer(http.HandlerFunc(func(w http.ResponseWriter, r *http.Request) {
    // Verify request
    if r.Method != "POST" {
        t.Errorf("expected POST, got %s", r.Method)
    }

    // Return response
    w.WriteHeader(http.StatusOK)
    w.Write([]byte(`{"status":"success"}`))
}))
defer server.Close()

// Use server.URL in test
client := NewClient(server.URL)
```

### HTTP Server with Error Response

```go
server := httptest.NewServer(http.HandlerFunc(func(w http.ResponseWriter, r *http.Request) {
    w.WriteHeader(http.StatusInternalServerError)
    w.Write([]byte(`{"error":"internal error"}`))
}))
defer server.Close()
```

### HTTP Server with Request Validation

```go
server := httptest.NewServer(http.HandlerFunc(func(w http.ResponseWriter, r *http.Request) {
    // Verify headers
    if r.Header.Get("Authorization") == "" {
        w.WriteHeader(http.StatusUnauthorized)
        return
    }

    // Verify body
    body, _ := io.ReadAll(r.Body)
    if len(body) == 0 {
        w.WriteHeader(http.StatusBadRequest)
        return
    }

    w.WriteHeader(http.StatusOK)
}))
defer server.Close()
```

---

## WebSocket Mocking (httptest)

### Basic WebSocket Server

```go
server := httptest.NewServer(http.HandlerFunc(func(w http.ResponseWriter, r *http.Request) {
    upgrader := websocket.Upgrader{}
    conn, err := upgrader.Upgrade(w, r, nil)
    if err != nil {
        t.Fatalf("failed to upgrade: %v", err)
    }
    defer conn.Close()

    // Handle WebSocket messages
    conn.WriteMessage(websocket.TextMessage, []byte("test message"))
}))
defer server.Close()

// Connect to mock server
wsURL := "ws" + strings.TrimPrefix(server.URL, "http")
```

---

## Good vs Bad Mock Usage

### GOOD: go-sqlmock

```go
db, mock, err := sqlmock.New()
if err != nil {
    t.Fatalf("failed to create mock: %v", err)
}
defer db.Close()

mock.ExpectExec("INSERT INTO tasks").
    WithArgs(task.ID, task.Name).
    WillReturnResult(sqlmock.NewResult(1, 1))
```

### GOOD: httptest

```go
server := httptest.NewServer(http.HandlerFunc(func(w http.ResponseWriter, r *http.Request) {
    w.WriteHeader(http.StatusOK)
    w.Write([]byte(`{"status":"ok"}`))
}))
defer server.Close()

client := NewClient(server.URL)
```

### BAD: Real Database

```go
db, err := sql.Open("mysql", "user:pass@tcp(localhost:3306)/testdb")
// Never use real databases in unit tests
```

### BAD: Real HTTP Server

```go
resp, err := http.Get("https://api.example.com/users")
// Never make real HTTP calls in unit tests
```
