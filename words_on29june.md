# MalikNode Final Project Blueprint & Summary (June 29, 2026)

This document serves as a complete project blueprint, summarizing the work accomplished, backend integrations, credential mappings, and outstanding tasks left for future pairing sessions.

---

## 1. Project Brand & Styling Guidelines
*   **Official Brand Name:** **MalikNode** (rebranded from PortBro).
*   **Signature Color Theme:** **Deep Blue & Indigo** (`#3b82f6` and `#6366f1`). All purple styles in layouts are replaced.
*   **Favicon & Logo:** Tab favicon uses the red/blue shield logo matching `dnsmalik.duckdns.org` chrome tab style.

---

## 2. Infrastructure & Directory Tree
*   **Nginx Document Roots:**
    *   Landing Page (`maliknode.app`): `/var/www/html/maliknode`
    *   Client Dashboard (`dashboard.maliknode.app`): `/var/www/html/maliknode/client`
    *   Developer Specs Portal: `/var/www/html/maliknode/md`
    *   Interactive Mockup Portal: `/var/www/html/maliknode/mockup`
*   **PostgreSQL Port:** `127.0.0.1:5432` (`database: maliknode_db`, `user: maliknode_user`)
*   **Central API Daemon Port:** `127.0.0.1:8086` (`api.maliknode.app` proxied via Nginx)
*   **SMTP Configuration Path:** `/etc/maliknode/central_api/config.json`

---

## 3. Realized Architecture & Codebases

### A. Phase 1: PostgreSQL Relational Schema
The database is initialized in PostgreSQL, containing:
*   `users`: Stores client profiles, plan mappings, subdomain domains, and WireGuard listen ports.
*   `peers`: Houses device labels, allocated subnet static IPs, and WireGuard public/private key pairs.
*   `bandwidth_stats`: Stores transaction data bytes mapping to devices, enabling real-time speed graphing.

### B. Phase 2: FastAPI Control Daemon (`main.py`)
Built as a systemd service (`maliknode-api.service`) listening on port `8086`:
*   **Zero-Downtime Reloads:** Uses `wg syncconf` to modify interfaces on-the-fly.
*   **AdGuard Multi-Tenancy:** Provisions isolated docker containers for each new tenant, mapping port `53` to the client WireGuard gateway IP (e.g. `10.88.x.1:53`).
*   **Shell Injection Protection:** Utilizes list-based `subprocess.run` calls (`shell=False`) to secure server commands.
*   **SMTP Gmail OTP:** Generates 10-minute validity OTP codes. Connects via `smtp.gmail.com` using `wedone01@gmail.com` (App Password: `cpdpetcbzikkqnvc`) and masks the sender address as `noreply@maliknode.app`.

### C. Phase 3: Authentication & Dashboard Integration
*   **Polished Auth Portal (`client/login.html`):** Combines signup, login, and verification screens into a single-page card with animated backgrounds.
*   **Auth Guard:** Injected a redirect script at the top of the dashboard index page. If `mn_token` is missing from localStorage, it redirects the browser to `login.html`.
*   **Landing Page Redirects:** Pointed Log In and Get Started links on `maliknode.app` directly to `login.html?mode=login` and `login.html?mode=signup` respectively.

---

## 4. Pending / Next Steps Roadmap

To make this product fully operational, the following tasks remain:

1.  **Fetch API Bindings in Dashboard (`client/index.html`):**
    *   Write the Javascript `fetch()` operations to dynamically fetch active devices from `api.maliknode.app/api/peers`.
    *   Connect the "Add Peer" form to query `POST /api/peers`, fetch the output `.conf` text payload, and draw the scannable QR Code using the frontend canvas utility.
2.  **AdGuard Home API Integration:**
    *   Configure the API daemon to send REST requests to the user's specific AdGuard container API (on port `3000 + user_id`) when the user toggles category blocklists or SafeSearch in their dashboard settings.
3.  **Horizontal Multi-VPS Cluster Linking:**
    *   When user capacity grows, link your secondary 3-Core / 18GB RAM VPS droplet over the private network (`10.1.x.x`). 
    *   Shift the WireGuard UDP listeners and Docker container workloads to the secondary node, keeping the SQLite database and web controller on the primary VPS.
