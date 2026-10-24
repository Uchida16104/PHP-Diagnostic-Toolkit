# PHP Diagnostic Toolkit

A lightweight, browser-based workspace for investigating PHP-related issues, recording hypotheses, checking selected code patterns, and demonstrating isolated SQL behavior. The project intentionally contains only two files: `index.html` and `README.md`.

> **Scope:** This is a triage and learning aid, not a PHP interpreter, PHP syntax checker, static analyzer, vulnerability scanner, or automatic repair tool. Browser checks cannot prove the root cause of a server-side PHP failure. Confirm all findings in the actual PHP environment.

## Repository name

`php-diagnostic-toolkit`

Use this exact, lowercase, kebab-case name when creating the GitHub repository.

## Files

```text
php-diagnostic-toolkit/
├── index.html   # Complete UI, CSS, application JavaScript, and embedded examples
└── README.md    # Setup, scope, dependency list, security notes, and validation workflow
```

## Features

- **Issue intake:** Records the observed symptom, issue category, PHP version, sanitized error message, and reproduction steps.
- **Investigation checklists:** Presents diagnostic steps for syntax, compatibility, runtime configuration, Composer dependencies, databases, security, performance, and unknown issues.
- **PHP text-pattern scan:** Flags a small number of recognizable patterns for manual review. Results are deliberately described as heuristics, not proof of validity or vulnerability.
- **Isolated SQL scratchpad:** Uses `sql.js` with an in-memory SQLite database and fictional sample rows. It is not connected to any PHP server or real database.
- **Optional Python text analysis:** Uses PyScript to count characters and lines in a sanitized error message and identify repeated lines.
- **Root-cause workflow:** Renders a Mermaid diagram showing reproduction, hypothesis testing, targeted remediation, and regression testing.
- **Explicit local notes and theme preference:** Notes are written to browser `LocalStorage` only when saved. The light/dark preference is saved separately.
- **Responsive interface:** Uses Bootstrap, Tailwind's browser build for prototyping utilities, EaseMotion CSS, Motion, Alpine.js, HTMX, and hyperscript.

## Run locally

The main interface can be opened as a static page, but PyScript and sql.js WebAssembly may fail when loaded from a `file://` URL. Prefer a local HTTP server.

### Python

```bash
cd php-diagnostic-toolkit
python3 -m http.server 8000
```

Open <http://localhost:8000> in a modern browser. Stop the server with `Ctrl+C` when finished.

### PHP's built-in static file server (optional)

If PHP is already installed, this also serves the static files; it does not make the tool execute pasted PHP snippets:

```bash
cd php-diagnostic-toolkit
php -S 127.0.0.1:8000
```

Open <http://127.0.0.1:8000>.

## External dependencies

The project has no package manifest or build step. Selected CDN dependencies are version-pinned in `index.html` to reduce unexpected changes. They require a network connection; a strict Content Security Policy, ad blocker, or offline environment may prevent some optional features from loading.

| Library | Version in `index.html` | Purpose |
|---|---:|---|
| EaseMotion CSS | 1.1.0 | Lightweight motion/utility styling |
| Tailwind CSS browser build | 4.3.3 | Prototype utility classes; browser/CDN build is for development and prototyping |
| Bootstrap CSS | 5.3.8 | Responsive layout and SQL result table styles |
| Motion | 14.6.0 | Optional title entrance effect |
| HTMX | 2.0.11 | Lightweight client-side interaction for the start-investigation link |
| Alpine.js | 3.17.5 | Optional lightweight UI behavior dependency; the core app does not require it for data safety or diagnosis |
| hyperscript | 0.9.14 | Optional HTML interaction enhancement |
| sql.js | 1.15.1 | SQLite compiled to WebAssembly for the isolated scratchpad |
| PyScript | 2026.7.3 | Optional in-browser Python text processing |
| Mermaid | 11.15.0 | Diagnostic workflow diagram |
| LocalStorage | Browser API | Explicitly saved notes and theme preference |

### PHP.js note

The project does **not** load or execute a `php.js` interpreter by default. The browser-based PHP VM at <https://github.com/niklasvh/php.js> describes itself as a personal learning project, warns against incorporating it into applications, and notes that functionality is incomplete. Executing arbitrary pasted PHP through that VM would blur the safety boundary this project is designed to maintain. The included text scan instead checks a handful of visible patterns without executing the snippet. If you evaluate a PHP-to-JavaScript project for research, review its source, scope, license, and security posture first; do not treat its output as equivalent to the real PHP runtime.

