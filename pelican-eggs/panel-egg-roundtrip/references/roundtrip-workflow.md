# Pelican and Pterodactyl Round-Trip Workflow

## Contents

1. Variables and resource ledger
2. Prepare Docker and Colima
3. Prepare official Compose files
4. Start and initialize Pelican
5. Pelican browser flow
6. Start and initialize Pterodactyl
7. Pterodactyl browser flow
8. Install exported artifacts
9. Validation
10. Cleanup audit
11. Troubleshooting

## 1. Variables and resource ledger

Choose unique values and keep them consistent:

```bash
RUN_ID="$(date +%s)-$$"
TMP_ROOT="${TMPDIR:-/tmp}/panel-egg-roundtrip-$RUN_ID"
COLIMA_PROFILE="panel-egg-$RUN_ID"
DOCKER_CONTEXT="colima-$COLIMA_PROFILE"
PELICAN_PROJECT="pelican-export-$RUN_ID"
PTERO_PROJECT="pterodactyl-export-$RUN_ID"
PELICAN_PORT=8081
PTERO_PORT=8082
mkdir -p "$TMP_ROOT"/{pelican,pterodactyl,downloads}
```

Confirm both ports are unused; choose different high ports if necessary. Record the target repository, egg paths, original Docker context, temporary paths, profile, projects, ports, and browser session names. Do not depend on memory for cleanup.

Generate credentials without displaying them:

```bash
openssl rand -base64 36 | tr -d '\n/+=' > "$TMP_ROOT/pelican-admin-password"
openssl rand -base64 36 | tr -d '\n/+=' > "$TMP_ROOT/pterodactyl-admin-password"
openssl rand -hex 24 > "$TMP_ROOT/pelican-db-password"
openssl rand -hex 24 > "$TMP_ROOT/pelican-db-root-password"
openssl rand -hex 24 > "$TMP_ROOT/pterodactyl-db-password"
openssl rand -hex 24 > "$TMP_ROOT/pterodactyl-db-root-password"
chmod 600 "$TMP_ROOT"/*password
```

## 2. Prepare Docker and Colima

Homebrew can install the Docker CLI without exposing it on the current `PATH`. Normalize it before starting Colima:

```bash
export PATH="$(brew --prefix docker)/bin:$PATH"
ORIGINAL_DOCKER_CONTEXT="$(docker context show 2>/dev/null || printf default)"
colima start "$COLIMA_PROFILE" --runtime docker --cpu 4 --memory 8 --disk 40
docker --context "$DOCKER_CONTEXT" version
export DOCKER_CONTEXT
```

Prefer `docker --context "$DOCKER_CONTEXT" compose` when the Compose plugin is discoverable. Homebrew installations may expose only the standalone `docker-compose`; in that case set `DOCKER_CONTEXT="$DOCKER_CONTEXT"` for every `docker-compose` invocation.

Colima may switch the global current Docker context. Restore the original/default context during cleanup.

## 3. Prepare official Compose files

Download fresh upstream files rather than bundling stale copies:

```bash
curl -fsSL \
  https://raw.githubusercontent.com/pelican-dev/panel/refs/heads/main/compose-full-stack.yml \
  -o "$TMP_ROOT/pelican/compose.yml"

curl -fsSL \
  https://raw.githubusercontent.com/pterodactyl/panel/refs/heads/1.0-develop/docker-compose.example.yml \
  -o "$TMP_ROOT/pterodactyl/compose.yml"
```

Inspect the downloaded files before adapting them because upstream structure can change.

### Pelican adaptations

- Set `APP_URL` to `http://localhost:$PELICAN_PORT`.
- Map host and container to the same port: `"$PELICAN_PORT:$PELICAN_PORT"`. Integrated Caddy listens on the port embedded in `APP_URL`, not necessarily port 80.
- Disable/comment the HTTPS mapping for local temporary use.
- Substitute random MariaDB passwords.
- Assign a unique subnet.
- Keep named volumes.
- If the Compose file mounts a `plugins` volume subpath, create that directory before starting the panel:

