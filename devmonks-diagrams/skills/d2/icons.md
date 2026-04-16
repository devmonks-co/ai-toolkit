# D2 Icon Resolution Guide

Icons in D2 accept any publicly accessible image URL (SVG, PNG, JPG). This guide defines how to find the right icon URL for any node.

## Resolution Priority

Try sources in this order. **Always validate every URL with curl before using.**

```
1. Terrastruct  →  2. jsdelivr (@mdi/svg)  →  3. Official provider logo  →  4. Web search
```

### 1. Terrastruct Icons (preferred — native to D2)

**Base URL:** `https://icons.terrastruct.com/`

URL-encode slashes as `%2F`, spaces as `%20`, `&` as `%26`.

| Category | Prefix | Naming pattern | Example |
|----------|--------|----------------|---------|
| Dev/Tech logos | `dev/` | lowercase tech name | `dev%2Fdocker.svg`, `dev%2Fredis.svg`, `dev%2Fpostgresql.svg`, `dev%2Freact.svg`, `dev%2Fpython.svg` |
| Essentials | `essentials/` | `NNN-name` (numbered) | `essentials%2F112-server.svg`, `essentials%2F092-network.svg` |
| Infrastructure | `infra/` | `NNN-name` (numbered) | `infra%2F019-network.svg` |
| AWS | `aws/` | `Category/Service-Name` | `aws%2FCompute%2FAmazon-EC2.svg`, `aws%2FDatabase%2FAmazon-RDS.svg` |
| GCP | `gcp/` | `Products and services/Category/Name` | Exists but many return 403 — validate carefully |
| Azure | `azure/` | `Category/Service-Name` | Exists but many return 403 — validate carefully |
| Social | `social/` | `NNN-name` (numbered) | `social%2F039-github.svg` |

**Key notes:**
- The `dev/` category is the most reliable — use it for any technology that has a recognizable name (docker, redis, postgresql, react, go, python, nginx, etc.)
- `essentials/` and `infra/` use numbered prefixes — the exact number must be discovered (try the terrastruct icon browser or web search for "terrastruct icon <name>")
- AWS icons generally work but some subcategory paths return 403 — always validate
- GCP and Azure have lower availability — fall back quickly if 403

### 2. jsdelivr CDN (@mdi/svg)

**Base URL:** `https://cdn.jsdelivr.net/npm/@mdi/svg/svg/`

Pattern: `<icon-name>.svg` — lowercase, hyphen-separated.

Browse available icons at https://pictogrammers.com/library/mdi/ to find the right name. Good for generic/abstract icons (lock, email, cloud, database, server, etc.).

### 3. Official Provider Logos

For branded services (Stripe, Twilio, Vercel, etc.), look for official SVG logos:
- Check the provider's press/brand page
- Try `https://cdn.jsdelivr.net/npm/simple-icons/icons/<name>.svg`
- Try the provider's GitHub repo for logo assets

### 4. Web Search

As a last resort, search for `<service name> SVG icon` or `<service name> logo SVG`. Prefer SVG from reputable CDNs over random hosts.

## Validation

**Every icon URL must be validated before use:**

```bash
curl -sL -o /dev/null -w '%{http_code}' "<url>"
```

- `200` → use it
- Anything else → try the next source in the priority chain
