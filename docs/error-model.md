# SAIF Standard Error Model v0.2

## Purpose

The SAIF Standard Error Model allows independent implementations to report failures with consistent meaning while remaining independent of programming language, transport, AI model, and Provider technology.

An error describes a protocol-level failure. It must not expose credentials, private prompts, internal stack traces, personal data, or confidential Provider information.

## Error Object

| Field | Type | Required | Description |
| --- | --- | --- | --- |
| `id` | string | yes | Unique error event identifier. |
| `type` | string | yes | Stable error type, such as `STATE_TRANSITION_ERROR`. |
| `created_at` | string | yes | RFC 3339 date-time when the error was created. |
| `metadata` | object | yes | Non-normative extension data. |
| `saif_version` | string | yes | SAIF version used to interpret the error. |
| `code` | string | yes | Stable machine-readable error code. |
| `category` | string | yes | Standard error category. |
| `message` | string | yes | Safe human-readable explanation. |
| `retryable` | boolean | yes | Whether an unchanged operation may be retried later. |
| `related_object` | object | no | Type and ID of the related SAIF object. |
| `details` | object | yes | Structured, non-sensitive diagnostic detail. |
| `correlation_id` | string | no | Identifier connecting related protocol activity. |

## Error Categories

| Category | Meaning |
| --- | --- |
| `VALIDATION` | An object does not conform to its schema or required semantics. |
| `VERSION` | The declared SAIF version is missing, unsupported, or incompatible. |
| `AUTHORIZATION` | Required authorization is missing, invalid, or insufficient. |
| `STATE_TRANSITION` | A requested lifecycle transition is not permitted. |
| `PROVIDER` | A Provider could not accept or execute the requested operation. |
| `SETTLEMENT` | A reconciliation or settlement representation failed. |
| `EXTENSION` | An extension is unknown, invalid, or incompatible. |
| `PROTOCOL` | Another core protocol rule was violated. |

## Error Code Format

Core error codes use:

```text
SAIF-{CATEGORY}-{NUMBER}
```

Examples:

- `SAIF-VALIDATION-0001` — required field missing;
- `SAIF-VERSION-0001` — unsupported SAIF version;
- `SAIF-AUTHORIZATION-0001` — authorization not verified;
- `SAIF-STATE-0001` — invalid state transition; and
- `SAIF-PROVIDER-0001` — Provider execution unavailable.

The textual category in an error object remains normative even when the shorter `STATE` token is used in a code.

## Retry Semantics

`retryable: true` means the operation may succeed later without changing its business intent, for example after temporary Provider unavailability.

`retryable: false` means the caller must change the object, authorization, state, or version before retrying. It does not prevent a human or organization from creating a new corrected Request.

## Related Object

When an error concerns a specific SAIF object, `related_object` should contain:

```json
{
  "type": "Request",
  "id": "req_001"
}
```

The reference supports audit correlation without embedding the full object in every error.

## Transport Independence

HTTP status codes, MCP tool errors, message acknowledgements, and language exceptions may carry a SAIF error, but they do not replace it. A transport-specific success response must not be interpreted as a successful Order, Execution, or Settlement unless the corresponding SAIF object confirms that outcome.

## Extension Errors

Private extensions must use namespaced codes and must not allocate values in the core `SAIF-` range. A proposal may request a standard code range through the [Extension Proposal Process](proposal-process.md).

## Example

See [`examples/error-example.json`](../examples/error-example.json).
