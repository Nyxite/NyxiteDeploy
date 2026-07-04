# Nyxite Deploy — Contents

Self-hosting: container stack, reverse proxy, DNS, VPN, and provisioning. Graduates from this private repo's `docs/INFRASTRUCTURE.md`.

## Contents

- docker-compose for the default stack (server, PostgreSQL, blob store) behind nginx; **Keycloak is an optional `enterprise` compose profile**, not part of the default (native auth is the default — #9)
- nginx reverse proxy configuration
- Cloudflare configuration (proxy, Universal SSL for public users, Origin CA internally)
- bind9 internal DNS configuration
- WireGuard VPN configuration (admin-only access)
- Hosting the `NyxiteAdmin` operator dashboard (Next.js SSR) as its own container, bound to the WireGuard interface only (admin-only)
- Hetzner VPS (ARM64) provisioning
- Origin CA root certificate installation notes for admin devices (Linux NSS via certutil, Windows certlm.msc, Android Settings -> Security)

## Open questions

See [../docs/OPEN-DECISIONS.md](../docs/OPEN-DECISIONS.md). Deploy-specific:

- Server-side secret injection (DB credentials; the native-auth signing key for the server's own tokens; the Keycloak client secret **only when the enterprise profile is enabled**). Note: under full E2EE there is **no server-held content KEK** — content keys live on clients.
- Backup strategy for the database and blob store (both ciphertext-only; useless without a member's key). The critical secret is each user's recovery phrase, held by users, not the operator — server DR cannot recover content if users lose their keys (the client-encrypted recovery blob on the server is unreadable without that phrase).
- Deployment target — **resolved**: the Docker Compose container stack is the primary self-hosting artifact; a single self-contained binary is not a v1 target
