# Traefik

Traefik v3 running in Docker, acting as the reverse proxy for all services — both Docker-based (via labels) and non-Docker (via file provider).

## Files

- `traefik.yml` — static config: entrypoints, providers, cert resolver
- `dynamic.yml` — file provider config: routes and services for non-Docker hosts (Proxmox, TrueNAS, ARM, etc.)

## Expected host layout

```
traefik-stack/
├── docker-compose.yml
└── traefik/
    └── data/
        ├── traefik.yml    ← bind-mounted to /traefik.yml in container
        ├── dynamic.yml    ← bind-mounted to /dynamic.yml in container
        └── acme.json      ← cert storage (chmod 600, gitignored)
```

## Cloudflare DNS Challenge

SSL certs are issued via Cloudflare DNS challenge — no port 80 exposure needed. Set `CF_DNS_API_TOKEN` in your stack `.env`.

The token needs `Zone:DNS:Edit` permission for your domain zone.

## File Provider Notes

`watch: true` enables hot-reload of `dynamic.yml` — changes apply without restarting the container.

**Critical**: any YAML syntax error or unrecognized field name in `dynamic.yml` causes the entire file to be rejected. All file-provider routes go offline simultaneously. Docker label routes are unaffected. Always validate before saving.

## Adding a non-Docker service

Add a router + service block to `dynamic.yml`:

```yaml
http:
  routers:
    myservice:
      rule: "Host(`myservice.yourdomain.com`)"
      entryPoints:
        - websecure
      tls:
        certResolver: cloudflare
      service: myservice

  services:
    myservice:
      loadBalancer:
        servers:
          - url: "http://192.168.1.x:PORT"
```

For upstream services with self-signed certs (Proxmox, TrueNAS), use `passHostHeader: true` and the named `insecureTransport` serversTransport.
