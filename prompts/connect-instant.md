# Connect my agent to Instant

Read this repository's README.md, AGENTS.md and OAuth/MCP contract. Connect this agent to my Instant account using the published supported MCP client and browser OAuth flow.

First check the release status and verified MCP URL for my intended environment. If the service is not published or compatible with this client, explain the exact missing prerequisite. Do not invent endpoints or ask me to paste a key as a substitute for this flow.

When available, start the client's authorization-code flow with PKCE and open its validated Instant authorization link. I will sign in and approve this client's requested permissions in the Instant account key manager. Let the client handle the callback and securely store credentials. Never put a password, login code, API key or OAuth token in this prompt or our conversation.

Identify this client's credential storage before connecting. Do not assume I have an environment file or vault. Prefer its supported secure credential store; use an explicitly configured vault such as Doppler only if needed. If secure persistence is unavailable, offer a session-only connection rather than writing credentials to plaintext files. If this agent runs on a VM while I use a browser on another machine, verify the supported callback path first; laptop localhost is not VM localhost.

After consent, show the connected account, environment and granted operations using nonsecret metadata. Start with a permitted key-metadata check. Ask for any additional permission through the account's consent flow; do not silently create, revoke, export or use keys.

Explain where I can inspect and revoke this connection. Do not make paid inference requests until we have agreed on the work and an explicit usage cap. Do not claim that account history, memories or operator-blind inference are available unless the published release has verified those capabilities.
