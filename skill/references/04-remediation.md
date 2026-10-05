# Remediation Guidance

Every remediation must show corrected code, fix the root cause, preserve developer intent, and say why the fix works (one sentence).

Templates below are **Python and JavaScript only** (enough for this demo skill). Match the project’s SQL driver placeholder style (`?`, `%s`, `:name`, …).

## SQL injection
```python
# Before
cursor.execute(f"SELECT * FROM users WHERE username = '{username}'")
# After (sqlite3 uses ?; other drivers may use %s or :username — match the project)
cursor.execute("SELECT * FROM users WHERE username = ?", (username,))
# Why: the driver treats the value as data, never as SQL syntax
```

## Command injection
```python
# Before
subprocess.run(f"convert {filename}", shell=True)
# After
subprocess.run(["convert", filename], shell=False)
# Why: list args bypass shell interpretation
```

## eval / code injection
```python
# Before
result = eval(expression)
# After — for simple literals only:
import ast
result = ast.literal_eval(expression)
# Or reject dynamic code and parse an allowlisted grammar (numbers/ops only).
# Why: literal_eval cannot execute arbitrary Python; eval can
```

## Path traversal
```python
# Before
path = EXPORT_DIR / user_filename
open(path)
# After
base = EXPORT_DIR.resolve()
candidate = (EXPORT_DIR / user_filename).resolve()
if not candidate.is_relative_to(base):
    raise ValueError("Path traversal detected")
open(candidate)
# Why: resolve collapses ../ ; is_relative_to avoids prefix tricks like base_evil/
```

## Insecure randomness
```python
# Before
random.seed(int(time.time()))
token = str(random.randint(100000, 999999))
# After
import secrets
token = secrets.token_urlsafe(32)
# Why: secrets uses the OS CSPRNG; random is predictable
```

## Secrets / API keys
```python
# Before
LLM_API_KEY = os.getenv("LLM_API_KEY", "sk-demo-insecure-default")
# After
LLM_API_KEY = os.environ["LLM_API_KEY"]  # fail closed if unset; never ship real defaults
# Why: default secrets in source become public credentials
```

## XSS
```javascript
// Before
el.innerHTML = userInput;
// After
el.textContent = userInput;
// Why: textContent does not interpret HTML/script
```

## LLM tools / sensitive data
```python
# Before: return HR row because the model asked / an output filter might catch secrets
# After: enforce authz in the tool handler before any DB read or secret is loaded
if not (is_admin(user) or user["employee_id"] == employee_id):
    raise HTTPException(status_code=403, detail="Forbidden")
# Why: models and output filters are not access-control boundaries
```

## Hallucinated dependency (slopsquatting)
```text
# Before: install a package name that appeared in an LLM answer but is not a known project
pip install super-secure-http-helpers

# After: verify the project exists on the official registry and matches an expected
# homepage/org; pin a known-good version in requirements/lockfile; prefer the
# well-known library that already does the job.
# Why: nonexistent names can be claimed later by attackers who publish malware
```
