## Encoding Hygiene
- All markdown stays UTF-8 (no BOM). If Vietnamese text shows mojibake (e.g., `lÃ `), run a one-time repair with `python -m pip install ftfy` and a small script using `ftfy.fix_encoding` on the affected files, then save back as UTF-8.
- Do not keep alternate encodings in the repo. The ftfy dependency is only for recovery; it is not required for normal builds.