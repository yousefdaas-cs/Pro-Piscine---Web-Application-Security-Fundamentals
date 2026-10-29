# Remediation Guidance: Protecting Against CSRF

To fully secure the application against Cross-Site Request Forgery, implement a defense-in-depth approach incorporating both cryptographic tokens and modern cookie attributes.

### 1. Implement Anti-CSRF (Synchronizer Token Pattern)
The most resilient defense is the integration of cryptographically secure, unique, and unpredictable tokens for every state-changing session.
* **Generation:** Generate a high-entropy, random token linked directly to the user's current session on the backend.
* **Injection:** Embed this token inside all HTML forms or supply it via an exposed meta tag for API contexts.
* **Verification:** Reject any inbound `POST`, `PUT`, `DELETE`, or `PATCH` request to sensitive endpoints if the submitted token is missing, invalid, or mismatched with the server-side session variable.

### 2. Enforce SameSite Cookie Attributes
Configure all session cookies to utilize modern browser handling flags to control how cookies are sent along with cross-site requests.
* **SameSite=Strict:** The cookie is completely withheld from all cross-site requests (such as clicking a link from an external email or a third-party site).
* **SameSite=Lax (Minimum Standard):** The cookie is withheld on cross-site subrequests (like images or hidden forms), but sent when a user naturally navigates to the origin site (e.g., following a standard hyperlink).

