# Deploying Cap on Dokploy

Uses prebuilt images (`ghcr.io/capsoftware/cap-web`, `cap-media-server`), so nothing is compiled on the server.

## 1. DNS

Point two A records at your Dokploy server:

- `cap.example.com` → the web app
- `s3.example.com` → MinIO (browsers upload and play videos directly from here, so it must be public)

## 2. Create the service

1. Dokploy → Project → **Create Service → Compose** (type: Docker Compose).
2. **Provider:** GitHub / Git → `https://github.com/azeezogundeko/Cap.git`, branch `main`.
3. **Compose Path:** `./docker-compose.dokploy.yml`

## 3. Environment

Open the **Environment** tab and paste `docker-compose.dokploy.env.example`, filling in:

- `CAP_URL`, `S3_PUBLIC_URL` — the two domains above, with `https://`
- `DATABASE_ENCRYPTION_KEY`, `NEXTAUTH_SECRET`, `MEDIA_SERVER_WEBHOOK_SECRET` — `openssl rand -hex 32` each
- `MYSQL_PASSWORD`, `MYSQL_ROOT_PASSWORD`, `MINIO_ROOT_PASSWORD` — strong random passwords

Don't change `DATABASE_ENCRYPTION_KEY` after first deploy, or stored secrets become unreadable.

## 4. Domains

In the **Domains** tab add:

| Service   | Host              | Container port | HTTPS |
|-----------|-------------------|----------------|-------|
| `cap-web` | `cap.example.com` | 3000           | on (Let's Encrypt) |
| `minio`   | `s3.example.com`  | 9000           | on (Let's Encrypt) |

Optional: `minio` port 9001 on another host for the MinIO console.

## 5. Deploy

Click **Deploy**. First start takes ~1–2 min (MySQL init + migrations). Then open `https://cap.example.com`.

- No email provider? The login link appears in the `cap-web` logs.
- Desktop app: Settings → Cap Server URL → `https://cap.example.com`.

## Troubleshooting

- **Videos won't upload/play:** `S3_PUBLIC_URL` must exactly match the MinIO domain and be reachable over HTTPS.
- **Redirects to localhost:** `CAP_URL` is wrong or missing — redeploy after fixing.
- **Large uploads fail:** raise any upload limit in front of Traefik (e.g. Cloudflare proxy caps at 100 MB — use DNS-only for `s3.`).
