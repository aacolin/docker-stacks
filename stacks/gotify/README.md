# Gotify behid Traefik reverse proxy on a remote host

This Docker Compose file deploys the Gotify server behind a Traefik reverse proxy. It configures an external data volume and attaches the service to the `reverse-proxy` network so Traefik can route and secure traffic.

⚠️ **Important — Traefik setup**

- This stack is designed to work with either a local **`traefik`** container or a **`traefik-kop`** bridge container.

---

### When Traefik runs on the *same host*

- The `traefik` container must run on the **same Docker host** as your services.
- All services must share the same Docker network (commonly `reverse-proxy`).
- In this setup, Traefik can directly access the local Docker socket to auto-discover services.

---

### When Traefik runs on a *different host*

- A Traefik instance **cannot natively manage remote Docker sockets.**
- To work around this, you can use **`traefik-kop`** on the remote host.

`traefik-kop` acts as a bridge by:

- Forwarding container metadata
- Forwarding HTTP traffic
- Sending metadata to a Redis database that the main Traefik instance reads

This allows the central Traefik server to discover and route to services running on other hosts.

---

### Networking requirements

- Both the Gotify container and `traefik-kop` must be attached to the same Docker network (commonly `reverse-proxy`).
- You may rename the network, but it must match across services.
- A typical `traefik-kop` compose includes:
  - `container_name: traefik-kop`
  - `networks: [reverse-proxy]`

---

### Routing & TLS responsibility

- The **central Traefik instance** (on the main host) performs:
  - Routing
  - TLS termination
  - Certificate management

- `traefik-kop` only forwards metadata and traffic.

---

### Before deployment

Ensure the proxy network exists:


Create it with:

`
docker network create reverse-proxy
`

Traefik labels explanation
- `traefik.enable=true`: tells Traefik to consider this container for routing.
- `traefik.http.routers.gotify.*`: defines the router name, entrypoint (`https`), host rule (from `${APP_URL}`), and TLS settings.
- `traefik.http.services.gotify.loadbalancer.server.*`: informs Traefik of the internal scheme and port to reach the Gotify service.

Deployment options and security notes
- You can deploy this compose directly with `docker compose up -d`.
- Alternatively use Dockhand to manage and deploy the stack. Dockhand can store and encrypt environment variables in its database and inject them into containers at runtime, which avoids leaving plaintext secrets on the host filesystem. Using Dockhand is generally safer for secrets management.

Backup recommendation
- Back up your environment variables and any secrets using an encrypted password manager such as Bitwarden or your preferred encrypted backup strategy. This provides an additional recovery layer in case the server is compromised or you lose access to the `.env` file.

## `.env` file (template)

The included `.env` file is a **template** that shows which environment variables the stack expects.  
It does **not** contain real values by default — all entries mudt be filled out.

---

## Variable explanations

`GOTIFY_DEFAULTUSER_NAME`
Default admin username created on first launch.

`GOTIFY_DEFAULTUSER_PASS`
Default admin password. Use a strong, unique password.

`APP_PORT`
Internal port Gotify listens on inside Docker.
Example: 8090 if 8080 is already in use.

`CERT_RESOLVER`
The certificate resolver configured in your Traefik instance.
Examples: letsencrypt, cloudflare.

`APP_URL`
Public URL for accessing Gotify.
Example:

https://gotify.yourdomain.com


`TZ`
Container timezone.
Examples:

America/Los_Angeles
Europe/Berlin
Asia/Kolkata


## example.env template contents

```bash
#GOTIFY_DEFAULTUSER_NAME=admin_username
#GOTIFY_DEFAULTUSER_PASS=super_secure_password
#APP_PORT=your_app_port
#CERT_RESOLVER=your_cert_resolver
#APP_URL=https://gotify.yourdomain.com
#TZ=your_local_timezone
```