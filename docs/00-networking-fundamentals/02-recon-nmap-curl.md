# Reconnaissance Against My Own Web Server (nmap + curl)

**Goal:** see my public server the way an attacker on the internet would, using the same tools, against my own infrastructure only.

**Target:** `13.63.229.50` (Cloudlearner-server, Apache). Scanned from my iPad (iSH), a separate network from the server itself, to get a genuine outside view rather than testing from inside the box.

## 1. Port scan: outside view vs inside view

Earlier work from inside the server (`ss -tlnp`) and a direct read of the AWS security group showed: port 80 and 443 open to the internet, no inbound rule for port 22.

An external nmap scan confirmed the same result independently:

`filtered` means packets went out and nothing came back, the signature of a firewall silently dropping traffic. This matches the security group having no rule for port 22 at all.

**Conclusion:** the server's own view and an independent external scan agree. SSH is unreachable from the internet despite `sshd` running locally; verified from both directions, not just read from a config.

**Limitation hit:** `-sV` and `-sC` failed under iSH's sandboxed networking (`SO_BINDTODEVICE` errors), returning a false "all filtered" result that contradicted the clean scan. Logged as a tooling limitation, not a server finding, worked around with direct `curl` checks instead.

## 2. What the server reveals about itself

**Finding:** the exact Apache version, OS family, and OpenSSL version are handed to any visitor with no effort. This is normally what `nmap -sV` tries to infer by fingerprinting; here the server states it outright. Anyone scanning the internet for a known vulnerability in this exact version can trivially find this server without even probing further.

**Fix (Phase 2):** `ServerTokens Prod` in Apache config, trims the banner to just `Server: Apache`.

Also missing from the response: no `Content-Security-Policy`, `X-Frame-Options`, or `Strict-Transport-Security` headers. Not a risk for a static page with no scripts or forms today, but a standard hardening gap to close.

## 3. Allowed HTTP methods

**Finding:** `TRACE` is enabled. TRACE causes the server to echo the exact request back in the response body. Combined with malicious JavaScript on another site a victim visits, this is the basis of a known attack class (Cross-Site Tracing), where an attacker's script can read the echoed response to steal cookies or auth headers that would otherwise be protected. No cookies or auth exist on this site today, so there's nothing to steal right now, but it's exactly the kind of default that becomes dangerous the moment the server starts handling anything stateful.

`POST` is also accepted despite there being no backend logic to process posted data, extra attack surface with no current purpose.

**Fix (Phase 2):** `TraceEnable off`; restrict methods to `GET, HEAD, OPTIONS` via `<LimitExcept>`.

## 4. Hidden/sensitive path check

Checked for common leftover files attackers (and the bot already seen in the server's own access log) probe for automatically:

All eight returned `404`. No exposed git folder, environment file, admin panel, backup, or config. Confirmed clean, not just assumed clean from reading the earlier log entries where an automated bot tried similar paths and also got `404`.

## Summary of findings

| Check | Result | Action |
|---|---|---|
| SSH reachable externally | Filtered (not reachable), verified from both inside and outside | None needed |
| Web ports reachable | Open (80, 443), as intended | None needed |
| Version banner | Exposed (Apache, OS, OpenSSL version) | Hide in Phase 2 (`ServerTokens Prod`) |
| HTTP methods allowed | Includes `TRACE` and unused `POST` | Restrict in Phase 2 (`TraceEnable off`, `<LimitExcept>`) |
| Security headers | None present (CSP, HSTS, X-Frame-Options) | Add in Phase 2 |
| Hidden/sensitive paths | None exposed | None needed |

## Terms used

**Recon(naissance):** gathering information about a target before deciding what, if anything, to attack. **Port scan:** checking which network doors respond. **Filtered/open/closed:** nmap's three possible verdicts per port. **Version banner:** software/OS details a server volunteers in its responses. **HTTP method:** the verb of a web request (GET, POST, OPTIONS, TRACE...). **TRACE/XST:** a method that echoes the request back, exploitable for credential theft if cookies/auth are present. **curl:** a command-line tool for making raw HTTP requests and inspecting the exact response.
