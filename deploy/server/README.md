# DropZone server

This folder turns your computer into the server where the CD workflow deploys DropZone.
Only a GitHub account is needed: the tunnel works without a Cloudflare account.

| Service | Role |
|---|---|
| `runner` | self-hosted GitHub Actions runner, label `dropzone-server`; it uses the Docker of this computer |
| `tunnel` | Cloudflare quick tunnel: public HTTPS address for the application published on port 8080 |

## Start

1. On GitHub: *Settings → Actions → Runners → New self-hosted runner*. Copy the token shown in the *Configure* step.
2. On GitHub: *Settings → Actions → General → Approval for running fork pull request workflows*: require approval for all external contributors (a runner on your computer must never run a stranger's code).
3. In a terminal, from the root of the repository:

   ```bash
   cp deploy/server/.env.example deploy/server/.env   # fill in REPO_URL and RUNNER_TOKEN
   docker compose -f deploy/server/compose.yml --env-file deploy/server/.env up -d
   ```

4. The runner appears as *Idle* in *Settings → Actions → Runners*.
5. Public address of the application (once deployed):

   ```bash
   docker logs dropzone-tunnel 2>&1 | grep trycloudflare.com
   ```

## Stop

```bash
docker compose -f deploy/server/compose.yml --env-file deploy/server/.env down
```

The runner stays registered (volume `runner-data`): `up -d` starts it again without a new token.
After `down -v`, remove it in *Settings → Actions → Runners* and register it again.

## Limits

- The application is online only while this computer is on.
- The tunnel address changes at each restart of the `tunnel` service.
- Quick tunnels have no uptime guarantee: fine for a course, not for production.
