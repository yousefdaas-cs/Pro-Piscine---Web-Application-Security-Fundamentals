# Vulnerability Fix Report: Server-Side Template Injection (SSTI)

This document outlines the vulnerability details and the exact fix for the Server-Side Template Injection (SSTI) flaw found in the application's implementation.

---

## 🔍 Vulnerability Analysis

The flaw exists in the `/render` route within the Python application script. 

### Vulnerable Code Section:
```python
user_input = request.form.get('input')
template = f'Hello, {{ {user_input} }}!'
result = render_template_string(template)
```

### The Root Cause:
* **Dynamic Template Generation:** The code uses an f-string (`f'Hello, {{ {user_input} }}!'`) to inject untrusted user input directly into the template's structure before compiling it.
* **Interpretation as Code:** Instead of treating `user_input` as plain text data, the Jinja2 engine interprets anything inside the brackets as active executable template logic. This allows attackers to run reflection techniques and execute arbitrary code.

---

## 🛠️ Remediation & Fix

To secure the application, **never construct template strings by concatenating or formatting user input**. Instead, define a static template string and pass the untrusted user input as a safe context variable.

### 1. Updated `.py` File Implementation

Replace current script with this secure configuration:

```python
from flask import Flask, request, render_template_string

app = Flask(__name__)

@app.route('/')
def index():
    return '''
    <html>
    <head>
        <title>Template Injection App</title>
    </head>
    <body>
        <h1>Template Injection App</h1>
        <form method="POST" action="/render">
            <label for="input">Enter your input:</label>
            <input type="text" id="input" name="input">
            <button type="submit">Render</button>
        </form>
        <div id="result">{{ result }}</div>
    </body>
    </html>
    '''

@app.route('/render', methods=['POST'])
def render():
    user_input = request.form.get('input')
    
    # FIX: The template string remains strictly static. 
    # The user input is passed safely as a context variable ('user_val').
    template = 'Hello, {{ user_val }}!'
    
    try:
        result = render_template_string(template, user_val=user_input)
    except Exception as e:
        result = str(e)
        
    return render_template_string(index(), result=result)

if __name__ == '__main__':
    app.run(host='0.0.0.0', port=5000)
```

---

## 🛡️ Security Validation & Impact

* **Context Separation:** By passing `user_val=user_input`, Flask ensures that Jinja2 evaluates the input strictly as a string literal.
* **Neutralization:** If an attacker submits a payload like `config.__class__...`, the application will simply output the literal text `Hello, config.__class__... !` safely on the page rather than evaluating it as a command.

