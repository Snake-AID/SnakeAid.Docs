## Encoding Hygiene
- All markdown stays UTF-8 (no BOM). If Vietnamese text shows mojibake (e.g., `lÃ `), run a one-time repair with `python -m pip install ftfy` and a small script using `ftfy.fix_encoding` on the affected files, then save back as UTF-8.
- Do not keep alternate encodings in the repo. The ftfy dependency is only for recovery; it is not required for normal builds.

## API Reproduction Context
- When reproducing HTTP requests that come from Swagger snippets or from the user's Git Bash terminal, execute them with `Git Bash` semantics by invoking `C:\Program Files\Git\bin\bash.exe`.
- Do not use PowerShell-native `curl`, `Invoke-WebRequest`, or ad-hoc PowerShell quoting for these reproductions unless the user explicitly asks for a PowerShell-specific reproduction.
- Preserve the request shape exactly as provided by the user whenever the goal is behavior matching:
  - keep the same URL, including IPv6 forms like `http://[::1]:8080/...`
  - keep the same headers unless the user asks for normalization
  - keep the same body shape and field casing
- For Git Bash `curl` reproductions:
  - prefer `curl --globoff` when the URL contains IPv6 brackets
  - prefer here-doc or temp-file payloads for JSON bodies to avoid quote escaping drift
  - prefer a temporary bash script when the command is long, rather than nesting complex quoting through PowerShell
- If a normalized request is needed for diagnosis, separate it explicitly from the exact reproduction:
  - first run `reproduce-exactly`
  - then run `normalize-request` only if needed, and state clearly that it is a diagnostic comparison rather than the canonical reproduction