```bash
docker --context "$DOCKER_CONTEXT" run --rm \
  -v "${PELICAN_PROJECT}_pelican-data:/data" alpine:latest \
  mkdir -p /data/plugins
```

### Pterodactyl adaptations

- Set `APP_URL` to `http://localhost:$PTERO_PORT`.
- Map `"$PTERO_PORT:80"` and disable/comment the HTTPS mapping.
- Substitute random MariaDB passwords.
- Add `RECAPTCHA_ENABLED: "false"` to the panel environment for local automation.
- Replace every `/srv/pterodactyl/...` bind mount with a project-scoped named volume and declare those volumes.
- Assign a subnet different from Pelican's.

Validate both configurations before starting:

```bash
docker-compose -p "$PELICAN_PROJECT" -f "$TMP_ROOT/pelican/compose.yml" config --quiet
docker-compose -p "$PTERO_PROJECT" -f "$TMP_ROOT/pterodactyl/compose.yml" config --quiet
```

## 4. Start and initialize Pelican

Use the official images specified by the Compose file:

```bash
docker-compose -p "$PELICAN_PROJECT" -f "$TMP_ROOT/pelican/compose.yml" pull
docker-compose -p "$PELICAN_PROJECT" -f "$TMP_ROOT/pelican/compose.yml" up -d --no-build
```

Wait for the panel container to become healthy and for `curl http://localhost:$PELICAN_PORT/` to return an HTTP response.

The full-stack image may start before database migrations are installed. Run migrations explicitly, then create an administrator:

```bash
docker-compose -p "$PELICAN_PROJECT" -f "$TMP_ROOT/pelican/compose.yml" \
  exec -T panel php artisan migrate --seed --force

PASS="$(cat "$TMP_ROOT/pelican-admin-password")"
docker-compose -p "$PELICAN_PROJECT" -f "$TMP_ROOT/pelican/compose.yml" \
  exec -T panel php artisan p:user:make \
  --email=export@example.invalid --username=exportadmin \
  --password="$PASS" --admin=1 --no-interaction
unset PASS
```

## 5. Pelican browser flow

Use an isolated `agent-browser` session and a temporary download directory:

```bash
agent-browser --session pelican-export \
  --download-path "$TMP_ROOT/downloads" \
  open "http://localhost:$PELICAN_PORT"
```

Follow the snapshot/ref loop:

1. Snapshot the login form.
2. Fill the administrator email and password read from the credential file; do not print the password.
3. Open `/admin`.
4. Navigate to **Eggs** and choose **Import**.
5. Select the **File** tab.
6. Upload the working PLCN_v3 YAML using `input[type=file]`.
7. Wait until the filename appears and upload status completes.
8. Submit and wait for the imported egg row.
9. Choose **Export**, then **As .yaml**.
10. Validate the newly downloaded file before renaming it to the expected repository filename.

If `agent-browser download` times out but a new UUID-named file appears, the download succeeded. Identify only the file created by the current action; do not pick an older artifact.

After securing the export outside Docker volumes, close the browser session and remove the Pelican project:

```bash
agent-browser --session pelican-export close
docker-compose -p "$PELICAN_PROJECT" -f "$TMP_ROOT/pelican/compose.yml" \
  down -v --remove-orphans
```

## 6. Start and initialize Pterodactyl

```bash
docker-compose -p "$PTERO_PROJECT" -f "$TMP_ROOT/pterodactyl/compose.yml" pull
docker-compose -p "$PTERO_PROJECT" -f "$TMP_ROOT/pterodactyl/compose.yml" up -d
```

Wait for `http://localhost:$PTERO_PORT/auth/login` to return HTTP 200. Create an administrator:

```bash
PASS="$(cat "$TMP_ROOT/pterodactyl-admin-password")"
docker-compose -p "$PTERO_PROJECT" -f "$TMP_ROOT/pterodactyl/compose.yml" \
  exec -T panel php artisan p:user:make \
  --email=export@example.invalid --username=exportadmin \
  --name-first=Export --name-last=Admin \
  --password="$PASS" --admin=1 --no-interaction
unset PASS
```

