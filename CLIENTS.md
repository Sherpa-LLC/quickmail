# Sherpa multi-client QuickMail

This is Sherpa's fork of [DivinPrince/quickmail](https://github.com/DivinPrince/quickmail).
One codebase, one Worker deploy **per client**, each driven by its own root-level
`wrangler.<client>.jsonc` (root-level because `main`/`assets` paths resolve relative
to the config file). The upstream `wrangler.jsonc` stays untouched so pulls never conflict.

## Current clients

| Client | Config | Worker | Mail domain | Web UI |
| --- | --- | --- | --- | --- |
| Mission Ready Repairs | `wrangler.mission-ready.jsonc` | `quickmail-mission-ready` | missionready.repair | https://mail.missionready.repair |

## Add a client

1. Copy an existing `wrangler.<client>.jsonc`, change `name`, `routes` pattern,
   `CLOUDFLARE_MAIL_DOMAINS`, and the D1/R2 names.
2. Create resources and fill in the D1 id:
   ```bash
   bunx wrangler d1 create quickmail-<client>
   bunx wrangler r2 bucket create quickmail-<client>-attachments
   ```
3. Migrate: `bunx wrangler d1 migrations apply quickmail-<client> -c wrangler.<client>.jsonc --remote`
4. Onboard the zone (Cloudflare DNS, Workers paid plan required for sending):
   ```bash
   bunx wrangler email sending enable <domain>
   bunx wrangler email routing enable <domain>
   ```
5. Deploy: `bun run build && bunx wrangler deploy -c wrangler.<client>.jsonc`
6. Dashboard (CLI can't do this): Email Routing → catch-all → "Send to a Worker" → the client's Worker.
   Review existing forward rules first — specific-address forwards take precedence over the catch-all.
7. Open `https://mail.<domain>/setup`, create the admin account (a Sherpa
   @yoursherpa.guide login), then add the client's user account.

## Pull upstream updates

```bash
git fetch upstream
git merge upstream/main
bun install
# redeploy every client:
bun run build
for c in wrangler.*.jsonc; do bunx wrangler deploy -c "$c"; done
# apply any new migrations per client:
bunx wrangler d1 migrations apply quickmail-<client> -c wrangler.<client>.jsonc --remote
```
