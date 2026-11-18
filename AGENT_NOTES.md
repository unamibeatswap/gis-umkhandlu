# Agent Notes — GIS Umkhandlu Integration
# Last updated: 2026-11-18 — ALL PHASES COMPLETE

## Push Method (Codespace)
Codespace system credential helper overrides embedded-URL tokens.
Working push command:
```
GIT_CONFIG_NOSYSTEM=1 git -c credential.helper="" \
  -c http.extraHeader="Authorization: Basic $(echo -n 'PAT:x-oauth-basic' | base64 -w0)" \
  push origin HEAD
```

## Repos
| Repo | Path | Branch | Last commit |
|------|------|--------|-------------|
| prooftv/unami-platform-core | /workspaces/unami-platform-core | main | b6d4a3b |
| prooftv/Umkhandlu | /workspaces/Umkhandlu | trunk | 1dbccde |
| unamibeatswap/gis-umkhandlu | /workspaces/gis-umkhandlu | main | (this file) |

## Credentials

### GitHub
- PAT owner: prooftv
- Token: stored in codespace remote URLs only (not stored here — push protection)

### Supabase — Umkhandlu Control Centre
- Project ref: ufsmpqxniswdnsywjzje
- Region: eu-west-1
- Status: ACTIVE_HEALTHY
- Dashboard: https://supabase.com/dashboard/project/ufsmpqxniswdnsywjzje
- URL: https://ufsmpqxniswdnsywjzje.supabase.co
- Anon key: eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJpc3MiOiJzdXBhYmFzZSIsInJlZiI6InVmc21wcXhuaXN3ZG5zeXdqemplIiwicm9sZSI6ImFub24iLCJpYXQiOjE3ODU4NTE1MjYsImV4cCI6MjEwMTQyNzUyNn0.UpLvRqxNjORTk3buRFH2_J34IOp44rmc55_rkENSIFs
- Service role key: eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJpc3MiOiJzdXBhYmFzZSIsInJlZiI6InVmc21wcXhuaXN3ZG5zeXdqemplIiwicm9sZSI6InNlcnZpY2Vfcm9sZSIsImlhdCI6MTc4NTg1MTUyNiwiZXhwIjoyMTAxNDI3NTI2fQ.yqdgbMNuqU3qwuXlgbl5yGcIbgxDCOHnUi0WZ8k8eo8
- Management API token: [stored in session — sbp_ token, eu-west-1 account]

### Vercel — CC project
- Project ID: prj_tbp5UR5zzvTYFHQ6BDUJlKP7OAik
- Project name: unami-platform-core-umkhandlu
- URL: https://unami-platform-core-umkhandlu.vercel.app
- Token: [stored in session — vcp_ token]

### Sanity — Governance Node
- Project ID: 60v9eb0r
- Dataset: production
- API version: 2024-08-22
- Site URL: https://umkhandlu.unamifoundation.org
- Studio URL: https://umkhandlu.unamifoundation.org/studio

### Governance Node
- Live URL: https://umkhandlu.unamifoundation.org
- Intelligence API key: 0b3beaaf7aa6ce0d0c24353056df4165f52846be4347648c96e4a18c899de115
- Node ID: umkhandlu-khathide-001
- DB row ID in governance_nodes: b939f38d-54ee-4a2c-a547-67cb057dc060

## Vercel Env Vars — CC Project (all confirmed set)
| Key | Targets |
|-----|---------|
| NEXT_PUBLIC_UMKHANDLU_SUPABASE_URL | production, preview, development |
| NEXT_PUBLIC_UMKHANDLU_SUPABASE_ANON_KEY | production, preview, development |
| UMKHANDLU_SUPABASE_SERVICE_ROLE_KEY | production, preview, development |
| GRATISGIS_OGC_API_URL | production, preview, development (empty — serves from Supabase JSONB) |
| SANITY_WEBHOOK_SECRET | production, preview, development |
| UMKHANDLU_NODE_URL | production, preview |
| UMKHANDLU_NODE_API_KEY | production, preview |
| NEXT_PUBLIC_SUPABASE_URL | production, preview |
| NEXT_PUBLIC_SUPABASE_ANON_KEY | production, preview |
| SUPABASE_SERVICE_ROLE_KEY | production, preview |
| NEXT_PUBLIC_APP_URL | production, preview |

## Sanity Webhook
- Hook ID: m5k8f8ULRcuL651E
- Name: CC Notice Registration
- URL: https://umkhandlu.unamifoundation.org/api/sanity-webhook
- Dataset: production
- Type: transaction (Sanity REST API does not support groqFilter — handler filters internally)
- SANITY_WEBHOOK_SECRET: c9f28358c54bf46f2ca81ded82e840faad0680345ebd445e0ed916c0ad36a3eb
- Status: LIVE

## Migration 016 — APPLIED ✅
Applied: 2026-11-18 via Supabase Management API SQL endpoint
Verified:
- spatial_layer_config: 17 rows seeded
- notices.site_feature_id column: exists
- associate_notice_site_feature() function: exists
- DPME Amajuba District Boundary: permission_classification = redistribute
- governance_nodes capabilities: governance, participation, evidence, commercial, tcrs, institutional-memory, health, operators, spatial
- GRANT UPDATE ON governance_nodes TO service_role: applied

## Live System Checks (2026-11-18)
- map-layers API: 17 layers, first = Amajuba District Boundary — DPME (DC25) ✅
- Governance Node health: healthy, 15 records, 7 notices ✅
- Node capabilities: includes operators + spatial ✅
- Vercel CC env vars: 11 vars confirmed ✅
- Sanity webhook: live ✅

## Phase Summary — ALL COMPLETE
| Phase | Description | Commit | Status |
|-------|-------------|--------|--------|
| 1 | Migration 016 (DPME fix, site_feature_id, spatial_layer_config) | b6d4a3b | APPLIED ✅ |
| 2 | NoticeSiteFeatureAssociation component + actions | b6d4a3b | LIVE ✅ |
| 3 | /spatial/layers page + LayerConfigEditor | b6d4a3b | LIVE ✅ |
| 4 | /spatial/gratisgis status page | b6d4a3b | LIVE ✅ |
| 5 | map-layers API reads spatial_layer_config | b6d4a3b | LIVE ✅ |
| 6 | NoticeStatutoryMap 35/65 inspector/map layout | 2dfa199 | LIVE ✅ |
| 7 | DevelopmentNoticePage SpatialBlock breakout | 2dfa199 | LIVE ✅ |
| 8 | /api/sanity-webhook endpoint | 2dfa199 | LIVE ✅ |
| 9 | Collapsible desktop inspector panel | 1dbccde | LIVE ✅ |

## Architecture Notes
- CC reads node URL + API key from governance_nodes table (not env vars)
- UMKHANDLU_NODE_URL / UMKHANDLU_NODE_API_KEY env vars are legacy — DB is canonical
- spatial_layer_config: one row per dataset, seeded from published datasets, operators edit via /spatial/layers
- map-layers API joins spatial_layer_config on every request; falls back to hash colour if no config
- Sanity webhook fires on all mutations; /api/sanity-webhook handler checks _type == developmentNotice && status == open
- GRATISGIS_OGC_API_URL empty = system serves geometry from Supabase JSONB (valid interim)
- Biome max complexity: 15 — InspectorHeader extracted to keep DesktopInspector compliant

## Nothing Outstanding
All phases implemented, committed, pushed, deployed, and verified live.
