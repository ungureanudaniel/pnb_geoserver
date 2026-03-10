# Park Map — GIS Web Stack

Interactive web map powered by **GeoServer + PostGIS + Leaflet.js**, served via **Nginx**, all running in **Docker**.

---

## Stack

```
map/index.html          ← Leaflet frontend
nginx/                  ← Nginx container (reverse proxy + static files)
geoserver/data/         ← GeoServer data dir (gitignored, lives on server)
docker-compose.yml      ← orchestrates both containers
```

---

## First-Time Server Setup

### 1. Clone the repo
```bash
git clone https://github.com/yourusername/park-map.git /opt/park-map
cd /opt/park-map
```

### 2. Create your .env file
```bash
cp .env.example .env
nano .env   # set GEOSERVER_ADMIN_PASSWORD and DOMAIN
```

### 3. Update your domain in nginx.conf
```bash
nano nginx/nginx.conf
# Replace both occurrences of 'your-domain.com'
```

### 4. Add SSL certificates
The cert for `map.bucegipark.ro` is managed in cPanel. Download it and place it in `nginx/ssl/`:

```
nginx/ssl/
├── fullchain.pem    ← Certificate (CRT) from cPanel SSL/TLS Manager
└── privkey.pem      ← Private Key from cPanel SSL/TLS Manager
```

This folder is gitignored — you need to do this step manually on every fresh clone (both locally and on the server).

> **Cert renewal:** when the cert expires, download the new one from cPanel and replace the files, then restart Nginx:
> ```bash
> docker compose restart nginx
> ```

### 5. Start all containers
```bash
docker compose up -d
```

### 6. Verify everything is running
```bash
docker compose ps
docker compose logs -f
```

GeoServer admin: https://map.bucegipark.ro/geoserver/web  
Map: https://map.bucegipark.ro

---

## Daily Dev Workflow

```bash
# 1. Edit map/index.html locally
# 2. Test by opening map/index.html in your browser
# 3. Commit and push
git add .
git commit -m "your message"
git push

# 4. Deploy to server
ssh user@your-server "cd /opt/park-map && git pull && docker compose restart nginx"

| Task | Command |
|---|---|
| Start stack | `docker compose up -d` |
| Stop stack | `docker compose down` |
| Restart Nginx only | `docker compose restart nginx` |
| Restart GeoServer | `docker compose restart geoserver` |
| View all logs | `docker compose logs -f` |
| GeoServer logs only | `docker compose logs -f geoserver` |
| Nginx logs only | `docker compose logs -f nginx` |
| Rebuild Nginx image | `docker compose build nginx` |

---

## GeoServer Connection (PostGIS Store)

When adding a PostGIS store in GeoServer, use these connection settings:

| Field | Value |
|---|---|
| Host | `host.docker.internal` |
| Port | `5432` |
| Database | your db name |
| Schema | `public` |
| User | your db user |
| Password | your db password |

> `host.docker.internal` resolves to the Ubuntu host machine from inside the container.

---

## Notes

- `.env` is gitignored — never commit passwords
- `geoserver/data/` is gitignored — managed directly on the server
- `nginx/ssl/` is gitignored — certs are managed on the server via certbot
- After `git pull` on the server, only restart nginx (`docker compose restart nginx`) unless you changed `docker-compose.yml` or GeoServer config (then `docker compose up -d` instead)