## What the checks actually do

### Investigation checklist

Checklist suggestions are based on the category selected by the user and a few strings found in the supplied error message. A generated item is a possible investigation path, not a claim that the tool has diagnosed the incident. The user must record the verified cause and supporting evidence after reproducing the failure.

### PHP pattern scan

The scan looks for a limited set of text patterns, including legacy `mysql_*` calls, `each()`, `create_function()`, `eval()`, short opening tags, request-superglobal references, a broad dynamic-SQL pattern, and a closing PHP tag at end of file. This is intentionally incomplete. It can miss defects, flag comments/examples, and cannot understand full PHP syntax, control flow, data flow, namespaces, dependencies, or runtime settings.

For syntax validation, run the affected version's PHP binary against the real file, for example:

```bash
php -v
php -l path/to/file.php
```

If web and CLI PHP may differ, inspect both environments in a safe way. Check Composer's dependency constraints and use the appropriate test suite and a PHP-aware static analyzer such as PHPStan or Psalm where applicable. Pin tool versions in CI and run the tests against the actual target versions.

### SQL scratchpad

The SQL console creates a new temporary SQLite database containing only fictional diagnostic-example rows. It is useful for exploring compatible SQLite statements and result rendering. It cannot validate MySQL/PostgreSQL-specific syntax, credentials, network connectivity, real schemas, transactions, database permissions, or actual PHP database bindings. Refreshing the page recreates the database.

### LocalStorage

- Notes are stored only after clicking **Save note to this browser**.
- The theme preference is saved separately.
- Notes are not synchronized, backed up, or encrypted by this project. Anyone with access to the browser profile may be able to read them.
- Do not save passwords, API tokens, session cookies, production configuration, customer data, or any other sensitive information.
- **Delete all saved notes** removes this tool's notes key. Clearing site/browser storage also removes the saved notes and theme preference.

## Suggested root-cause workflow

1. Record the error message exactly, with secrets and personal information removed.
2. Record the PHP version, SAPI/runtime, OS, relevant extensions, dependency versions, and the recent change that preceded the issue.
3. Create a minimal, deterministic reproduction in a safe development environment.
4. Form one testable hypothesis and change one relevant variable at a time.
5. Verify a cause with repeatable evidence rather than inferring it from a correlation.
6. Apply the smallest appropriate fix and run regression tests on the affected PHP version and database engine.
7. Document the confirmed cause, evidence, remediation, and remaining uncertainties.

## Security and privacy

- Treat all pasted errors and snippets as potentially sensitive. Redact secrets before pasting anything into the browser.
- The application does not send the issue report or notes to an application server. However, external CDN requests are made to load libraries, so this is not an offline-only page.
- Never execute untrusted PHP or SQL against a production environment. This project does not execute PHP; the SQL scratchpad is isolated to its own in-memory SQLite database.
- Do not store secrets in LocalStorage. LocalStorage is not a secure secret store.
- Review and pin external dependencies before any organizational or production deployment. For production, consider self-hosting reviewed dependencies and applying a restrictive Content Security Policy and appropriate Subresource Integrity attributes.
- Use the official PHP documentation and security advisories to validate version-specific findings. Browser-side pattern matching is not a substitute for a security review.

## Browser support and limitations

Use a modern browser with JavaScript and WebAssembly enabled. CDN access is required for external dependencies. PyScript can take time to initialize and may not work from `file://`; use the local HTTP server instructions above. If one enhancement fails to load, the page should still expose the investigation form and most core interactions, but the affected SQL, Python, animation, or diagram feature may be unavailable.

The UI has not been executed against a full matrix of browsers or PHP versions in this generated deliverable. Before relying on it, manually test the interface in your target browsers and validate any proposed PHP fix in a real environment matching the affected PHP version, dependencies, and database engine.

## License

No project-wide license is declared by these two files. Before publishing or redistributing the repository, choose a license for your own code and confirm the licenses and notices of all external dependencies. The upstream project links above are provided for dependency review, not as an endorsement of every dependency in a production environment.
