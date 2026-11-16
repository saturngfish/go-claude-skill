---
name: go-service-conventions
description: Team conventions for Go services - code formatting (blank line before return, guard returns, logical blocks), the required library stack, and the project directory layout. Use when writing, editing, reviewing or scaffolding Go service code.
---

# Go service conventions

Apply these rules whenever you write, edit, review or scaffold Go code for a service.

## 1. Code formatting

### Rules

1. **Blank line before `return`** when the `return` follows one or more statements inside the same block.
2. **Guard return** (an early exit that is the only line inside an `if`) is written **without** a blank line. The body of the condition already isolates it.
3. **Logical blocks** inside a function (declaration → computation → return) are separated by blank lines.
4. One-line wrapper functions (`func X() T { return ... }`) are fine and stay on one line.

### Good

```go
func EncodeBattle(id int64) string { return Encode(Battle, id) }
func EncodeItem(id int64) string   { return Encode(Item, id) }

if resp.IsError() {
	if resp.Error != nil && resp.Error.Message == msgNotEnoughFunds {
		return true, nil
	}

	return false, types.NewServiceError(types.ErrorServerError, "WITHDRAW_FAILED")
}

func isUsdStablecoin(symbol string) bool {
	_, ok := usdStablecoins[strings.ToUpper(strings.TrimSpace(symbol))]

	return ok
}

func ConvertUsdToActiveCrypto(priceUsd float64, targetSymbol string) (float64, error) {
	// ...other code
	result, err := prices.Default.Convert(ctx, decimal.NewFromFloat(priceUsd), "USD", targetSymbol)
	if err != nil {
		return 0, err
	}

	f, _ := result.Float64()

	return f, nil
}

func Roll(serverSeed, clientSeed string, nonce int64) uint64 {
	h := hmac.New(sha256.New, []byte(serverSeed))
	h.Write([]byte(clientSeed))

	nonceBytes := make([]byte, 8)
	binary.LittleEndian.PutUint64(nonceBytes, uint64(nonce))
	h.Write(nonceBytes)

	sum := h.Sum(nil)

	return binary.BigEndian.Uint64(sum[:8]) % RollResolution
}
```

### Bad

```go
// No blank lines between logical blocks or before return
func Roll(serverSeed, clientSeed string, nonce int64) uint64 {
	h := hmac.New(sha256.New, []byte(serverSeed))
	h.Write([]byte(clientSeed))
	nonceBytes := make([]byte, 8)
	binary.LittleEndian.PutUint64(nonceBytes, uint64(nonce))
	h.Write(nonceBytes)
	sum := h.Sum(nil)
	return binary.BigEndian.Uint64(sum[:8]) % RollResolution
}

// return directly after another statement, no blank line
if resp.IsError() {
	if resp.Error != nil && resp.Error.Message == msgNotEnoughFunds {
		return true, nil
	}
	return false, types.NewServiceError(types.ErrorServerError, "WITHDRAW_FAILED")
}

func isUsdStablecoin(symbol string) bool {
	_, ok := usdStablecoins[strings.ToUpper(strings.TrimSpace(symbol))]
	return ok
}
```

## 2. Stack

Do not swap these libraries for alternatives unless explicitly asked.

| Purpose | Library |
|---|---|
| Language | Go |
| Database access | GORM |
| Inter-service communication / primary transport (recommended) | NATS |
| Web server (REST) | Fiber v3 |
| Web server (when high performance is required) | Any that fits the task (for example, for SSE) |
| Validation | `github.com/go-playground/validator/v10` |
| AMQP | `github.com/rabbitmq/amqp091-go` |
| Logging | `github.com/rs/zerolog` |
| Decimal numbers | `github.com/shopspring/decimal` |
| Configuration | `github.com/spf13/viper` |
| JWT | `github.com/golang-jwt/jwt/v5` |
| Redis | `github.com/redis/go-redis/v9` |
| Kafka | `github.com/segmentio/kafka-go` |
| UUID | `github.com/google/uuid` |

## 3. Project layout

| Directory | Purpose |
|---|---|
| `cmd` | Service entry points, usually only `main.go` |
| `internal/dto` | Handler request/response structs and everything related to them |
| `internal/transport/nats` | Router + `natsfm.Handle` registration + broadcast/JetStream subscriptions |
| `internal/transport/http` | Health / Fiber |
| `internal/transport/ampq` | AMQP consumers/handlers |
| `internal/services` | Business logic, endpoint functions only |
| `internal/helpers` | Helper functions, including the ones endpoints need |
| `internal/utils` | Technical code: log, ptr, validate, local types, config |
| `internal/repository` | Formerly `queries`: every function directly tied to database calls and model handling |
| `internal/tasks` | Cron / periodic background jobs |
| `internal/clients` | External SDKs; move to `go-lib` once used by more than one service |
| `pkg/models` | GORM models and reusable response models |

### Where code goes

- Endpoint handler → `internal/services`; its DTOs → `internal/dto`.
- Any database query → `internal/repository`, never in services.
- New external SDK → `internal/clients`; once a second service needs it → `go-lib`.
- GORM model → `pkg/models`.

## Review checklist

- [ ] Every `return` that follows other statements in its block has a blank line before it.
- [ ] Guard returns inside `if` have no extra blank lines.
- [ ] Logical blocks in a function are separated by blank lines.
- [ ] Libraries come from the stack table.
- [ ] Files live in the correct directories.
