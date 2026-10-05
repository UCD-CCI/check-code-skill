# Security Checks

Check every category that applies to the languages and features in scope. Never skip a class because the code “looks fine.”

## Injection

| Class | CWE | Look for | Safe pattern |
|-------|-----|----------|--------------|
| SQL injection | 89 | f-strings / concat into SQL; unsafe ORM `.raw()` | Bound parameters / query builder |
| Command injection | 78 | `os.system`, `subprocess(..., shell=True)`, shell strings with user data | Arg list, `shell=False` |
| Code injection / eval | 94 | `eval` / `exec` / `render_template_string` on user input | Never evaluate untrusted code; parse/allowlist |
| SSTI | 94 | User input as template **source** | Data-only rendering; sandboxed engines |
| XXE | 611 | XML parsers with external entities enabled | Disable DTDs / external entities |
| Header / response splitting | 113 | Request values copied into headers/`Location` without CRLF rejection | Framework-safe header APIs; allowlisted redirects |

## XSS — CWE-79

- Reflected / stored: user data in HTML without encoding
- DOM: `innerHTML` / `document.write` / `eval` with attacker-controlled sources  
Safe: context-aware encoding; prefer `textContent`

## Access control — OWASP A01

- **Missing authentication** on sensitive routes
- **Missing function-level authz** on admin/privileged actions
- **IDOR / BOLA** — object IDs from the client without ownership checks
- **Path traversal (CWE-22)** — user path joined to a base dir without `realpath` + prefix check (`../`)
- **Mass assignment** — client can set `role`, `isAdmin`, ownership fields

Enforce authz in application code for sensitive operations; edge-only rules are defense in depth.

## Auth, sessions, secrets

- Plaintext or fast hashes for passwords (MD5/SHA1) — use bcrypt/Argon2id
- Hardcoded or insecure default secrets / API keys in source or env fallbacks (CWE-798)
- Session not rotated after login; long-lived tokens without expiry
- Passwords or secrets stored in session cookies or returned by `/me`-style APIs
- Sensitive data in logs or error responses

## Crypto & randomness

- `random` / `Math.random` / `rand` for reset tokens, session IDs, API keys — use CSPRNG (`secrets`, `crypto.randomBytes`, etc.)
- Predictable seeding (`random.seed(time...)`) for security values
- Weak TLS (`verify=False`, `InsecureSkipVerify`)
- ECB mode, hardcoded keys/IVs, JWT `alg: none`
- Secret compares with `==` — use constant-time compare (`hmac.compare_digest`)

## Deserialization, SSRF, CSRF, redirects

- `pickle.loads`, unsafe `yaml.load`, PHP `unserialize`, Java `ObjectInputStream` on untrusted data
- User-controlled URLs to HTTP clients / webhooks / preview fetchers (SSRF)
- Cookie-auth state changes without CSRF protection
- Open redirect via unvalidated `next` / `return_to`

## Supply chain — hallucinated / typo-squatted packages

AI assistants and copy-pasted snippets often invent package names that do not exist yet (**hallucinated dependencies** / “slopsquatting”). An attacker can later publish a **malicious package under that exact name** on PyPI/npm; the next `pip install` / `npm install` pulls malware.

**Flag when you see:**
- Imports or dependency pins for packages that look plausible but are uncommon, oddly spelled, or not verifiable in the lockfile/registry evidence in scope
- README/setup instructions that add libraries suggested by an LLM without a known upstream
- Near-miss names that mimic popular libraries (`reqeusts`, `lodashes`, `open-ai-python`, etc.)

**Safe pattern:** only add dependencies that exist on the official registry, match a known project/URL, and are pinned via a lockfile; prefer well-known packages over novel names from chat.

When reporting, CWE-829 (Inclusion of Functionality from Untrusted Control Sphere) or CWE-1357 (Reliance on Insufficiently Trustworthy Component) often fit; OWASP category can be supply-chain / LLM-assisted insecure advice.

## Language quick flags (apply what matches)

**Python:** `eval`/`exec`, `shell=True`, `pickle.loads`, unsafe `yaml.load`, `random` for tokens, `open(user_path)`, f-string SQL, `verify=False`  
**JavaScript / Node:** `eval`, `innerHTML`, `child_process.exec`, `Math.random` for tokens, unvalidated `res.redirect`  

This demo skill focuses on Python and JS; do not expand into other languages unless the reviewed code is clearly in another language and the same sink classes apply.

## LLM / GenAI trust boundaries

When the app uses models, tools, or prompt-shaped workflows, also check:

1. **Secrets in prompts** — anything in the system/user context can leak; do not put API keys or confidential phrases in prompts
2. **Output filters are not access control** — exact-string redaction / model refusals are bypassable (encoding, paraphrase); authorize *before* data reaches the model
3. **Tools need the same authz as APIs** — LLM-triggered lookups/actions must enforce authentication and object/function-level authorization in server code
4. **Untrusted content is not instructions** — tickets, emails, pasted notes must not auto-trigger privileged tools (indirect prompt injection)
5. **Excessive agency** — privileged actions (reset password, approve, export) must not be callable without real role checks just because a model “chose” a tool
6. **Hallucinated libraries** — model-suggested package names that are not real can be registered later as malware; treat novel dependency names as supply-chain risk (see above)

Prefer findings that name the missing server-side control, not only “the model might refuse.”
When reporting, use OWASP LLM/GenAI category labels where they fit (see `03-output-format.md`).
