# Passbolt Stack

Traefik v3 + Passbolt CE in a single compose stack.

Traefik is the only container that needs to be network-aware across all stacks, so it lives here and the `proxy` network is defined as an external-style named network that other stacks can join.

## Prerequisites

- Docker + Docker Compose
- A domain managed by Cloudflare (for DNS challenge SSL)
- A Cloudflare API token with `Zone:DNS:Edit` permission

## Setup

**1. Create the proxy network** (once, before first `up`):

```bash
docker network create --subnet 172.16.100.0/24 proxy
```

Or let compose create it — it's defined in the compose file.

**2. Create the acme.json cert store**:

```bash
mkdir -p traefik/data
touch traefik/data/acme.json
chmod 600 traefik/data/acme.json
```

Traefik will not start if `acme.json` is missing or has incorrect permissions.

**3. Copy and fill the env file**:

```bash
cp .env.example .env
# edit .env with your values
```

**4. Place Traefik configs**:

Copy `traefik.yml` and `dynamic.yml` from the `traefik/` directory at the root of this repo into `traefik/data/` here.

**5. Bring up the stack**:

```bash
docker compose up -d
```

## First Passbolt Admin User

After the stack is healthy, create the first admin:

```bash
docker exec passbolt su -m -c \
  "/usr/share/php/passbolt/bin/cake passbolt register_user \
  -u your-email@example.com \
  -f First -l Last -r admin" -s /bin/sh www-data
```

## Notes

- Traefik depends on Passbolt being healthy before it starts — this avoids 502s during initial Passbolt startup
- Passbolt is on two networks: `proxy` (Traefik ingress) and `passbolt_internal` (MariaDB, isolated)
- MariaDB is on `passbolt_internal` only — not reachable from the proxy network
- `PASSBOLT_REGISTRATION_PUBLIC: "true"` — lock this down after creating your admin user if you don't want open registration
- GPG and JWT keys are stored in named volumes; back these up or Passbolt loses the ability to decrypt existing passwords
