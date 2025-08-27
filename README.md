# PocketBase Docker with Nginx

Production-ready PocketBase setup with Docker Compose and Nginx reverse proxy.

## Quick Start

1. Clone the repository:
```bash
git clone https://github.com/nazt/pocketbase-docker-nginx.git
cd pocketbase-docker-nginx
```

2. Copy the environment file:
```bash
cp .env.example .env
```

3. Start the services:
```bash
docker-compose up -d
```

4. Access PocketBase:
- Direct access: http://localhost:8091
- Via Nginx proxy: http://localhost
- Admin UI: http://localhost/_/

## Services

### PocketBase
- Port: 8091 (direct access)
- Data: `./data/pb_data` (local volume)
- Version: 0.29.3 (configurable in .env)

### Nginx
- Port: 80 (HTTP)
- Proxy: Routes to PocketBase on port 8080
- Config: `./nginx/nginx.conf`

## Architecture

```
[Client] → [Nginx:80] → [PocketBase:8080] → [Local Volume:./data/pb_data]
```

## Directory Structure

```
.
├── Dockerfile              # PocketBase custom image
├── docker-compose.yml      # Service orchestration
├── nginx/
│   └── nginx.conf         # Nginx reverse proxy config
├── data/
│   └── pb_data/          # PocketBase data (gitignored)
├── .env.example           # Environment template
└── README.md              # This file
```

## Environment Variables

| Variable | Default | Description |
|----------|---------|-------------|
| PB_VERSION | 0.29.3 | PocketBase version |
| NGINX_PORT | 80 | Nginx HTTP port |
| SERVER_NAME | localhost | Server name for Nginx |

## Admin Setup

First-time setup requires creating an admin account:
1. Navigate to http://localhost/_/
2. Create your admin account
3. Configure PocketBase settings

## Backup & Restore

### Backup
```bash
docker-compose exec pocketbase /pb/pocketbase backup
```

### Restore
Place backup in `./data/pb_data` and restart:
```bash
docker-compose down
docker-compose up -d
```

## Development

### View logs
```bash
docker-compose logs -f pocketbase
docker-compose logs -f nginx
```

### Restart services
```bash
docker-compose restart
```

### Stop services
```bash
docker-compose down
```

### Remove all data (WARNING!)
```bash
docker-compose down -v
rm -rf ./data/pb_data/*
```

## Security Notes

- PocketBase admin panel is at `/_/` path
- Configure "User IP proxy headers" in PocketBase settings (X-Real-IP, X-Forwarded-For)
- For production, use SSL/TLS certificates with Nginx
- Change default admin credentials immediately

## License

MIT