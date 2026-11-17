# Agent Notes — GIS Umkhandlu Integration

## PAT / Auth
- **GitHub PAT (prooftv)**: stored in codespace remote URLs — retrieve from GitHub → Settings → Developer Settings → PAT
- Codespace system credential helper (`/.codespaces/bin/gitcredential_github.sh`) overrides embedded-URL tokens.
- **Working push method**: `GIT_CONFIG_NOSYSTEM=1 git -c credential.helper="" -c http.extraHeader="Authorization: Basic <base64(PAT:x-oauth-basic)>" push origin HEAD`

## Repos
| Repo | Path | Branch | Last pushed commit |
|------|------|--------|--------------------|
| prooftv/unami-platform-core | /workspaces/unami-platform-core | main | b6d4a3b |
| prooftv/Umkhandlu | /workspaces/Umkhandlu | trunk | 2dfa199 |

## Supabase
- **Project**: `ufsmpqxniswdnsywjzje` (Umkhandlu Control Centre)
- **Dashboard**: https://supabase.com/dashboard/project/ufsmpqxniswdnsywjzje
- **Migration 016** (`016_dpme_permission_and_notice_spatial.sql`) — **NOT YET APPLIED**
  - File: `/workspaces/unami-platform-core/apps/umkhandlu/supabase/migrations/016_dpme_permission_and_notice_spatial.sql`
  - Apply via: Supabase Dashboard → SQL Editor → paste and run
  - What it does: DPME permission fix, `notices.site_feature_id` columns, `associate_notice_site_feature()` function, `spatial_layer_config` table + seed
- Env var prefix: `NEXT_PUBLIC_UMKHANDLU_` and `UMKHANDLU_`
- **Credentials needed from operator**: `NEXT_PUBLIC_UMKHANDLU_SUPABASE_URL`, `NEXT_PUBLIC_UMKHANDLU_SUPABASE_ANON_KEY`, `UMKHANDLU_SUPABASE_SERVICE_ROLE_KEY`

## Vercel
- **CC project**: `umkhandlu` (deploys from `prooftv/unami-platform-core`, `apps/umkhandlu`)
- **Governance Node project**: deploys from `prooftv/Umkhandlu`
- **Vercel CLI**: available via `npx vercel@65.17.0`
- **Token needed**: `VERCEL_TOKEN` (from vercel.com/account/tokens)
- **Env vars to add to CC Vercel project** (all environments):
  - `GRATISGIS_OGC_API_URL` — OGC API root URL for GratisGIS (leave blank until persistent deployment)
  - `SANITY_WEBHOOK_SECRET` — random secret matching Sanity webhook config

## Sanity Webhook
- **Endpoint**: `https://umkhandlu.unamifoundation.org/api/sanity-webhook`
- **Filter**: `_type == "developmentNotice" && status == "open"`
- **Secret**: must match `SANITY_WEBHOOK_SECRET` Vercel env var
- **Status**: endpoint code deployed, webhook NOT yet configured in Sanity Studio

## Remaining Blockers (as of last session)
1. ✅ GitHub push — DONE (both repos pushed via Authorization header method)
2. ⬜ Migration 016 — paste into Supabase SQL editor for `ufsmpqxniswdnsywjzje`
3. ⬜ `GRATISGIS_OGC_API_URL` — add to CC Vercel project (can be empty string initially)
4. ⬜ `SANITY_WEBHOOK_SECRET` — add to CC Vercel project, then configure Sanity webhook

## Phase Summary
- Phase 1: Migration 016 (DPME fix, site_feature_id, spatial_layer_config)
- Phase 2: NoticeSiteFeatureAssociation component + actions
- Phase 3: /spatial/layers page + LayerConfigEditor + upsertLayerConfigAction
- Phase 4: /spatial/gratisgis status page + GratisGisDiscoverButton
- Phase 5: map-layers API reads spatial_layer_config
- Phase 6: NoticeStatutoryMap 35/65 inspector/map layout
- Phase 7: DevelopmentNoticePage SpatialBlock breakout
- Phase 8: /api/sanity-webhook endpoint

## Build Status
- CC (`unami-platform-core/apps/umkhandlu`): `tsc --noEmit` clean, `next build` successful
- Governance Node (`Umkhandlu`): `tsc --noEmit` clean, biome passes; `next build` fails only on pre-existing `campaigns/export/print` route (Sanity env not set in codespace — unrelated)
