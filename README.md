# partner
PETUNJUK INSTALASI

unzip openclaw-main1.zip
rm -r openclaw-main1.zip
cd openclaw-main

nano scripts/docker/setup.sh
OPENCLAW_EXTRA_MOUNTS=/var/run/docker.sock:/var/run/docker.sock
export OPENCLAW_ALLOW_INSECURE_PRIVATE_WS="${OPENCLAW_ALLOW_INSECURE_PRIVATE_WS:-}
OPENCLAW_TZ=Asia/Jakarta
HOME_VOLUME_NAME="${OPENCLAW_HOME_VOLUME:-$(pwd)}"
SANDBOX_ENABLE=1

nano Dockerfile
ARG OPENCLAW_VARIANT=slim
OPENCLAW_DOCKER_APT_PACKAGES="nano curl git"
OPENCLAW_INSTALL_DOCKER_CLI=1


chmod +x docker-setup.sh
./docker-setup.sh


nano docker-compose.main.yml
-------------------------------
services:
  openclaw-gateway:
    image: ${OPENCLAW_IMAGE:-openclaw:local}
    environment:
      HOME: /home/node
      TERM: xterm-256color
      OPENCLAW_GATEWAY_TOKEN: ${OPENCLAW_GATEWAY_TOKEN:-}
      OPENCLAW_ALLOW_INSECURE_PRIVATE_WS: ${OPENCLAW_ALLOW_INSECURE_PRIVATE_WS:-}
      CLAUDE_AI_SESSION_KEY: ${CLAUDE_AI_SESSION_KEY:-}
      CLAUDE_WEB_SESSION_KEY: ${CLAUDE_WEB_SESSION_KEY:-}
      CLAUDE_WEB_COOKIE: ${CLAUDE_WEB_COOKIE:-}
      TZ: ${OPENCLAW_TZ:-}
    volumes:
      - ${OPENCLAW_CONFIG_DIR}:/home/node/.openclaw
      - ${OPENCLAW_WORKSPACE_DIR}:/home/node/.openclaw/workspace
      ## Uncomment the lines below to enable sandbox isolation
      ## (agents.defaults.sandbox). Requires Docker CLI in the image
      ## (build with --build-arg OPENCLAW_INSTALL_DOCKER_CLI=1) or use
      ## scripts/docker/setup.sh with OPENCLAW_SANDBOX=1 for automated setup.
      ## Set DOCKER_GID to the host's docker group GID (run: stat -c '%g' /var/run/docker.sock).
      ##- /home/warungbudina/asistenku/openclaw-main:/home/node/openclaw-main:rw
      - /var/run/docker.sock:/var/run/docker.sock
    group_add:
      - "${DOCKER_GID:-}"
    ports:
      - "${OPENCLAW_GATEWAY_PORT:-18789}:18789"
      - "${OPENCLAW_BRIDGE_PORT:-18790}:18790"
    init: true
    restart: unless-stopped
    command:
      [
        "node",
        "dist/index.js",
        "gateway",
        "--bind",
        "${OPENCLAW_GATEWAY_BIND:-lan}",
        "--port",
        "18789",
      ]
    healthcheck:
      test:
        [
          "CMD",
          "node",
          "-e",
          "fetch('http://127.0.0.1:18789/healthz').then((r)=>process.exit(r.ok?0:1)).catch(()=>process.exit(1))",
        ]
      interval: 30s
      timeout: 5s
      retries: 5
      start_period: 20s

  cloudflared:
    image: cloudflare/cloudflared:latest
    container_name: cloudflared
    restart: always
    command: >
      tunnel --no-autoupdate run --token ${CLOUDFLARED_TOKEN:-}
-------------------------------------------------------------------------------------------------------------

docker compose down
docker compose \
-f docker-compose.main.yml \
up -d

sudo docker compose exec openclaw-gateway sh -lc 'id && ls -l /var/run/docker.sock && docker ps'

cp /home/balibruntattour/openclaw-main/scripts/sandbox-browser-setup.sh /home/balibruntattour/openclaw-main
chmod +x sandbox-browser-setup.sh
./sandbox-browser-setup.sh



docker exec -it  sh

openclaw config set agents.defaults.sandbox.mode off
openclaw config validate

openclaw gateway status
openclaw tui --deliver
openclaw devices list
openclaw devices approve 
