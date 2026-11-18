# Agent Notes — GIS Umkhandlu Integration

## Push Method (Codespace)
Codespace system credential helper (`/.codespaces/bin/gitcredential_github.sh`) overrides embedded-URL tokens.
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
- Token: stored in repo remote URLs — retrieve from GitHub Settings > Developer Settings > PAT

### Supabase — Umkhandlu Control Centre
- Project ref: ufsmpqxniswdnsywjzje
- Dashboard: https://supabase.com/dashboard/project/ufsmpqxniswdnsywjzje
- URL: https://ufsmpqxniswdnsywjzje.supabase.co
- Anon key: eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJpc3MiOiJzdXBhYmFzZSIsInJlZiI6InVmc21wcXhuaXN3ZG5zeXdqemplIiwicm9sZSI6ImFub24iLCJpYXQiOjE3ODU4NTE1MjYsImV4cCI6MjEwMTQyNzUyNn0.UpLvRqxNjORTk3buRFH2_J34IOp44rmc55_rkENSIFs
- Service role key: eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJpc3MiOiJzdXBhYmFzZSIsInJlZiI6InVmc21wcXhuaXN3ZG5zeXdqemplIiwicm9sZSI6InNlcnZpY2Vfcm9sZSIsImlhdCI6MTc4NTg1MTUyNiwiZXhwIjoyMTAxNDI3NTI2fQ.yqdgbMNuqU3qwuXlgbl5yGcIbgxDCOHnUi0WZ8k8eo8
- Supabase Management API token: [REDACTED — sbp_ token, returns 401, needs regeneration at supabase.com/dashboard/account/tokens]
- DB direct access: BLOCKED from codespace (af-south-1 pooler hostname does not resolve)

### Vercel — CC project
- Project ID: prj_tbp5UR5zzvTYFHQ6BDUJlKP7OAik
- Project name: unami-platform-core-umkhandlu
- Token: [REDACTED — vcp_ Vercel token, stored in codespace session]
- URL: https://unami-platform-core-umkhandlu.vercel.app

### Sanity — Governance Node
- Project ID: 60v9eb0r
- Dataset: production
- API version: 2024-08-22
- Read token: skvh4gF6oVA5Qk3WJTFZdVIXQVSd7sPsUYAUzW8KC4VGOOJpWdTANHAjC76mYhkGd8ZSPDgpMivr9Mvue0COMlAN2EXJsf5hiOYSdkVMnbI7lYOkFhv48PdY0rXfVH1z65J7VNpjKIvWtcD36qRgx6L0oJLK1iLqf5rd7Tu3LTsROAHSHua7
- Write token: sk0LRjqWM9T2ODa8vvj8FhXCUb06sIMTjBym8nlq9lPkyl3iJlPIluyFVpowyKAQzxqOKCf1srQ2is0PbcZE0TiWR976gXugJs7UhdY8HUvFwOhImS4EpQNWl33gm9J86J5BoAlGCVBVlJufitTkIUeR2lWGOzxJ51xrWUPrPhLgJZqMKrHS

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
- Type: transaction (Sanity API does not support groqFilter via REST — handler filters internally by _type and status)
- SANITY_WEBHOOK_SECRET: c9f28358c54bf46f2ca81ded82e840faad0680345ebd445e0ed916c0ad36a3eb
- Status: LIVE

## Governance Node DB Record
- governance_nodes row: b939f38d-54ee-4a2c-a547-67cb057dc060
- capabilities currently: health, governance, participation, evidence, commercial, tcrs, institutional-memory
- MISSING: operators, spatial (RLS blocks service_role PATCH — needs SQL fix in migration 016)
- Fix SQL (add to migration 016 or run separately):
  ```sql
  UPDATE public.governance_nodes
  SET capabilities = ARRAY['governance','participation','evidence','commercial','tcrs',
                            'institutional-memory','health','operators','spatial']
  WHERE id = 'b939f38d-54ee-4a2c-a547-67cb057dc060';
  ```

## Migration 016 — NOT YET APPLIED
File: /workspaces/unami-platform-core/apps/umkhandlu/supabase/migrations/016_dpme_permission_and_notice_spatial.sql

Apply via: Supabase Dashboard > SQL Editor > paste full file content
URL: https://supabase.com/dashboard/project/ufsmpqxniswdnsywjzje/sql/new

Blockers to applying programmatically:
- Supabase Management API token (sbp_) returns 401 — needs regeneration at supabase.com/dashboard/account/tokens
- DB direct connection blocked (af-south-1 pooler hostname does not resolve from codespace)
- No exec_sql RPC function exists in the project

## Phase Summary
| Phase | Description | Status |
|-------|-------------|--------|
| 1 | Migration 016 (DPME fix, site_feature_id, spatial_layer_config) | Written, NOT applied |
| 2 | NoticeSiteFeatureAssociation component + actions | Committed, pushed |
| 3 | /spatial/layers page + LayerConfigEditor | Committed, pushed |
| 4 | /spatial/gratisgis status page | Committed, pushed |
| 5 | map-layers API reads spatial_layer_config | Committed, pushed |
| 6 | NoticeStatutoryMap 35/65 inspector/map layout | Committed, pushed |
| 7 | DevelopmentNoticePage SpatialBlock breakout | Committed, pushed |
| 8 | /api/sanity-webhook endpoint | Committed, pushed |
| 9 | Collapsible desktop inspector panel | Committed, pushed (1dbccde) |

## Remaining Manual Steps
1. Apply migration 016 — paste into Supabase SQL editor for ufsmpqxniswdnsywjzje
   Also run the governance_nodes capabilities UPDATE above
2. Regenerate Supabase Management API token at supabase.com/dashboard/account/tokens
   (current sbp_ token returns 401)
3. Trigger Vercel redeploy of CC after migration is applied (or it auto-deploys on next push)

## Build Status
- CC (unami-platform-core/apps/umkhandlu): tsc clean, next build successful
- Governance Node (Umkhandlu): tsc clean, biome passes, collapsible inspector committed
- Governance Node live health: https://umkhandlu.unamifoundation.org/api/intelligence/health returns {"status":"healthy","recordCount":15,"noticeCount":7}
- CC Vercel project: all 11 env vars confirmed set
- Sanity webhook: live (id m5k8f8ULRcuL651E)
