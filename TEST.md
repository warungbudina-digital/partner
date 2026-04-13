OPENCLAW

openclaw doctor --fix
openclaw config validate
openclaw config set agents.defaults.sandbox.docker.image openclaw-sandbox-common:bookworm-slim
openclaw config set agents.defaults.sandbox.mode all
openclaw config set agents.defaults.sandbox.workspaceAccess rw
openclaw config set agents.defaults.sandbox.docker.network bridge
openclaw config set agents.defaults.sandbox.docker.readOnlyRoot false
openclaw config set agents.defaults.sandbox.docker.privileged true
openclaw config set agents.defaults.sandbox.docker.capAdd '["ALL"]' --strict-json
openclaw config set agents.defaults.sandbox.docker.binds '["/var/run/docker.sock:/var/run/docker.sock:rw"]' --strict-json
openclaw config get agents.defaults.sandbox.docker
openclaw config get gateway.controlUi.allowedOrigins
openclaw config set gateway.trustedProxies '["172.19.0.0/16"]' --strict-json
openclaw config set gateway.auth.mode '"token"' --strict-json
openclaw config set gateway.auth.token '"xx"' --strict-json
openclaw config set gateway.controlUi.allowedOrigins '["http://localhost:18789","http://127.0.0.1:18789","https://claw.delitourandphotography.com"]' --strict-json
openclaw config set gateway.trustedProxies '["172.19.0.0/16"]' --strict-json
openclaw tui --deliver


sudo docker compose up -d --force-recreate

