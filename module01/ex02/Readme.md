# Server-Side Request Forgery (SSRF) with Local File Scheme Access (LFI)

## Vulnerability Type
*   **Vulnerability Name:** Server-Side Request Forgery (SSRF) via Local File Protocol Handler

## Explanation
Server-Side Request Forgery (SSRF) occurs when a web application accepts a user-supplied URL or file path and fetches data from that resource without proper validation. When an application passes this input directly to a backend HTTP client or a file system parser, it trusts the server's own networking and file permissions to resolve the request.

If the backend parser or HTTP library supports alternative URL schemes (such as `file://`), an attacker can shift the context of the request away from external internet resources. Instead, the server is forced to interact with its own local loopback interfaces or local file storage. By specifying the `file://` scheme, the attacker tricks the application into operating as a Local File Inclusion (LFI) or local file read primitive, allowing unauthorized access to restricted system configuration files like `/etc/passwd`.

### Real-World Scenarios
1. **PDF Generators:** An attacker inputs `file:///etc/passwd` into a "Print to PDF" tool. The server reads the file and prints the system secrets directly onto the generated PDF.
2. **Image Upload links:** An attacker inputs the file link into an "Import Profile Picture from URL" field. The server reads the system text file instead of an image, leaking the content in an error message.

### How to Fix It
* **Allow Only HTTP/HTTPS:** Use a URL parser to check every input link. Block any link that does not start strictly with `http://` or `https://`.

