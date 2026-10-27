# OAuth/MCP connection contract

Draft for implementation review, October 26, 2026. This is a product security contract, not an endpoint inventory or implementation claim. Exact scope strings, client registration and production URLs remain release configuration.

## Trust and account ownership

The browser authenticates the person to Instant. An authorization service presents client-specific consent and issues delegated credentials. The MCP service verifies those credentials and enforces account permissions. The client stores credentials outside model-visible text. The platform administrator credential remains a server-side service credential.

The existing portal session is not an OAuth access token. A native inference key is not an MCP token. Use the official SDK's resource-server boundary with a reviewed authorization implementation; its bearer-token middleware does not provide a complete authorization server. [Official SDK server authorization](https://ts.sdk.modelcontextprotocol.io/v2/serving/authorization).

An authenticated grant binds the account, client, permitted operations and applicable key set. Resolve the account from verified authentication, never from a tool-supplied account ID. Apply ownership and grant checks to every object and side effect. An agent cannot enlarge its own grant.

## Login and consent

Publish a verified HTTPS MCP resource URL per environment. Implement protected-resource metadata, its authorization-server references and authorization-server discovery. A client follows the authentication challenge and validates the discovered issuer; it must not send credentials to arbitrary metadata URLs. [Official discovery requirements](https://modelcontextprotocol.io/specification/2026-07-28/basic/authorization/authorization-server-discovery).

Use an explicitly supported client registration mechanism. Current MCP supports pre-registration and Client ID Metadata Documents; Dynamic Client Registration is retained for compatibility and deprecated for new integrations. Do not advertise registration mechanisms that are not implemented. A public native client has no securely embedded client secret. [Official client registration](https://modelcontextprotocol.io/specification/2026-07-28/basic/authorization/client-registration).

The client starts each connection with fresh PKCE S256 and CSRF state, retains the expected issuer, validates the callback and stores credentials securely. The callback belongs to that client; neither the agent prompt nor a generic static login link can replace this flow. [Official SDK client OAuth](https://ts.sdk.modelcontextprotocol.io/v2/clients/oauth.html).

The authorization page shows the signed-in account, requesting client, requested operations, affected keys and revocation control. Login alone does not approve a client. Consent and denial require CSRF protection; registered callback validation must reject open redirects. Authorization codes are short-lived, single-use and bound to the client, redirect, resource and PKCE challenge. [Official authorization security](https://modelcontextprotocol.io/specification/2026-07-28/basic/authorization/security-considerations).

Authorization and token requests identify the intended MCP resource. Tokens are short-lived, audience-bound and sent in the Authorization header. Reject invalid tokens with 401 and insufficient permissions with 403. Validate issuer, audience, lifetime and grant state before dispatch; an authenticated transport/state handle is not account authorization. Refresh tokens, if enabled, need secure storage, rotation/reuse detection and revocation. [Official MCP authorization](https://modelcontextprotocol.io/specification/2026-07-28/basic/authorization).

## Initial capability boundaries

The following are proposed permission categories, not published OAuth scope names:

| Permission | Permitted effect | Required boundary |
| --- | --- | --- |
| Account/key metadata | Read approved account and key metadata | No secret or other account's objects |
| Key create | Create a project key in the approving account | Explicit consent, durable operation ID, quota and recoverable issuance |
| Key label | Rename an approved key | Account ownership and allowed key set |
| Key revoke | Revoke an approved key | Explicit lifecycle grant; reject unrelated/default-key changes outside it |
| Inference use | Use an approved key for the requested work | Separate authorization and account usage limits |
| Secret export | Deliver a key to protected client credential storage | Disabled until a reviewed delivery mechanism exists; never ordinary tool text |
| History/memory access | Read/write all such data in the approving account | Separate consent plus enrolled trusted-device encryption; not part of initial key management |

For the first release, choose and publish exact scope names and the supported client before claiming interoperability. Key creation may return an operation/key identifier and metadata; raw-secret delivery must use a protected non-model channel or remain unavailable. Do not expose administrator credentials or pass an MCP token to native inference. [Official token and proxy security guidance](https://modelcontextprotocol.io/docs/2026-07-28/tutorials/security/security_best_practices).

## Revocation, integrity and scale

Store connection grants, expiry, revocation, idempotency and account limits durably. Revoking a connection must stop new authorized operations on all workers, including requests with an otherwise unexpired token. Scope/account checks must happen before admission and again at the commit boundary for a pending mutation. Define how already accepted inference and in-flight mutations finish; do not promise retroactive cancellation.

Separate connection revocation from key revocation. If a native key has been exported, revoking only OAuth does not invalidate that key. The account manager must show this distinction and offer explicit revocation of connection-owned credentials. Never revoke shared keys implicitly.

Use versioned writes and transactionally enforced operation IDs for key management and future memory writes. Concurrent retries must replay the original result or return an explicit conflict. Do not hold a database transaction across a platform request. A failure between platform issuance and local persistence needs reconciliation without duplicate issuance or secret leakage.

Use per-account and per-client concurrency, queue, payload and operation limits. Edge rate limits complement these controls; they cannot enforce paid balances or replace account authorization. Do not claim numerical capacity before measured load and multi-worker evidence.

## Privacy boundaries

The intended encrypted history/memory store holds ciphertext and device-specific key envelopes; the trusted device decrypts. Enrollment, recovery and browser-history import are separate release gates. Revocation cannot erase retained plaintext or data keys. Future records need fresh encryption keys after device revocation; rewrapping an unchanged key cannot remove access already obtained.

Inference still requires processing content. The current design does not prove that operators cannot inspect prompts or responses. TLS, a no-log setting, a WAF and server-side encryption are not evidence of operator-blind computation. Any stronger claim requires an independently reviewed architecture and end-to-end attestation evidence, not just a repository statement.

Keep request bodies, responses, credentials and decrypted memories out of operational logging and traces. Audit only necessary actor/grant identifiers, operation IDs, timestamps, usage and outcomes under a documented retention policy. Metadata and model-visible content have different privacy boundaries.

## Evidence before calling the flow live

Record the service revision, environment, client/version and date for each check:

1. Fresh-client discovery, browser login, named-client consent, callback and a permitted metadata-only operation.
2. Cancellation without grant; code replay, wrong verifier, changed callback, issuer mismatch and wrong audience rejected.
3. Cross-account object access and missing-scope mutations rejected without side effects.
4. Grant revocation enforced on two workers while an old token remains otherwise valid.
5. Repeated and concurrent key-create operations issue at most one key and never expose credentials in tool output, errors, logs or browser history.
6. Scope escalation needs consent; account switching cannot reuse another account's grant.
7. One authorized bounded inference and authoritative usage reconciliation, separately from key-management checks, before funded use.
8. History/memory synchronization and privacy guarantees remain unavailable until their distinct acceptance tests pass.

The published release must identify which checks passed and which remain local or unverified. A copied prompt, login page or successful token exchange alone is insufficient evidence.
