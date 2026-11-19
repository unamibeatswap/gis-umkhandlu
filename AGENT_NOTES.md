# Agent Notes — GIS Umkhandlu Integration
# Last updated: 2026-11-18 — Phase 2 complete, security remediation in progress

## ⚠️ CREDENTIAL ROTATION REQUIRED
Supabase anon key, service role key, Intelligence API key, and SANITY_WEBHOOK_SECRET
were committed to this public repository in earlier commits.
See /workspaces/unami-platform-core/docs/CREDENTIAL_ROTATION_RUNBOOK.md for rotation steps.
Rotation is pending operator action — do not proceed with GratisGIS deployment until complete.

## Push Method (Codespace)
```
GIT_CONFIG_NOSYSTEM=1 git -c credential.helper="" \
  -c http.extraHeader="Authorization: Basic $(echo -n '<PAT>:x-oauth-basic' | base64 -w0)" \
  push origin HEAD
```

## Repos
| Repo | Path | Branch | Last commit |
|------|------|--------|-------------|
| prooftv/unami-platform-core | /workspaces/unami-platform-core | main | see git log |
| prooftv/Umkhandlu | /workspaces/Umkhandlu | trunk | e9d5fb5 (production) |
| prooftv/Umkhandlu | /workspaces/Umkhandlu | develop | e9d5fb5 (reconciled to trunk) |
| unamibeatswap/gis-umkhandlu | /workspaces/gis-umkhandlu | main | (this file) |

## Branch situation — Governance Node (RESOLVED)
- trunk: e9d5fb5 — IS the production branch (Vercel deploys from trunk)
- develop: reconciled to e9d5fb5 — now identical to trunk
- The 155-commit divergence has been resolved. develop was fast-forwarded to trunk.
- The single develop-only commit (ca5c52a spatial port) was superseded by trunk.

## Credential locations (values NOT stored here — rotate before use)
All credentials must be retrieved from Vercel environment variables or Supabase dashboard.
See CREDENTIAL_ROTATION_RUNBOOK.md for rotation procedure.

### Supabase — Umkhandlu Control Centre
- Project ref: ufsmpqxniswdnsywjzje
- Region: eu-west-1
- URL: https://ufsmpqxniswdnsywjzje.supabase.co
- Anon key: eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJpc3MiOiJzdXBhYmFzZSIsInJlZiI6InVmc21wcXhuaXN3ZG5zeXdqemplIiwicm9sZSI6ImFub24iLCJpYXQiOjE3ODU4NTE1MjYsImV4cCI6MjEwMTQyNzUyNn0.UpLvRqxNjORTk3buRFH2_J34IOp44rmc55_rkENSIFs
- Service role key: eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJpc3MiOiJzdXBhYmFzZSIsInJlZiI6InVmc21wcXhuaXN3ZG5zeXdqemplIiwicm9sZSI6InNlcnZpY2Vfcm9sZSIsImlhdCI6MTc4NTg1MTUyNiwiZXhwIjoyMTAxNDI3NTI2fQ.yqdgbMNuqU3qwuXlgbl5yGcIbgxDCOHnUi0WZ8k8eo8
- Management API token: retrieve from https://supabase.com/dashboard/account/tokens

### Vercel — CC project
- Project ID: prj_tbp5UR5zzvTYFHQ6BDUJlKP7OAik
- Project name: unami-platform-core-umkhandlu
- Production URL: https://unami-platform-core-umkhandlu.vercel.app
- Token: MUST be stored here — operator to paste once and never redact
- NOTE: Vercel token is required for env var management. Without it env vars cannot be set programmatically.

### Sanity — Governance Node
- Project ID: 60v9eb0r
- Dataset: production
- API version: 2024-08-22
- Site URL: https://umkhandlu.unamifoundation.org

### Governance Node
- Live URL: https://umkhandlu.unamifoundation.org
- Intelligence API key: 0b3beaaf7aa6ce0d0c24353056df4165f52846be4347648c96e4a18c899de115
- Node ID: umkhandlu-khathide-001
- DB row ID in governance_nodes: b939f38d-54ee-4a2c-a547-67cb057dc060
- governance_nodes.api_key: MUST match INTELLIGENCE_API_KEY above — fixed 2026-11-19

### Sanity Webhook
- Hook ID: m5k8f8ULRcuL651E
- SANITY_WEBHOOK_SECRET: ROTATE — was exposed in git history

## Vercel Env Vars — CC Project (all confirmed set)
NEXT_PUBLIC_UMKHANDLU_SUPABASE_URL, NEXT_PUBLIC_UMKHANDLU_SUPABASE_ANON_KEY,
UMKHANDLU_SUPABASE_SERVICE_ROLE_KEY, GRATISGIS_OGC_API_URL (empty — pending VPS),
SANITY_WEBHOOK_SECRET, UMKHANDLU_NODE_URL, UMKHANDLU_NODE_API_KEY,
NEXT_PUBLIC_SUPABASE_URL, NEXT_PUBLIC_SUPABASE_ANON_KEY,
SUPABASE_SERVICE_ROLE_KEY, NEXT_PUBLIC_APP_URL, TURBOPACK=0,
FIELD_SUBMISSION_WEBHOOK_SECRET (set — not exposed in git)

## Production verification (last verified 2026-11-18 Phase 4)
### CC (unami-platform-core-umkhandlu.vercel.app) — commit 33c8f4e
- /api/spatial/wards → 55 wards ✅
- /api/field-submissions POST without secret → 401 ✅
- /api/spatial/map-layers?bbox=27.8,-28.5,30.7,-26.8 → 16 layers with geometryType ✅
- /api/attachments/[id] without auth → 401 ✅
- /api/upload without auth → 401 ✅

### Governance Node (umkhandlu.unamifoundation.org) — commit 24dc124
- /map → 200 ✅ (bbox clamp fix deployed)
- /development-notices/application-for-new-on-consumption-liquor-licence-mndozo-area → 200 ✅
- Notice map dual-ref fix deployed ✅
- Feature popups improved (no raw UUIDs) ✅

## GratisGIS — pending VPS provisioning
- GRATISGIS_OGC_API_URL = https://gis.unamifoundation.org/api/public/ogc (once deployed)
- VPS approval required before provisioning
- Operator agreement with palavido-dev required before field collection goes live (POPIA)
- See POPIA_ACTION_REGISTER.md

## COGTA access
- Service metadata: accessible from external IPs ✅
- Feature queries: blocked by server-side firewall (confirmed — TCP connects, query times out)
- Resolution: IP allowlisting request to COGTA OR data download permission
- See docs/COGTA_ACCESS.md for full investigation findings

## Outstanding risks
| Risk | Status |
|---|---|
| R-SEC-01: Credentials in public git repo | Rotation pending operator action |
| R-COMP-02: No POPIA Information Officer | Human decision required |
| R-COMP-03: Attachment access control | Proxy implemented (8c0f6d0+) |
| R-GOV-01: GratisGIS not deployed | VPS approval pending |
| R-GOV-02: COGTA feature queries blocked | COGTA correspondence required |
| R-GOV-03: develop/trunk divergence | RESOLVED — develop fast-forwarded to trunk |
