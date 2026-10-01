<img width="1376" height="768" alt="image" src="https://github.com/user-attachments/assets/19d4505d-eecd-428b-80b9-faa0efddf364" />

# Credential Checker

Credential Checker is a personal security utility that allows users to generate credentials, evaluate password entropy, and check credentials against known public data breach records. The application computes cryptographic hashes in the user's web browser and uses k-anonymity range queries so that cleartext passwords never cross the network. A scoped language model assistant is embedded to explain derived scan results and provide defensive account hygiene advice without accessing cleartext credentials.

<img width="1440" height="900" alt="image" src="https://github.com/user-attachments/assets/4d9e0b5b-80d6-43f7-9e06-9bb2f94ffb64" />


## Features

- **Random credential generation:** Generates cryptographically random passwords, Diceware-style multi-word passphrases, and pseudonymous usernames client-side using the Web Crypto API.
- **Batch generation and export:** Supports generating multiple credentials simultaneously with one-click clipboard copying and JSON file export.
- **Client-side entropy analysis:** Calculates information entropy, evaluates character pool sizes, flags sequential and leetspeak patterns, and estimates crack times across hardware tiers.
- **Dual-source password breach check:** Hashes passwords locally with SHA-1 and Keccak-512, sending only truncated prefixes to HaveIBeenPwned and XposedOrNot to check exposure counts without exposing credentials.
- **Interactive k-anonymity visualizer:** Displays real-time separation of transmitted hash prefixes and locally withheld suffixes.
- **Email breach detection:** Queries public breach collections by email address using the XposedOrNot endpoint.
- **Scoped security assistant:** Provides structured account remediation guidance using an embedded security prompt, strictly refusing off-topic or harmful queries.
- **Zero-trust agent pipeline:** Inspects inputs with a prompt firewall, validates role permissions, tracks runtime anomalies with a circuit breaker, and records events to an HMAC-chained audit log.

## How it works

### System data flow

```
+-------------------------------------------------------------+
| Client Browser (app/page.tsx)                               |
| - Generates credentials using Web Crypto API                |
| - Computes SHA-1 (HIBP) and Keccak-512 (XposedOrNot) hashes |
| - Sends only 5-char and 10-char hash prefixes               |
+-------------------------------------------------------------+
                              |
            +-----------------+-----------------+
            |                                   |
            v                                   v
+-------------------------------+ +-------------------------------+
| External Breach APIs (Direct) | | Serverless API (Next.js)      |
| - api.pwnedpasswords.com      | | - /api/check-password (Proxy) |
| - passwords.xposedornot.com   | | - /api/check-email (Proxy)    |
| - api.xposedornot.com         | | - /api/chat (Security Advisor)|
+-------------------------------+ +-------------------------------+
                                                |
                                                v
                                  +-------------------------------+
                                  | Zero-Trust Security Pipeline  |
                                  | - Prompt Firewall             |
                                  | - Policy Engine               |
                                  | - Anomaly Detector            |
                                  | - HMAC-Chained Audit Logger   |
                                  +-------------------------------+
                                                |
                                                v
                                  +-------------------------------+
                                  | Model Gateway / Fallback      |
                                  | - OpenRouter (Gemini Flash)   |
                                  | - Deterministic Local Advisor |
                                  +-------------------------------+
```

### Agent execution flow

```
+-------------------------------------------------------------+
| Step 1: User sends query with derived scan metrics          |
| Example: "My password appeared in a breach, what do I do?"  |
+-------------------------------------------------------------+
                              |
                              v
+-------------------------------------------------------------+
| Step 2: Prompt firewall inspects input for injections       |
| - Scans for overrides, jailbreaks, and sensitive tokens     |
+-------------------------------------------------------------+
                              |
                              v
+-------------------------------------------------------------+
| Step 3: Policy engine checks role permissions and goals     |
| - Verifies role "SecurityAdvisor" allows the action         |
+-------------------------------------------------------------+
                              |
                              v
+-------------------------------------------------------------+
| Step 4: System prompt assembled with derived metadata       |
| - Grounding principles from content/strategy.md injected    |
+-------------------------------------------------------------+
                              |
                              v
+-------------------------------------------------------------+
| Step 5: Model generates response or fallback executes       |
| - Output sanitized and event recorded in HMAC audit trail   |
+-------------------------------------------------------------+
```

### Request lifecycle

