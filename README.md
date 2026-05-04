# Web Offlinea
HTB Offlinea
from pathlib import Path

## Overview
This challenge combines an SSRF validation flaw with a Python format-string injection to leak the Flask `SECRET_KEY`, then uses that key to forge a valid JWT and access the protected endpoint. The final result is the flag from the `secrets` table.  

## Environment
- Target host
- Kali Linux terminal
- VPN connection to Hack The Box

## Reconnaissance
- Confirmed the host was alive and serving a normal HTML page.
- Opened the target in Firefox and observed the Offlinea interface.
- Verified that the downloaded PDF was a valid one-page A4 document.
- Noted that the PDF metadata suggested a Chromium-based rendering engine and referenced `127.0.0.1:8000/bartender.php`.

## Code Review
The extracted lab files showed a clear split between the public-facing service and internal application components:
- `bartender.php`
- `app.py`
- `init_db.py`
- `templates/bartender.html`

Key findings:
- `bartender.php` forwards user-supplied URLs to an internal Flask service on `127.0.0.1:5000`.
- The PHP validation checks private and reserved IP ranges.
- The internal Flask app uses Selenium and headless Chrome with JavaScript disabled to render webpages into PDF files.
- `init_db.py` creates the `secrets` and `history` tables and inserts an initial secret from `flag.txt`.

## Vulnerabilities
### 1) SSRF bypass
The PHP layer validates the last `url` parameter, while the internal Flask service processes the first one. Supplying multiple `url` parameters bypasses the filter and forces the backend to access localhost.

### 2) Python format-string injection
The application formats log data with Python `.format()` using values from the database. That allows access to Flask globals and leaks `app.config`, including the `SECRET_KEY`.

## Exploitation Chain
1. Bypass SSRF validation with multiple `url` parameters.
2. Trigger the internal service to render a crafted payload.
3. Leak the Flask `SECRET_KEY` through format-string injection.
4. Forge a valid JWT using the leaked key.
5. Access the protected `/bartender` endpoint.
6. Retrieve the contents of the `secrets` table and the flag.

## Result
The exploit was successful and the flag was obtained:
`HTB{3b0cbe383374cf3f02336f9e0822997d}`

## Notes
This challenge shows how small inconsistencies between components can be chained into a full compromise of a web application.
"""

path = Path("/mnt/data/README.md")
path.write_text(readme, encoding="utf-8")
print(path)
