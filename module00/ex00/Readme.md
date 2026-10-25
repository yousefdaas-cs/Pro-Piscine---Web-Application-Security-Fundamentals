# Vulnerability Report: DOM-Based Cross-Site Scripting (DOM XSS)

## Overview
This document outlines a critical vulnerability found within the client-side JavaScript of the application. The input processing flow allows user-supplied data to reach sensitive execution contexts without sanitization, resulting in **DOM-Based Cross-Site Scripting (DOM XSS)**. 

##in this scenario we are dealing with DOM-based XSS.
**the other types are:**
- Reflected XSS
- Stored XSS
  - Blind XSS

---

## Vulnerability Breakdown
The vulnerability stems from the implementation of the `displayText()` function within the application's client-side script:

```javascript
function displayText() {
    var userInput = document.getElementById("inputText").value;
    document.getElementById("output").innerHTML = "<b>" + userInput + "</b>";

    var script = document.createElement("script");
    script.textContent = userInput;
    document.body.appendChild(script);
}
```

### 1. The Sources (Unsafe Inputs)
* **`document.getElementById("inputText").value`**: The data is retrieved directly from a user-controlled text entry form field.

### 2. The Sinks (Dangerous Outputs)
This application simultaneously utilizes two separate dangerous DOM execution sinks:
* **Sink 1 (`.innerHTML`)**: The input string is concatenated inside `<b>` tags and directly assigned to the `.innerHTML` property of the `#output` div. This interprets any submitted HTML tags (such as `<img onerror=...>` or `<svg onload=...>`) as executable code.
* **Sink 2 (`script.textContent`)**: The script dynamically initializes a standard HTML `<script>` element, binds the raw `userInput` text directly into the `textContent` execution environment, and appends it onto the document body. This executes any raw JavaScript string directly within the user's session context.

---

## Behavior & The "Double-Submit" Phenomenon
When injecting a destructive structural payload like `document.body.innerHTML += ...`, the script execution cycle breaks:
1. **First Click**: The payload enters both sinks. When the JavaScript engine runs the dynamic `<script>` tag element, modifying `document.body.innerHTML` forces the browser to completely wipe out and rebuild the application's living DOM mid-flight. This action halts script tracking before it updates visually.
2. **Second Click**: Because the entire page DOM structure was re-rendered from the initial payload execution, hitting the trigger event a second time fires the freshly appended code against the brand-new structural frame, successfully surfacing the target data.
