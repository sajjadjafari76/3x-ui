# Deploying on Runflare (Kubernetes)

The image works unchanged on Runflare; the entrypoint adapts automatically:

- `PORT` (set by the platform) is used as the panel port when `XUI_PORT` is not set.
- Fail2ban is auto-disabled when the container lacks `NET_ADMIN` (always the case on Runflare).

## Steps
1. Build and push the image (Runflare pulls from a registry; the build needs GitHub access for Xray/geo files):
   `docker build -t <registry>/<user>/3x-ui:latest . && docker push <registry>/<user>/3x-ui:latest`
2. In the service's **Images** page, select that image.
3. Environment variables:
   - `XUI_PORT=8000`, and on the service's *Application* page choose **Expose Port 8000** (Runflare only offers 80, 3000, 5000, 8000).
   - `XUI_ENABLE_FAIL2BAN=false` (optional, auto-detected)
   - Optional: `XUI_DB_TYPE=postgres` + `XUI_DB_DSN=...` to keep data in an external DB.
4. **Persistent disk**: mount a volume at `/etc/x-ui` (database). Without it, settings/users are lost on every restart.
   Optionally also `/root/cert` and `/root/.acme.sh`.
5. Expose the panel through Runflare's HTTP domain. Xray inbounds on other ports are only reachable if the platform
   exposes extra TCP/UDP ports; HTTP-only routing supports WebSocket/gRPC/HTTPUpgrade transports behind the panel domain
   (use `security: none` on the inbound; TLS is terminated by the platform).
