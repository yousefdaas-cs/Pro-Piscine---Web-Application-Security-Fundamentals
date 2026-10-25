# Verified Exploit Payloads

The following verified payloads demonstrate how an attacker can leverage the dual sinks to extract session metadata or sensitive objects (like `document.cookie`).

---

## 1. Native Script Vector (Recommended for 1-Click Execution)
Since the application evaluates user input straight inside a dynamically appended `<script>` tag context, we do not need to wrap our payload in HTML brackets. You can write pure, unadulterated JavaScript.

* **Payload:**
  ```javascript
  document.getElementById('cookieOutput').innerText = document.cookie;
  ```
* **Execution Flow:** Targets the empty `#cookieOutput` container block safely using `innerText` without resetting the document body mid-execution. Runs cleanly on the **very first click**.

---

## 2. Image Error Handling Vector (Exploiting the HTML Sink)
This payload ignores the script constructor sink entirely and forces execution directly out of the initial `.innerHTML` parameter mapping.

* **Payload:**
  ```html
  <img src=x onerror="document.getElementById('cookieOutput').innerText = document.cookie">
  ```
* **Execution Flow:** The browser drops the broken image source markup onto the DOM element tree, instantly triggers the `onerror` callback flag, and prints the cookie to the page area on the **first submission**.

---

## 3. Destructive Body Appender (The 2-Click Method)
The payload that initially exposed the structural multi-step update loop constraint.

* **Payload:**
  ```html
  document.body.innerHTML += '<h1>' + document.cookie + '</h1>'
  ```
* **Execution Flow:** Wipes the DOM environment structure clear during initial processing, requiring a **second page submission action** or manual container reload to fully show the modified output.


