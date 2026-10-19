---
name: server
description: >
  Start a Project portal's Vite dev server on a random free port and report the mounted URL, then screenshot it to
  confirm it actually rendered. Trigger when asked to start the server, run or preview the portal, show the local
  server, or restart it after a change; also covers stopping one and finding what is already running. Kept identical,
  byte for byte, in every Project repository's `.agents/skills/`; this canonical copy lives in Navigator's own
  `.agents/skills/`.
---

# Start the portal on a random port

Before naming a table, field, state, or UI concept, follow the **Find the term first** section in `AGENTS.md`.

A Project repository has `navigator.yaml` at its root declaring `host` and `project`, and a `portal/` beside it holding
a React and Vite app. Start that app and hand back a URL a browser can open.

## The port is random, never the default

**Do not start Vite on 5173, and do not let it count up to the next free port.** A predictable port breaks two ways:

- Every Project repository shares the same Vite config. Two portals started the same afternoon land on 5173 and 5174,
  and the second tab can show the *first* Project's matter: a privileged surface for the wrong client, in a window that
  looks correct.
- A stale server from an earlier session holds the default, so `pnpm dev` attaches you to yesterday's bundle and every
  edit appears to do nothing.

Pick a random high port, check it is free, and pass `--strictPort` so Vite fails loudly instead of drifting:

| Command on a port already in use | What Vite does |
| --- | --- |
| `pnpm dev --port N --strictPort` | `Error: Port N is already in use` — refuses to start |
| `pnpm dev --port N` | `Port N is in use, trying another one...` — **silently binds N+1** |

## Run it

```bash
root=$PWD; while [ ! -f "$root/navigator.yaml" ] && [ "$root" != / ]; do root=$(dirname "$root"); done
[ -f "$root/navigator.yaml" ] || { echo "not inside a Project repository"; exit 1; }
code=$(sed -n 's/^project:[[:space:]]*["'"'"']\{0,1\}\([a-z0-9-]*\).*/\1/p' "$root/navigator.yaml")

port=""
for _ in $(seq 1 25); do
  candidate=$(( 20000 + RANDOM % 40000 ))
  if ! lsof -iTCP:"$candidate" -sTCP:LISTEN -n -P >/dev/null 2>&1; then port=$candidate; break; fi
done
[ -n "$port" ] || { echo "no free port in 25 tries"; exit 1; }

cd "$root/portal" && pnpm dev --port "$port" --strictPort
```

Launch it in the background, then poll the task's output until the `Local:` line appears (the output is empty until
then). Report the URL Vite prints, not one you composed.

On `EADDRINUSE` (a race between the check and the bind), pick a new random port and rerun. Never let Vite choose.

## The URL is the mount, not the bare host

The bundle is built at a base of `/app/projects/<code>/portal/`. `http://localhost:<port>/` is a 404 that looks like a
broken server. Always hand over the full mounted path, trailing slash included, since Vite joins asset URLs onto it.

## Look at it before saying it works

`curl` returns the single-page-application shell for any path under the mount, so a 200 proves routing, not rendering.
Screenshot with the installed Chrome (no driver needed):

```bash
"/Applications/Google Chrome.app/Contents/MacOS/Google Chrome" \
  --headless --disable-gpu --window-size=1400,1600 --virtual-time-budget=4000 \
  --screenshot=portal.png "http://localhost:$port/app/projects/$code/portal/"
```

Then read the PNG. A blank frame is a failure to launch, not a slow paint; `--virtual-time-budget` already waited. macOS
has no `timeout` by default, so background the process and poll instead.

## Stopping, and what is already up

```bash
lsof -iTCP -sTCP:LISTEN -n -P | grep -i node
```

Stop a backgrounded server through the harness's task-stop, not `kill`, so the task list matches reality.
