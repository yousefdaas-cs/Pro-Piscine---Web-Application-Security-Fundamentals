# Server-Side Template Injection (SSTI)

## What is SSTI?
**Server-Side Template Injection (SSTI)** is a critical vulnerability that occurs when user-supplied input is unsafely concatenated into a template instead of being passed as data. This allows an attacker to inject malicious payloads into a template, which is then executed on the server, potentially leading to Remote Code Execution (RCE), data exfiltration, or complete server compromise. 

While SSTI can affect various web technologies and template engines, it is highly prevalent in ecosystems using engines like **Jinja (Python)**, Twig (PHP), and FreeMarker (Java).

---

## The Secure Boundary: Template Source vs. Template Data

The most effective way to prevent SSTI during code reviews is to strictly distinguish between **template source** and **template data**. 

### Safe Practice: Passing Data to Context
Passing request data or user content as a value to the context dictionary inside `render()` is perfectly safe. Jinja treats this strictly as data and **will not** compile or execute any template syntax contained within that value a second time.

```python
# SAFE: User input is treated strictly as template data
user_input = request.args.get('name')
return render_template('index.html', name=user_input)
```

### Dangerous Practice: Dynamic Source Compilation
The vulnerability triggers when untrusted request data, stored user content, or strings concatenated with either one are used to construct the **template source itself**. This forces the template engine to compile and evaluate the malicious input.

Avoid passing raw user input into compilation functions such as:
* `Template(...)`
* `Environment.from_string()`
* Any equivalent runtime string-compilation methods

```python
# VULNERABLE: User input becomes part of the template source
user_input = request.args.get('name')
template = Environment().from_string(f"<h1>Hello {user_input}</h1>") # <--- SSTI Boundary
return template.render()
```

---

## Remediation Strategies
* **Treat Templates as Code:** Keep templates static within localized files and never generate template strings dynamically from user inputs.
* **Contextual Passing:** Always pass user-controlled variables exclusively into the template context via keyword arguments in the `.render()` function.
* **Input Sanitization & Sandboxing:** If dynamic template generation is an absolute business requirement, implement strict allow-lists for inputs and utilize heavily restricted, sandboxed environments.
