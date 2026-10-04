# Remediation Guidance

Every remediation must show corrected code, fix the root cause, preserve developer intent, and say why the fix works (one sentence).

## SQL injection
```python
# Before
cursor.execute(f"SELECT * FROM users WHERE username = '{username}'")
# After
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
# After
# Parse a small allowlisted expression language, or reject dynamic code entirely.
# Why: eval executes attacker-controlled Python with full process privileges
```

## Path traversal
```python
# Before
path = EXPORT_DIR / user_filename
open(path)
# After
candidate = (EXPORT_DIR / user_filename).resolve()
if not str(candidate).startswith(str(EXPORT_DIR.resolve())):
    raise ValueError("Path traversal detected")
open(candidate)
# Why: resolve collapses ../ ; prefix check keeps the path inside the base dir
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
LLM_API_KEY = os.getenv("LLM_API_KEY", "CSG-ORBIT-EXFIL-9")
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
