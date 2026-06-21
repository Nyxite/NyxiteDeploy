# Nyxite Deploy — Contents

Self-hosting: container stack, reverse proxy, DNS, VPN, and provisioning.

## Contents

- docker-compose for the full stack (server, database, Keycloak)
- Cloudflare configuration (proxy, Universal SSL for public users, Origin CA internally)
- bind9 internal DNS configuration
- WireGuard VPN configuration (admin-only access)
- Hetzner VPS provisioning
- Origin CA root certificate installation notes for admin devices (Linux NSS via certutil, Windows certlm.msc, Android Settings -> Security)

## Open questions

- Secret injection for the KEK: environment secret now vs Vault later (mechanism owned here; ties to the server decision)
- Backup strategy for the database, file storage, and the master key
- Single self-contained binary vs container as the primary deployment target
