# security-review

Post-development security review skill for Claude Code.

## Features

- OWASP Top 10 vulnerability scanning
- Input validation & injection attack detection (SQL, XSS, Command, Template)
- Authentication/authorization pattern review
- Sensitive data exposure check
- Dependency vulnerability audit
- Frontend-specific security checks
- Structured report with PASS/WARN/FAIL ratings

## Usage

```
/security-review [file path, directory, or git diff range]
```

Examples:
```
/security-review src/api/
/security-review HEAD~3
/security-review src/auth/login.ts
```
