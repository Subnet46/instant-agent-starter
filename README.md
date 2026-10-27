# Instant agent starter

Connect an agent to your Instant account through a browser sign-in and a revocable permission grant. Your password, API keys and OAuth tokens do not belong in an agent prompt.

**Status — October 26, 2026:** this repository is a connection contract and agent instruction template. A publicly deployed OAuth/MCP service and supported client have not yet passed the acceptance checks below. There is no verified public MCP or authorization URL in this release. Do not infer one from the Instant website or use the browser account API as an MCP endpoint.

## Intended connection

1. Give your agent the [connection prompt](prompts/connect-instant.md).
2. A supported MCP client discovers the published Instant service and opens its verified authorization page.
3. Sign in to Instant in your browser. The account key manager identifies the requesting client, your account, the requested permissions and which keys the connection may manage. Approve or cancel there.
4. The client completes OAuth authorization with PKCE and stores its credentials securely. The agent receives a connection, not credentials pasted into its conversation.
5. Inspect or revoke the connection in the account key manager. Revocation must stop subsequent operations even if an access token has not expired.

OAuth still gives the client a credential to protect. The client must identify its storage method: an operating-system credential store, an explicitly configured vault, or a session-only connection held in memory. Do not assume an environment file or vault exists, and do not silently persist credentials in plaintext. Doppler is an optional, explicitly configured storage provider, not a prerequisite for using Instant.

For an agent on a remote VM, the browser callback must actually reach that client. A callback to `127.0.0.1` on a laptop does not reach a separate VM. The published supported-client instructions must explain a verified cross-device path before claiming that flow works.

This journey is the release target, not a claim that it is already available. The repository must publish the verified MCP resource URL, environment, supported client/version and acceptance evidence before offering a working Connect link.

## What the connection may do

The first release targets account-owned key management. Each operation is limited by the account owner's grant. Listing key metadata, creating a key, changing its label, revoking it, using it for inference and exporting its secret are distinct capabilities. An initial connection must not silently grant all of them.

Account history and memory access is a separate planned capability. Its intended grant covers all history and memories in the approving account, with decryption on an explicitly trusted device. It requires encrypted storage, device enrollment and recovery work beyond key management.

The [OAuth/MCP contract](docs/oauth-mcp-contract.md) defines the boundaries and release checks. The [agent instructions](AGENTS.md) apply to clients consuming this starter.

## Privacy and usage

Current inference processes prompts and responses in plaintext at the serving systems. TLS and encrypted storage do not establish that the operator cannot access that content. No confidential-computing or independently verified operator-blind inference guarantee is made here.

An agent running on a trusted device can see the content it decrypts. A model or service to which that agent sends content can also see it. Revocation cannot erase previously downloaded plaintext.

A login or key-management check is not a paid inference test. Before inference, establish the requested work and an explicit request, token or spending cap. Preserve actual reported usage; do not substitute estimates for charges or missing usage.

## Release acceptance

- A fresh supported client completes browser sign-in, account-specific consent and a metadata-only key operation without exposing credentials in its transcript.
- Cancellation, wrong account, wrong audience, missing scope, expired token, replayed code and a revoked grant fail before side effects.
- Key creation is idempotent and cannot return a raw secret in ordinary tool text. Concurrent retries cannot create duplicate keys.
- The owner can identify and revoke each connection. Every application worker enforces that revocation.
- Published URLs, environments and client versions are verified. Local or simulated checks are labeled accordingly.

This document does not authorize production deployment, financial transactions, unrestricted inference or automatic key rotation.
