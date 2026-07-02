# Nyxite Deploy — Contents

Self-hosting: container stack, reverse proxy, DNS, VPN, and provisioning. Graduates from this private repo's `docs/INFRASTRUCTURE.md`.

## Contents

- docker-compose for the full stack (server, PostgreSQL, Keycloak, blob store)
- nginx reverse proxy configuration
- Cloudflare configuration (proxy, Universal SSL for public users, Origin CA internally)
- bind9 internal DNS configuration
- WireGuard VPN configuration (admin-only access)
- Hosting the `NyxiteAdmin` operator dashboard (Next.js SSR) as its own container, bound to the WireGuard interface only (admin-only)
- Hetzner VPS (ARM64) provisioning
- Origin CA root certificate installation notes for admin devices (Linux NSS via certutil, Windows certlm.msc, Android Settings -> Security)

## Open questions

See [../docs/OPEN-DECISIONS.md](../docs/OPEN-DECISIONS.md). Deploy-specific:

- Server-side secret injection (Keycloak client secret, DB credentials). Note: under full E2EE there is **no server-held content KEK** — content keys live on clients.
- Backup strategy for the database and blob store (both ciphertext-only; useless without a member's key). The critical secret is each user's recovery key, held by users, not the operator — server DR cannot recover content if users lose their keys.
- Single self-contained binary vs container as the primary deployment target
