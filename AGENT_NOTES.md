# Agent Notes — GIS Umkhandlu Integration
# Last updated: 2026-11-18 — Phase 2 spatial fixes committed and deploying

## Push Method (Codespace)
```
GIT_CONFIG_NOSYSTEM=1 git -c credential.helper="" \
  -c http.extraHeader="Authorization: Basic $(echo -n 'PAT:x-oauth-basic' | base64 -w0)" \
  push origin HEAD
```

## Repos
| Repo | Path | Branch | Last commit |
|------|------|--------|-------------|
| prooftv/unami-platform-core | /workspaces/unami-platform-core | main | 7162859 |
| prooftv/Umkhandlu | /workspaces/Umkhandlu | trunk | e9d5fb5 |
| unamibeatswap/gis-umkhandlu | /workspaces/gis-umkhandlu | main | (this file) |

## Credentials

### Supabase — Umkhandlu Control Centre
- Project ref: ufsmpqxniswdnsywjzje
- Region: eu-west-1
- URL: https://ufsmpqxniswdnsywjzje.supabase.co
- Anon key: eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJpc3MiOiJzdXBhYmFzZSIsInJlZiI6InVmc21wcXhuaXN3ZG5zeXdqemplIiwicm9sZSI6ImFub24iLCJpYXQiOjE3ODU4NTE1MjYsImV4cCI6MjEwMTQyNzUyNn0.UpLvRqxNjORTk3buRFH2_J34IOp44rmc55_rkENSIFs
- Service role key: eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJpc3MiOiJzdXBhYmFzZSIsInJlZiI6InVmc21wcXhuaXN3ZG5zeXdqemplIiwicm9sZSI6InNlcnZpY2Vfcm9sZSIsImlhdCI6MTc4NTg1MTUyNiwiZXhwIjoyMTAxNDI3NTI2fQ.yqdgbMNuqU3qwuXlgbl5yGcIbgxDCOHnUi0WZ8k8eo8
- Management API token: [sbp_ token — stored in session]

### Vercel — CC project
- Project ID: prj_tbp5UR5zzvTYFHQ6BDUJlKP7OAik
- Project name: unami-platform-core-umkhandlu
- URL: https://unami-platform-core-umkhandlu.vercel.app
- Token: [vcp_ token — stored in session]

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

## Vercel Env Vars — CC Project (all confirmed set)
NEXT_PUBLIC_UMKHANDLU_SUPABASE_URL, NEXT_PUBLIC_UMKHANDLU_SUPABASE_ANON_KEY,
UMKHANDLU_SUPABASE_SERVICE_ROLE_KEY, GRATISGIS_OGC_API_URL (empty),
SANITY_WEBHOOK_SECRET, UMKHANDLU_NODE_URL, UMKHANDLU_NODE_API_KEY,
NEXT_PUBLIC_SUPABASE_URL, NEXT_PUBLIC_SUPABASE_ANON_KEY,
SUPABASE_SERVICE_ROLE_KEY, NEXT_PUBLIC_APP_URL

## Sanity Webhook
- Hook ID: m5k8f8ULRcuL651E
- SANITY_WEBHOOK_SECRET: c9f28358c54bf46f2ca81ded82e840faad0680345ebd445e0ed916c0ad36a3eb
- Status: LIVE

## Phase Summary
| Phase | Description | Commit | Status |
|-------|-------------|--------|--------|
| 1 | Migration 016 applied | b6d4a3b | LIVE ✅ |
| 2 | NoticeSiteFeatureAssociation | b6d4a3b | LIVE ✅ |
| 3 | /spatial/layers + LayerConfigEditor | b6d4a3b | LIVE ✅ |
| 4 | /spatial/gratisgis status page | b6d4a3b | LIVE ✅ |
| 5 | map-layers API reads spatial_layer_config | b6d4a3b | LIVE ✅ |
| 6 | NoticeStatutoryMap 35/65 layout | 2dfa199 | LIVE ✅ |
| 7 | DevelopmentNoticePage SpatialBlock | 2dfa199 | LIVE ✅ |
| 8 | /api/sanity-webhook endpoint | 2dfa199 | LIVE ✅ |
| 9 | Collapsible desktop inspector | 1dbccde | LIVE ✅ |
| 10 | Spatial rendering fixes (Phase 2) | 7162859 / e9d5fb5 | DEPLOYING ⏳ |

## Phase 2 — Spatial Rendering Root Cause Analysis

### Bug 1: /map bbox error
- Root cause: MAX_VIEWPORT_SPAN_DEGREES was 2. Full Amajuba district is ~2.9° wide.
  GovernanceMapLeaflet calls fitBounds(coverage) on all 55 wards, then immediately
  fires updateViewport() capturing the full district extent (~2.9° × ~1.7°).
  This exceeded the 2° limit → "bbox must be a valid WGS84 extent no wider than 2 degrees."
- Fix: Raised MAX_VIEWPORT_SPAN_DEGREES from 2 to 5 in map-layers/route.ts (7162859)
- Rationale: 5° still prevents runaway global queries; Amajuba (~2.9°) fits comfortably.

### Bug 2: Blank Notice map
- Root cause: Leaflet initialises into a flex container that has not yet been painted.
  The map container has zero dimensions at the time createMap() runs.
  Leaflet renders tiles but cannot calculate tile positions → blank white canvas.
- Fix: Added requestAnimationFrame(() => map.invalidateSize()) after map creation.
  Added ResizeObserver to re-invalidate when inspector collapses/expands (e9d5fb5).
- Same fix applied to GovernanceMapLeaflet for consistency.

### Bug 3: Perpetual "Loading…" layer states
- Root cause: Layer status label in DesktopInspector only checked returnedCount,
  not requestStatus. Layers with requestStatus='not-requested' or 'error' showed "Loading…".
- Fix: Extracted layerStatusText() helper; shows Loading/Unavailable/Not loaded/
  N in view/No features/None in view based on requestStatus + status (e9d5fb5).

### API verification (pre-fix)
- bbox=29.8,-27.8,30.2,-27.4 → 17 layers, returnedCount=1 for first layer ✅
- bbox=27.8,-28.5,30.7,-26.8 (full district) → rejected with 2° error (now fixed)
- All 42,939 approved features confirmed in DB
- Geometry format: GeoJSON Polygon/MultiPolygon, WGS84, lng/lat order ✅

## GratisGIS Status
- GRATISGIS_OGC_API_URL: empty in Vercel (not configured)
- No self-hosted production instance confirmed
- gratisgis-client.ts exists and is fully implemented (OGC API client)
- /spatial/gratisgis page shows connection status to operators
- Current geometry serving: Supabase JSONB (42,939 approved features, 17 datasets)
- GratisGIS field PWA: NOT YET VERIFIED — requires production instance URL

## Next Phases (from brief)
- Phase 3: Verify actual GratisGIS deployment (need production URL from operator)
- Phase 4: Define collection architecture
- Phase 5: Integrate shared Field Collection PWA
- Phase 6: Complete operator workflows in CC
- Phase 7: Complete public experiences in Governance Node
- Phase 8: End-to-end verification
