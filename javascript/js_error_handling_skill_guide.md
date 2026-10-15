# JavaScript Native — Robust Error Handling & Diagnostics

## Error Handling Decision Matrix

| Goal | Tool | Why |
| :--- | :--- | :--- |
| Preserving original stack trace | `new Error(msg, { cause: err })` | Chains the underlying root cause onto the new high-level error without losing diagnostics. |
| Domain-specific error handling | Custom `class extends Error` | Allows granular `instanceof` catching for specific operational failures (e.g. `NotFoundError`). |
| Normalizing unknown exceptions | Safe Error Normalizer function | Guarantees non-Error throwables (strings, numbers, plain objects) become standard `Error` objects. |
| Catching unhandled async errors | `unhandledrejection` listener | Catches lost promises globally in browsers and Node.js before they crash the process. |
| Cleanup on both success and failure | `try ... finally` | Guarantees resource release (locks, file handles, loaders) regardless of whether an error was thrown. |

---

## Error Causation (`Error.cause`)

When catching a low-level network or database error and wrapping it in a higher-level domain error, never concatenate strings (`"Failed: " + err.message`). String concatenation destroys the original stack trace and error type.

Use the native `{ cause }` option:

```javascript
async function loadUserProfile(userId) {
  try {
    const res = await fetch(`/api/users/${userId}`);
    if (!res.ok) throw new Error(`HTTP ${res.status}`);
    return await res.json();
  } catch (err) {
    // Preserves the network/fetch stack trace inside .cause!
    throw new Error(`Failed to load profile for user ${userId}`, { cause: err });
  }
}

// When debugging or logging:
try {
  await loadUserProfile(42);
} catch (err) {
  console.error(err.message); // High-level explanation
  console.error(err.cause);   // Original low-level exception with full stack trace!
}
```

---

## Custom Error Hierarchies

Structure clear error classes for actionable error recovery:

```javascript
class DomainError extends Error {
  constructor(message, options) {
    super(message, options);
    this.name = this.constructor.name;
  }
}

class ValidationError extends DomainError {
  constructor(message, fields = [], options) {
    super(message, options);
    this.fields = fields;
  }
}

class NotFoundError extends DomainError {
  constructor(resource, id, options) {
    super(`${resource} with ID '${id}' was not found.`, options);
    this.resource = resource;
    this.id = id;
  }
}

// Consuming with clean instanceof branch logic
try {
  throw new NotFoundError('Invoice', 'inv_998');
} catch (err) {
  if (err instanceof NotFoundError) {
    // Send 404
  } else if (err instanceof ValidationError) {
    // Send 422
  } else {
    // Send 500
  }
}
```

---

## Normalizing Unknown Exceptions

In JavaScript, anything can be thrown: `throw "oops"`, `throw null`, or `throw 404`. Any resilient catch block or utility must normalize unknown throwables into proper `Error` instances:

```javascript
function toError(maybeError) {
  if (maybeError instanceof Error) return maybeError;
  
  if (typeof maybeError === 'string') {
    return new Error(maybeError);
  }

  try {
    return new Error(JSON.stringify(maybeError));
  } catch {
    return new Error(String(maybeError));
  }
}

// Usage in async wrappers:
try {
  await riskyThirdPartyLibrary();
} catch (rawError) {
  const err = toError(rawError);
  console.error('Safely logged:', err.message, err.stack);
}
```

---

## Global Async Error Boundaries

Ensure unexpected unhandled promise rejections are logged rather than dying silently:

### Browser Environment
```javascript
window.addEventListener('unhandledrejection', (event) => {
  console.error('Unhandled Promise Rejection:', event.reason);
  // Report to telemetry (Sentry, Datadog, etc.)
  // event.preventDefault(); // Prevents default browser console error if handled
});
```

### Node.js Environment
```javascript
process.on('unhandledRejection', (reason, promise) => {
  console.error('Unhandled Rejection at:', promise, 'reason:', reason);
  // In production, log and perform graceful shutdown if necessary
});
```

---

## Do Not

| BAD | GOOD | Why |
| :--- | :--- | :--- |
| `throw new Error("Msg: " + err.message)` | `throw new Error("Msg", { cause: err })` | String concatenation discards the underlying call stack, making root cause debugging impossible. |
| `throw "Something went wrong"` | `throw new Error("Something went wrong")` | Strings have no `.stack` trace or `.name` metadata. |
| Empty catch block: `catch (e) {}` | Log, handle, or rethrow with context | Silent catch blocks mask critical defects and cause bizarre downstream failures. |
| Catching without checking error type | Check `err instanceof SpecificError` | Generic catches accidentally swallow unexpected syntax errors or null reference bugs. |
| Manual cleanup only in the `try` block | Put cleanup in `finally { ... }` | If code throws before reaching cleanup, locks or loading indicators remain stuck indefinitely. |