1. The user creates or checks a credential in the browser interface.
2. The browser calculates mathematical entropy in `lib/strength.ts` and generates SHA-1 and Keccak-512 hashes.
3. The browser requests breach ranges by prefix from external APIs or falls back to `/api/check-password` and `/api/check-email`.
4. Derived metadata (entropy score, breach counts, breach names) is passed to the chat component.
5. When the user sends a message, `/api/chat` validates the request payload using Zod schemas in `security/input_validation.ts`.
6. The prompt firewall in `security/prompt_firewall.ts` scans the message for instruction overrides and malicious patterns.
7. The policy engine in `security/policy_engine.ts` checks that the requested goal and action are permitted for the agent role.
8. The anomaly detector in `security/anomaly_detector.ts` checks for execution loops or velocity spikes.
9. Grounding text from `content/strategy.md` and derived scan metadata are assembled into the system prompt.
10. The request is sent to the OpenRouter gateway, or processed by the local deterministic fallback if no API key is present.
11. The response is sanitized in `security/index.ts` and logged to `.audit/security_audit.jsonl` by `security/audit_logger.ts`.

## Key modules

| File | Purpose |
|---|---|
| `app/page.tsx` | Main application shell coordinating the three functional zones and state synchronization. |
| `components/Generator.tsx` | User interface for single and batch credential generation and JSON export. |
| `components/BreachChecker.tsx` | User interface for password and email breach lookups with k-anonymity visualization. |
| `components/Chatbot.tsx` | Chat interface with client-side credential redaction and markdown rendering. |
| `components/StrategyCard.tsx` | Collapsible interface presenting core operational security principles. |
| `lib/generator.ts` | Cryptographically random password, passphrase, and username generation functions. |
| `lib/strength.ts` | Information entropy calculations, hardware crack-time estimates, and pattern detection. |
| `lib/pwnedPasswords.ts` | SHA-1 hashing and k-anonymity range check client for HaveIBeenPwned. |
| `lib/xposedPassword.ts` | Keccak-512 hashing and k-anonymity check client for XposedOrNot. |
| `lib/breachCheck.ts` | Email breach query client for XposedOrNot public REST endpoint. |
| `app/api/chat/route.ts` | Chat endpoint handling security pipeline stages, prompt assembly, and model routing. |
| `app/api/check-password/route.ts` | Serverless proxy endpoint for HaveIBeenPwned and XposedOrNot password checks. |
| `app/api/check-email/route.ts` | Serverless proxy endpoint for XposedOrNot email breach checks. |
| `app/api/security/route.ts` | Diagnostic endpoint for health status, kill-switch control, and test execution. |
| `middleware.ts` | Request routing middleware enforcing rate limits and Content Security Policy headers. |
| `security/prompt_firewall.ts` | Input inspection service detecting prompt injections, credential leaks, and overrides. |
| `security/policy_engine.ts` | Role-based access control engine enforcing deny-by-default goal and action rules. |
| `security/anomaly_detector.ts` | Execution loop, velocity spike, and goal drift detector with circuit breaker. |
| `security/memory_guard.ts` | In-memory conversation history manager with poisoning checks and snapshot rollbacks. |
| `security/tool_guard.ts` | Sandboxed tool execution registry with parameter validation and rate limiting. |
| `security/audit_logger.ts` | Append-only HMAC-SHA256 hash-chained security event logger. |
| `security/identity_manager.ts` | Token-based agent identity issuer and verifier for role enforcement. |
| `security/inter_agent_security.ts` | Message signing and verification service with replay prevention nonces. |
| `security/sandbox_executor.ts` | Isolated JavaScript execution sandbox using Node.js virtual machine contexts. |
| `security/security_tests.ts` | Automated verification test suite exercising security controls. |
| `content/strategy.md` | Grounding security rules and in-scope topic definitions for the chatbot. |

## Security design

- **K-anonymity prefix lookup:** Passwords are never sent over the network in cleartext or as full hashes.
- **Client-side credential redaction:** The chat interface intercepts potential passwords before transmission to prevent accidental cleartext sharing.
- **Input firewall inspection:** Incoming chat text is inspected for prompt injections, system prompt leaks, and roleplay jailbreaks.
- **Deny-by-default role policy:** The policy engine restricts agent actions to explicitly allowed goals and tools.
- **Circuit breaker kill switch:** Trips automatically upon detecting repetitive execution loops or high-velocity bursts.
- **HMAC-chained audit log:** Every security event is recorded with an HMAC-SHA256 signature linked to the previous entry hash.
- **Trust boundaries:** The client browser is untrusted; the serverless backend executes input validation and security policy checks; third-party APIs receive only truncated hash prefixes or non-sensitive email query strings.


## Roadmap

- Add configurable wordlist selection for the passphrase generator, including standard Diceware and EFF long wordlists.
- Add client-side Web Crypto export for encrypted credential backup vaults.
- Implement server-sent events streaming in the chat route for faster response rendering.
- Add support for custom WebAuthn passkey registration simulation.
