## 9. Security Impact

Stored XSS can be especially dangerous because malicious input persists in the application and may affect multiple users who view the same content.

Depending on the application's functionality, successful exploitation may allow:

- Manipulation of webpage content.
- Unauthorized actions performed as a victim.
- Access to information exposed to JavaScript.
- Phishing or deceptive content displayed through a trusted website.

The actual impact depends on the application's permissions and security controls.

## 10. Remediation

### Recommended defenses

**1. Avoid inline JavaScript event handlers**

Instead of constructing `onclick` attributes dynamically, register event listeners in trusted application code.

**2. Apply context-aware output encoding**

HTML attribute encoding and JavaScript string encoding are different requirements. Protect each context appropriately.

**3. Use safe DOM APIs**

Use DOM properties and event listeners rather than constructing HTML strings containing untrusted input.

**4. Validate and sanitize Website values**

Validate URLs using an appropriate URL parser and an allowlist of permitted schemes, such as HTTPS.

**5. Implement Content Security Policy**

A restrictive CSP can reduce the impact of inline script injection, although it should not replace safe output handling.
