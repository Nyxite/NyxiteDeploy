# Nyxite Deploy — Contents

Self-hosting: container stack, reverse proxy, DNS, VPN, and provisioning. Graduates from this private repo's `docs/INFRASTRUCTURE.md`.

## Contents

- docker-compose for the default stack (server, PostgreSQL, blob store) behind nginx; **Keycloak is an optional `enterprise` compose profile**, not part of the default (native auth is the default — #9)
- nginx reverse proxy configuration
- Cloudflare configuration (proxy, Universal SSL for public users, Origin CA internally)
- bind9 internal DNS configuration
- WireGuard VPN configuration (admin-only access)
- Hosting the `NyxiteAdmin` operator dashboard (Next.js SSR) as its own container, bound to the WireGuard interface only (admin-only)
- **License token as config (self-hosted stack).** A licensed operator supplies their per-instance license token via `NYXITE_LICENSE_TOKEN` (env var / mounted secret), dropped straight into the compose `environment:` block or `.env`. It is **optional** — absent = community mode (free non-commercial, all core features, group cap 16). The server verifies it **offline** against an embedded **hybrid Ed25519 + ML-DSA-65** public key (both signature halves); the deploy never calls the license server for boot. Air-gapped customers install a manually-issued **offline license** (time-boxed, no check-in) the same way. (See [features/license.md](https://github.com/Nyxite/Nyxite/blob/main/features/license.md), L-1–L-7.)
- **Vendor license server (`NyxiteLicense`) — separate, isolated infra, NOT part of this stack.** The maintainer deploys it independently (its own host/container, no network path to any customer content plane); it is publicly reachable only for the check-in (`/register`) and revocation feed. Self-hosters never run it.
- **Vendor helpdesk (`NyxiteSupport`) — isolated infra, NOT part of the default stack.** Like `NyxiteLicense`, the central bug-reporting / ticketing helpdesk is run by the maintainer as isolated vendor infra (its own host/container, no path to any content plane). By default a self-hosted server only **relays** authenticated reports to that **central** desk: **every** instance shows the report UI and routes upstream (SUP-9), non-silently via the SUP-1 consent notice. Server-side relay config: `NYXITE__Support__Enabled`, `__ServiceBaseUrl` (central desk), `__InstanceCredential`. **Enterprise opt-in (SUP-10–SUP-13):** a licensed operator may instead run **their own desk** by adding the `NyxiteSupport` **`docker-compose.override.yml`** to their stack (the base compose files never name it) and pointing `__ServiceBaseUrl` at the local desk — then only **confirmed bugs** route upstream. The self-hosted desk is **enterprise-gated at runtime** (a valid entitlement lease, not the file's presence — SUP-11) and degrades/locks with the other enterprise features on a licence lapse. It runs on the consensual non-E2EE support plane, disjoint from content. See [features/support.md](https://github.com/Nyxite/Nyxite/blob/main/features/support.md).
- Hetzner VPS (ARM64) provisioning
- Origin CA root certificate installation notes for admin devices (Linux NSS via certutil, Windows certlm.msc, Android Settings -> Security)

## Open questions

See [../docs/OPEN-DECISIONS.md](https://github.com/Nyxite/Nyxite/blob/main/docs/OPEN-DECISIONS.md). Deploy-specific:

- Server-side secret injection (DB credentials; the native-auth signing key for the server's own tokens; the **`PASSWORD_PEPPER`** secret — a rotatable server-side secret held **outside Postgres**, HMAC-SHA256-pre-hashed into the account-password Argon2id verifier so a **DB-only leak yields uncrackable verifiers**; the Keycloak client secret **only when the enterprise profile is enabled**). Note: under full E2EE there is **no server-held content KEK** — content keys live on clients, and the client-side content/E2EE asymmetric crypto is **hybrid post-quantum at v1.0.0** (X25519 + ML-KEM-768 key wrap; Ed25519 + ML-DSA-65 signatures; symmetric AES-256-GCM / Argon2id / BLAKE3 unchanged), none of which the server ever holds.
- Backup strategy for the database and blob store (both ciphertext-only; useless without a member's key). The critical secret is each user's recovery phrase, held by users, not the operator — server DR cannot recover content if users lose their keys (the client-encrypted recovery blob on the server is unreadable without that phrase).
- Deployment target — **resolved**: the Docker Compose container stack is the primary self-hosting artifact; a single self-contained binary is not a v1 target
