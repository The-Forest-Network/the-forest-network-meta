## Push Notifications (Sygnal, APNs, FCM)

### Architecture

`tfn-synapse` → client's registered pusher URL → push gateway → APNs (iOS) / FCM (Android) → device.

`tfn-sygnal` is already deployed (`docker-compose.yml`) as the self-hosted push gateway, with APNs configured for both production and sandbox (`.p8` keys mounted from `/media-server/config/tfn/sygnal/`). It has no exposed port — it only sits on the `proxy` Docker network.

Element X does **not** use this self-hosted sygnal at all — it registers its pusher against `https://matrix.org/_matrix/push/v1/notify`, Element's own shared, publicly-hosted gateway. That path is independent of anything in this stack and just works.

### Confirmed bugs — Forest Network app, iOS (found 2026-07-13)

Queried the live pusher registrations directly (`docker exec tfn-postgres psql -U synapse -d synapse -c "SELECT app_id, kind, data FROM pushers;"`) and cross-checked against `synapse/push/httppusher.py` and `sygnal/gcmpushkin.py` inside the running containers. Two separate bugs, both currently breaking delivery:

1. **`app_id` mismatch.** The Forest Network app always registers its pusher with `app_id: earth.theforestnetwork.ios.ios.prod` (doubled `ios`) — confirmed consistent across 4 days of retained logs and two different user accounts, so it's baked into the app, not a one-off. `sygnal.yaml` only defines `earth.theforestnetwork.ios.prod` (single `ios`) — no match, so sygnal has nothing to route these notifications to.
2. **Unreachable push URL.** The app registers `data.url` as `http://tfn-sygnal:5000/_matrix/push/v1/notify` — Synapse's internal Docker hostname on the `proxy` network (subnet `172.18.0.0/16`), over plain HTTP. Synapse's default SSRF protection (`ip_range_blocklist`, defaults to `DEFAULT_IP_RANGE_BLOCKLIST` which includes `172.16.0.0/12`) blocks the homeserver from ever making this request. This means the Forest Network app's own push path has likely never delivered a single notification through `tfn-sygnal`, independent of bug #1.

A push "received a month ago" (~2026-06-12/13, when sygnal was first stood up) is best explained by Element X's independent matrix.org gateway, not the Forest Network app / tfn-sygnal path.

### Fix required (iOS, app-side)

The Forest Network app needs to register its pusher with:
- `app_id` matching a key actually defined in `sygnal.yaml`
- `data.url` = the **public** HTTPS sygnal endpoint (e.g. `https://sygnal.theforestnetwork.earth/_matrix/push/v1/notify`) — not the internal Docker hostname, because of the SSRF blocklist above.

Either rename the `sygnal.yaml` key to match what the app already sends, or fix the app to send what `sygnal.yaml` expects — just make sure the two agree.

### Cloudflare Tunnel route needed

`config/cloudflared/` has no local config — tunnel routes are managed in the Cloudflare Zero Trust dashboard, not this repo. Confirm/add a public hostname route:
- `sygnal.theforestnetwork.earth` → `http://tfn-sygnal:5000`

This is required for both the iOS fix above and the Android setup below.

### Android (FCM) setup checklist

1. **Firebase** — create/open the Firebase project, add an Android app with the exact `applicationId` the app is built with. Project Settings → Service Accounts → **Generate new private key**. Confirmed the installed `tfn-sygnal` image only supports the FCM **v1** API (`project_id` + `service_account_file` in `gcmpushkin.py`) — the legacy server-key API isn't an option.
2. **Sygnal config** (`/media-server/config/tfn/sygnal/`):
   - Save the service account JSON, e.g. `firebase-service-account.json`.
   - Fill in the existing placeholder block in `sygnal.yaml`:
     ```yaml
     earth.theforestnetwork.android:
       type: gcm
       project_id: <firebase-project-id>
       service_account_file: /sygnal/firebase-service-account.json
     ```
   - Add the volume mount to `tfn-sygnal` in `docker-compose.yml`, same pattern as the existing `.p8` mounts:
     ```yaml
     - /media-server/config/tfn/sygnal/firebase-service-account.json:/sygnal/firebase-service-account.json:ro
     ```
   - Redeploy `tfn-sygnal`.
3. **Cloudflare Tunnel** — confirm/add the `sygnal.theforestnetwork.earth` route (see above).
4. **Android app** — bundle `google-services.json` from the same Firebase project, integrate the FCM SDK to get a registration token, and on login call `POST /_matrix/client/v3/pushers/set` with `kind: http`, `app_id` matching the `sygnal.yaml` key exactly, `pushkey` = the FCM token, `data.url` = the public sygnal URL above.

### Key learning

Whichever `app_id`/URL convention gets used, the app-side pusher registration and `sygnal.yaml` must agree exactly, and the URL must be publicly reachable over HTTPS — not an internal Docker hostname. This exact mismatch is what silently broke iOS push.
