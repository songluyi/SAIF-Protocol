# SAIF Standard Error Model v0.3

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

## Canonical Error Location

When a Standard Error or conformance expectation identifies an input location,
`details.path` uses an [RFC 6901](https://www.rfc-editor.org/rfc/rfc6901.html)
JSON Pointer relative to the complete input
instance evaluated by the operation or schema. The empty string identifies the
root instance. `/` identifies a member whose name is the empty string.

Pointer escaping is mandatory: `~0` represents `~`, and `~1` represents `/`.
An error concerning a missing member uses the pointer that the member would have
occupied. A semantic operation error points to the smallest input value that
caused the primary failure. Bindings MUST preserve this pointer losslessly.

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

## v0.3 Core Error Code Registry

The following codes are reserved by the v0.3 Reference Node profile:

| Code | Category | Meaning |
| --- | --- | --- |
| `SAIF-VALIDATION-0001` | `VALIDATION` | Required field or semantic input missing. |
| `SAIF-VALIDATION-0002` | `VALIDATION` | Value is outside a required enumeration or shape. |
| `SAIF-VERSION-0001` | `VERSION` | Version is unsupported, incompatible, or downgraded. |
| `SAIF-AUTHORIZATION-0001` | `AUTHORIZATION` | Authorization was not verified. |
| `SAIF-AUTHORIZATION-0002` | `AUTHORIZATION` | Request or Authorization revision mismatch. |
| `SAIF-AUTHORIZATION-0003` | `AUTHORIZATION` | Authorization Decision expired. |
| `SAIF-AUTHORIZATION-0004` | `AUTHORIZATION` | Authorization is inactive or revoked. |
| `SAIF-AUTHORIZATION-0005` | `AUTHORIZATION` | Agent, Owner, or actor identity mismatch. |
| `SAIF-AUTHORIZATION-0006` | `AUTHORIZATION` | Conversion used a non-`ALLOW` decision. |
| `SAIF-AUTHORIZATION-0007` | `AUTHORIZATION` | Provider result actor is not authorized. |
| `SAIF-AUTHORIZATION-0008` | `AUTHORIZATION` | Decision replayed against another Request. |
| `SAIF-STATE-0001` | `STATE_TRANSITION` | Lifecycle transition is not allowed. |
| `SAIF-STATE-0002` | `STATE_TRANSITION` | Expected revision is stale or lost a concurrency race. |
| `SAIF-PROVIDER-0001` | `PROVIDER` | Provider execution is unavailable. |
| `SAIF-EXTENSION-0001` | `EXTENSION` | Required extension is unsupported. |
| `SAIF-EXTENSION-0002` | `EXTENSION` | Extension version is incompatible. |
| `SAIF-EXTENSION-0003` | `EXTENSION` | Extension namespace and identifier do not match. |
| `SAIF-EXTENSION-0004` | `EXTENSION` | Manifest digest is invalid or conflicts. |
| `SAIF-EXTENSION-0005` | `EXTENSION` | Required and unknown behavior are inconsistent. |
| `SAIF-EXTENSION-0006` | `EXTENSION` | Extension attempted to weaken a core authorization or security invariant. |
| `SAIF-PROTOCOL-0003` | `PROTOCOL` | Idempotency key was reused with different input. |
| `SAIF-PROTOCOL-0004` | `PROTOCOL` | Required atomic outcome could not complete. |
| `SAIF-PROTOCOL-0005` | `PROTOCOL` | Portable error or audit data disclosed prohibited secrets. |
| `SAIF-PROTOCOL-0006` | `PROTOCOL` | Cross-object reference integrity failed. |
| `SAIF-PROTOCOL-0007` | `PROTOCOL` | Idempotency evidence was lost before the advertised retention expiry. |
| `SAIF-PROTOCOL-0008` | `PROTOCOL` | Required Security Profile conformance evidence is incomplete. |

No other specification or extension may assign a different meaning to these
codes. New core codes require governance review; private codes remain namespaced.

## Example

See [`examples/error-example.json`](../examples/error-example.json).
