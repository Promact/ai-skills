# Security

- Never hardcode secrets, API keys, or credentials in source — use
  environment variables or a secrets manager.
- Treat all external input (user input, query params, headers, file
  uploads) as untrusted; validate and sanitize before use.
- Use parameterized queries/ORM methods — never build SQL via string
  concatenation.
- Encode output appropriately for its context to prevent XSS (web) and
  injection (shell, SQL, template) vulnerabilities.
- Apply the principle of least privilege for service accounts, API tokens,
  and database roles.
- Keep dependencies patched; flag known-vulnerable packages during review.

<!-- TODO: replace/extend with Promact's internal security checklist -->