If CAPTCHA remains visible, confirm the container received `RECAPTCHA_ENABLED=false`, recreate only the panel service, and run `php artisan config:clear`.

## 7. Pterodactyl browser flow

1. Open `/auth/login` in a separate `agent-browser` session.
2. Log in with the temporary administrator.
3. Open `/admin`.
4. Navigate to **Nests → Import Egg**.
5. Upload the working PTDL_v2 JSON using `input[type=file]`.
6. Inspect `select option` values with `agent-browser eval`; select an appropriate temporary nest through the real select element (commonly `#pImportToNest`).
7. Import and wait for the egg configuration page.
8. Confirm key fields such as startup command and denylist survived import.
9. Download from **Configuration → Export**.
10. Validate the download as PTDL_v2 JSON.

Then close the browser and remove the Pterodactyl project with `down -v --remove-orphans`.

## 8. Install exported artifacts

Copy the validated downloads over the intended repository paths. Do not reformat or manually patch them afterward. Panel serialization can legitimately change:

- `exported_at`
- object/key order
- YAML placement of `sort`
- quoting and escaping
- trailing newlines

These are acceptable only when semantics are preserved.

## 9. Validation

Run the smallest checks that prove the requested change and round-trip integrity:

```bash
jq empty path/to/pterodactyl-egg.json
ruby -e 'require "yaml"; YAML.load_file(ARGV[0])' path/to/pelican-egg.yaml
git diff --check
```

Extract `.scripts.installation.script` from both formats and run `bash -n`. Compare scripts after normalizing only the final newline. Validate requested fields directly (`file_denylist`, startup, images, variables, URLs, etc.). Review the full Git diff before commit.

## 10. Cleanup audit

Close all task browser sessions, restore the Docker context, delete the isolated Colima profile, and remove the workspace:

```bash
agent-browser --session pelican-export close 2>/dev/null || true
agent-browser --session pterodactyl-export close 2>/dev/null || true
unset DOCKER_CONTEXT
docker context use "$ORIGINAL_DOCKER_CONTEXT" >/dev/null 2>&1 || true
colima delete "$COLIMA_PROFILE" --force
rm -rf "$TMP_ROOT"
```

Colima can leave a profile data disk after deletion. Remove only the exact task profile disk after confirming the name:

```bash
rm -rf "$HOME/.colima/_lima/_disks/colima-$COLIMA_PROFILE"
```

Verify absence of:

- Both Compose projects' containers, networks, and volumes.
- The `colima-$COLIMA_PROFILE` Docker context.
- `$HOME/.colima/$COLIMA_PROFILE`.
- `$HOME/.colima/_lima/colima-$COLIMA_PROFILE` or the exact profile VM directory reported by Colima.
- `$HOME/.colima/_lima/_disks/colima-$COLIMA_PROFILE`.
- Temporary Compose files, credentials, downloads, extracted scripts, screenshots, and browser sessions.

Do not claim cleanup from `down -v` alone; deleting and auditing the isolated Colima profile is the stop condition.

## 11. Troubleshooting

### Pelican returns an empty HTTP response

Read Caddy logs. If Caddy listens on `:$PELICAN_PORT` while Compose maps `$PELICAN_PORT:80`, change the mapping to `$PELICAN_PORT:$PELICAN_PORT` and recreate the panel service.

### Pelican panel fails on `plugins` subpath

Pre-create `plugins` in the named `pelican-data` volume with a disposable Alpine container, then start the panel again.

### Administrator creation says the users table is missing

Run `php artisan migrate --seed --force` before `p:user:make`.

### Pterodactyl login displays reCAPTCHA

Set `RECAPTCHA_ENABLED=false`, recreate the panel service, and clear Laravel configuration cache.

### `docker compose` is unavailable although Docker Compose is installed

Use Homebrew's standalone `docker-compose` binary with `DOCKER_CONTEXT` set to the task context.

### Browser refs stop working

Re-snapshot after every navigation, modal open/close, upload state change, or Livewire render. Never reuse stale refs.
