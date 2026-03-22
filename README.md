# MCP Proxy

A production-ready proxy system with Redis caching, email server, and Nginx reverse proxy.

## Architecture

- **Redis**: In-memory data store for caching and session management
- **Email Server**: MCP email service for handling email operations
- **Nginx**: Reverse proxy for SSL termination and load balancing

## Prerequisites

- Docker and Docker Compose
- SSL certificates (Let's Encrypt or custom)

## Quick Start

### 1. Clone the repository

```bash
git clone <repository-url>
cd mcp-proxy
```

### 2. Configure environment variables

```bash
# Copy example environment files
cp .env.redis.example .env.redis
cp .env.email.example .env.email
```

Edit the environment files with your configuration:

**`.env.redis`:**
```env
REDIS_PORT=6379
REDIS_PASSWORD=your_secure_redis_password_here
```

**`.env.email`:**
```env
REDIS_HOST=redis
REDIS_PORT=6379
REDIS_PASSWORD=your_secure_redis_password_here
# Add other email server configuration
```

### 3. Configure Nginx

Update `nginx.conf` with your domain and SSL certificate paths.

### 4. Build and start services

```bash
# Build the email server image
docker build -t email-mcp .

# Start all services
docker-compose -f docker-compose.prod.yml up -d
```

## Service Startup Order

1. **Redis** starts first (with health check)
2. **Email Server** starts after Redis is healthy
3. **Nginx** starts after Email Server is running

## Network Configuration

### Internal Network
- **Purpose**: Private communication between services
- **Services**: Redis ↔ Email Server
- **Access**: Not accessible from outside Docker network

### Public Network
- **Purpose**: Public-facing traffic
- **Services**: Nginx
- **Ports**: 80 (HTTP), 443 (HTTPS)

### Security
- Redis and Email Server are **not directly accessible** from the host
- Only Nginx exposes public ports (80, 443)
- Internal services communicate via Docker's internal DNS

## Service Details

### Redis
- **Image**: `redis:7-alpine`
- **Port**: Configurable via `REDIS_PORT` (default: 6379)
- **Authentication**: Password protected via `REDIS_PASSWORD`
- **Persistence**: AOF (Append Only File) enabled
- **Health Check**: Automatic health monitoring

### Email Server
- **Image**: `email-mcp` (custom built)
- **Internal Port**: 8002
- **Access**: Only via Nginx proxy
- **Dependencies**: Requires Redis to be healthy

### Nginx
- **Image**: `nginx:alpine`
- **Ports**: 80 (HTTP), 443 (HTTPS)
- **SSL**: Configured with Let's Encrypt certificates
- **Configuration**: Mounted from `./nginx.conf`

## Docker Commands

```bash
# Start services
docker-compose -f docker-compose.prod.yml up -d

# Stop services
docker-compose -f docker-compose.prod.yml down

# View logs
docker-compose -f docker-compose.prod.yml logs -f

# View specific service logs
docker-compose -f docker-compose.prod.yml logs -f redis
docker-compose -f docker-compose.prod.yml logs -f email-server
docker-compose -f docker-compose.prod.yml logs -f nginx

# Restart a service
docker-compose -f docker-compose.prod.yml restart email-server

# Rebuild and restart
docker-compose -f docker-compose.prod.yml up -d --build
```

## Redis Connection

Services can connect to Redis using:

**Connection String:**
```
redis://:${REDIS_PASSWORD}@redis:${REDIS_PORT}
```

**Connection Details:**
- **Host**: `redis` (Docker service name)
- **Port**: From `REDIS_PORT` env variable
- **Password**: From `REDIS_PASSWORD` env variable

## Data Persistence

### Redis Data
- Volume: `redis-data`
- Location: `/data` in container
- Persistence: AOF enabled

## Troubleshooting

### Redis connection issues
```bash
# Check Redis health
docker exec mcp-redis redis-cli -a <password> ping

# View Redis logs
docker-compose -f docker-compose.prod.yml logs redis
```

### Email server not starting
```bash
# Check if Redis is healthy
docker ps

# View email server logs
docker-compose -f docker-compose.prod.yml logs email-server
```

### Nginx issues
```bash
# Test Nginx configuration
docker exec mcp-proxy-nginx-prod nginx -t

# Reload Nginx configuration
docker exec mcp-proxy-nginx-prod nginx -s reload
```

## Security Best Practices

1. **Never commit sensitive files** (`.env`, `.env.redis`, `.env.email`)
2. **Use strong passwords** for Redis authentication
3. **Keep SSL certificates secure** and renew before expiration
4. **Regularly update Docker images** for security patches
5. **Monitor logs** for suspicious activity

## Development

For development, you can expose the email server port:

```yaml
email-server:
  ports:
    - "8002:8002"
```

This allows direct access to `localhost:8002` for testing.

## License

[Your License Here]
