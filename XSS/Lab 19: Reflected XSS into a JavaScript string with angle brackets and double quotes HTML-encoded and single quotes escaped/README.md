## 7. Security Impact

Successful reflected XSS can allow attacker-controlled JavaScript to execute in the context of the vulnerable application's origin.

Depending on the application, possible consequences include:

- Manipulation of webpage content.
- Unauthorized actions performed as the victim.
- Access to information exposed to JavaScript.
- Phishing or deceptive content presented through the trusted application.

The actual impact depends on the application's functionality and security controls.

## 8. Remediation

### Recommended defenses

**1. Avoid embedding untrusted input directly into JavaScript**

Do not construct executable JavaScript source by concatenating user-controlled values.

- **. Use safe serialization**

Use a reliable JSON serializer when transferring server-side values into JavaScript. Ensure that values are safely embedded in the surrounding HTML context as well.

**3. Escape backslashes and quotes correctly**

If JavaScript string encoding is unavoidable, use a context-aware encoder that handles all relevant characters and escaping rules correctly.

**4. Prefer safe DOM APIs**

Use `textContent` for displaying user-controlled text rather than generating HTML or executable JavaScript dynamically.

**5. Implement Content Security Policy**

A restrictive CSP can provide additional defense against script injection, but it does not replace correct output encoding.
