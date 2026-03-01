Manage the local OpenClaw Docker gateway (container name: `openclaw-docker`, image: `openclaw:local`, model: `moonshotai/kimi-k2.5` via OpenRouter).

Interpret `$ARGUMENTS` as one of:

- **start**: `docker start openclaw-docker`. If the container doesn't exist, read `OPENROUTER_API_KEY` from the repo `.env` file and create it:
  ```
  OPENROUTER_KEY=$(grep '^OPENROUTER_API_KEY=' .env | cut -d= -f2-)
  docker run -d --name openclaw-docker \
    -e HOME=/home/node -e TERM=xterm-256color \
    -e OPENROUTER_API_KEY="$OPENROUTER_KEY" \
    -v "$HOME/.openclaw/docker:/home/node/.openclaw" \
    -v "$HOME/.openclaw/docker-workspace:/home/node/.openclaw/workspace" \
    -p 18789:18789 -p 18790:18790 \
    --init --restart unless-stopped \
    openclaw:local \
    node openclaw.mjs gateway --allow-unconfigured --bind lan --port 18789
  ```
  If `OPENROUTER_API_KEY` is not found in `.env`, stop and tell the user to add it.
- **stop**: `docker stop openclaw-docker`
- **restart**: `docker restart openclaw-docker`
- **status**: Show `docker ps --filter name=openclaw-docker` and `docker logs --tail 20 openclaw-docker`
- **logs** (or **logs -f**): `docker logs --tail 50 openclaw-docker` (add `-f` if requested)
- **exec <cmd>**: `docker exec openclaw-docker node openclaw.mjs <cmd>` (e.g. `exec status`, `exec config get models`)
- **shell**: `docker exec -it openclaw-docker bash`
- **rebuild**: Build the image from repo root (`docker build -t openclaw:local .`), then stop/remove/recreate the container (re-read key from `.env`)
- **send <message>**: `docker exec openclaw-docker node openclaw.mjs message send "<message>"`
- **url**: Print the dashboard URL: `http://localhost:18789/?token=change-me-to-a-long-random-token`

If no argument is given, show the container status.

Config lives at `~/.openclaw/docker/openclaw.json`. Workspace at `~/.openclaw/docker-workspace/`.

After running the command, briefly report the result.
