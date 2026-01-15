# Docker Deployment Guide

This guide walks through deploying the TLDW YT Video application to your own server using Docker and Docker Compose.

## Prerequisites

- Docker and Docker Compose installed on your server
- Redis already running on your server
- OpenAI API key
- Supadata API key (optional, for transcript fetching)

## Setup

### 1. Environment Configuration

Create a `.env.local` file in the project root:

```bash
# OpenAI API Configuration
OPENAI_API_KEY=your_openai_api_key_here

# Supadata API Configuration (optional)
SUPADATA_API_KEY=your_supadata_api_key_here

# Redis Configuration
# If using the Redis from docker-compose:
REDIS_HOST=redis
REDIS_PORT=6379

# If using external Redis already running on your server:
REDIS_HOST=127.0.0.1
REDIS_PORT=6379
```

### 2. Choose Your Setup

#### Option A: Docker Compose with Bundled Redis (Simpler)

Use the provided `docker-compose.yml`:

```bash
# Start services
docker-compose up -d

# View logs
docker-compose logs -f app

# Stop services
docker-compose down
```

#### Option B: Docker Compose without Redis (Your Existing Setup)

If you want to use your existing Redis server instead:

1. Modify `docker-compose.yml` and remove the `redis` service entirely
2. Update the app `depends_on` to only depend on the network (or remove it)
3. Set environment variables to point to your existing Redis

```bash
# Start only the app
docker-compose up -d app

# View logs
docker-compose logs -f app
```

#### Option C: Just Docker (No Compose)

```bash
# Build the image
docker build -t tldr-yt-video .

# Run the container
docker run -d \
  -p 3000:3000 \
  -e OPENAI_API_KEY=your_key \
  -e SUPADATA_API_KEY=your_key \
  -e REDIS_HOST=127.0.0.1 \
  -e REDIS_PORT=6379 \
  --restart unless-stopped \
  --name tldr-app \
  tldr-yt-video
```

## Production Deployment

### Recommended: Use Nginx as Reverse Proxy

Create an Nginx config:

```nginx
server {
    listen 80;
    server_name your-domain.com;

    # Redirect to HTTPS
    return 301 https://$server_name$request_uri;
}

server {
    listen 443 ssl http2;
    server_name your-domain.com;

    ssl_certificate /path/to/cert.pem;
    ssl_certificate_key /path/to/key.pem;

    location / {
        proxy_pass http://localhost:3000;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection 'upgrade';
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
        proxy_cache_bypass $http_upgrade;
    }
}
```

### SSL Certificate with Certbot

```bash
sudo certbot certonly --standalone -d your-domain.com
```

### Systemd Service File (Optional)

Create `/etc/systemd/system/tldr-app.service`:

```ini
[Unit]
Description=TLDW YT Video Application
After=docker.service redis.service
Requires=docker.service

[Service]
Type=simple
User=root
WorkingDirectory=/path/to/tldr-yt-video
ExecStart=/usr/bin/docker-compose up
ExecStop=/usr/bin/docker-compose down
Restart=unless-stopped

[Install]
WantedBy=multi-user.target
```

Enable and start:

```bash
sudo systemctl daemon-reload
sudo systemctl enable tldr-app
sudo systemctl start tldr-app
```

## Monitoring & Logs

```bash
# View logs from docker-compose
docker-compose logs -f app

# View specific number of lines
docker-compose logs --tail=100 app

# View logs from specific service
docker-compose logs -f redis

# Check container status
docker-compose ps
```

## Maintenance

### Updating the Application

```bash
# Pull latest code
git pull origin main

# Rebuild the Docker image
docker-compose up -d --build

# This will automatically restart the app with new code
```

### Backing Up Redis Data

```bash
# The redis_data volume stores data at:
docker volume inspect tldr-network_redis_data

# Manual backup:
docker exec tldr-redis redis-cli --rdb /data/backup.rdb
docker cp tldr-redis:/data/backup.rdb ./redis_backup.rdb
```

### Scaling & Performance

- Adjust Node.js memory: Add `-e NODE_OPTIONS="--max-old-space-size=512"` to docker-compose
- Use a load balancer (Nginx) if scaling to multiple instances
- Monitor container resource usage: `docker stats`

## Troubleshooting

### Container won't start

```bash
# Check logs
docker-compose logs app

# Verify environment variables
docker-compose config

# Check image build errors
docker-compose up --build
```

### Redis connection issues

```bash
# Test Redis connectivity
docker-compose exec app redis-cli -h redis ping

# Or if using external Redis
docker-compose exec app redis-cli -h your_redis_host ping
```

### Port conflicts

```bash
# Change exposed port in docker-compose.yml
ports:
  - "8080:3000"  # Use port 8080 instead of 3000

# Then access at http://your-server:8080
```

### Out of disk space

```bash
# Clean up Docker resources
docker system prune -a

# Remove old images
docker image prune -a

# Remove dangling volumes
docker volume prune
```

## Security Considerations

1. **Environment Variables**: Store sensitive keys securely, use `.env.local` (git-ignored)
2. **Firewall**: Only expose ports 80 and 443, restrict Redis access
3. **Updates**: Regularly update Docker images: `docker-compose pull && docker-compose up -d`
4. **Backups**: Regular Redis backups if data persistence is important
5. **Reverse Proxy**: Always use Nginx or similar as a reverse proxy in production

## Additional Resources

- [Docker Compose Docs](https://docs.docker.com/compose/)
- [Next.js Production Deployment](https://nextjs.org/docs/deployment)
- [Nginx Reverse Proxy](https://nginx.org/en/docs/beginners_guide.html)
