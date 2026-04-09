# SQLDeployer Docker Setup Guide

## Overview

This guide provides detailed instructions for containerizing and running SQLDeployer using Docker.

## Dockerfile Details

### Multi-stage Build

The Dockerfile uses a multi-stage build process:

1. **Build Stage**: Uses `mcr.microsoft.com/dotnet/sdk:8.0.101-alpine3.18`
   - Compiles the application
   - Publishes self-contained executable
   - Target platform: `linux/amd64`

2. **Runtime Stage**: Uses `mcr.microsoft.com/dotnet/runtime-deps:8.0.101-alpine3.18`
   - Minimal runtime environment
   - Non-root user execution
   - Security hardening

### Security Features

- **Non-root user**: Runs as `sqldeployer:1001`
- **Read-only filesystem**: Where possible
- **No new privileges**: Security option enabled
- **Resource limits**: CPU and memory constraints
- **Health checks**: Built-in health monitoring

## Quick Start

### 1. Build Image

```bash
# Basic build
./scripts/docker-build.sh

# With custom tag
./scripts/docker-build.sh sqldeployer v1.0.0

# With registry
./scripts/docker-build.sh sqldeployer v1.0.0 myregistry.com
```

### 2. Run Container

```bash
# Basic run
./scripts/docker-run.sh

# With custom parameters
./scripts/docker-run.sh sqldeployer:latest my-container ./data ./output ./logs
```

### 3. Using Docker Compose

```bash
# Start services (from project root)
docker-compose -f docker/docker-compose.yml up -d

# View logs
docker-compose -f docker/docker-compose.yml logs -f sqldeployer

# Stop services
docker-compose -f docker/docker-compose.yml down
```

## Environment Configuration

### Environment File

Create `env/sqldeployer.env.sh`:

```bash
# Copy template
cp env/sqldeployer.env.sh.example env/sqldeployer.env.sh

# Edit configuration
nano env/sqldeployer.env.sh
```

### Key Environment Variables

```bash
# Database Configuration
export DATABASE_TYPE="sqlserver"
export CONNECTION_STRING="Server=localhost;Database=MyDB;Integrated Security=true;"

# File Paths
export SQL_FILES_DIRECTORY="/app/data"
export OUTPUT_PATH="/app/output"
export LOGS_PATH="/app/logs"

# Logging
export LOG_LEVEL="Information"
export LOG_FORMAT="json"
```

## Volume Mounts

### Required Directories

```bash
# Create local directories
mkdir -p ./data ./output ./logs

# Mount in Docker
docker run -v ./data:/app/data \
           -v ./output:/app/output \
           -v ./logs:/app/logs \
           sqldeployer:latest
```

### Docker Compose Volumes

```yaml
volumes:
  - ./data:/app/data:rw
  - ./output:/app/output:rw
  - ./logs:/app/logs:rw
  - /etc/env/sqldeployer.env.sh:/etc/env/sqldeployer.env.sh:ro
```

## Network Configuration

### Ports

- **8080**: Future web interface (if implemented)

### Networks

```yaml
networks:
  sqldeployer-network:
    driver: bridge
```

## Resource Management

### Limits

```yaml
deploy:
  resources:
    limits:
      cpus: '1.0'
      memory: 512M
    reservations:
      cpus: '0.5'
      memory: 256M
```

### Health Checks

```yaml
healthcheck:
  test: ["CMD", "/app/SqlDeployer", "--version"]
  interval: 30s
  timeout: 10s
  retries: 3
  start_period: 5s
```

## Registry Operations

### Push to Registry

```bash
# Login to registry
docker login myregistry.com

# Push image
./scripts/docker-push.sh sqldeployer v1.0.0 myregistry.com
```

### Pull from Registry

```bash
# Pull image
docker pull myregistry.com/sqldeployer:v1.0.0

# Run pulled image
docker run myregistry.com/sqldeployer:v1.0.0
```

## Troubleshooting

### Common Issues

1. **Permission Denied**
   ```bash
   # Fix directory permissions
   sudo chown -R 1001:1001 ./data ./output ./logs
   ```

2. **Container Won't Start**
   ```bash
   # Check logs
   docker logs sqldeployer
   
   # Check environment
   docker run --rm sqldeployer:latest env
   ```

3. **Database Connection Issues**
   ```bash
   # Test connection string
   docker run --rm sqldeployer:latest --connstring "your-connection-string" --version
   ```

### Debugging

```bash
# Interactive shell
docker run -it --rm sqldeployer:latest /bin/sh

# Check file permissions
docker run --rm sqldeployer:latest ls -la /app

# Test application
docker run --rm sqldeployer:latest --help
```

## Production Deployment

### Security Checklist

- [ ] Non-root user execution
- [ ] Read-only filesystem where possible
- [ ] Resource limits applied
- [ ] Health checks configured
- [ ] Secrets management implemented
- [ ] Network policies applied

### Monitoring

```bash
# Container stats
docker stats sqldeployer

# Health status
docker inspect sqldeployer --format='{{.State.Health.Status}}'

# Resource usage
docker exec sqldeployer ps aux
```

## Advanced Configuration

### Custom Entrypoint

```dockerfile
# Custom entrypoint script
COPY docker-entrypoint.sh /usr/local/bin/
RUN chmod +x /usr/local/bin/docker-entrypoint.sh
ENTRYPOINT ["/usr/local/bin/docker-entrypoint.sh"]
```

### Multi-architecture Build

```bash
# Build for multiple architectures
docker buildx build --platform linux/amd64,linux/arm64 -t sqldeployer:latest .
```

### CI/CD Integration

```yaml
# GitHub Actions example
- name: Build and push Docker image
  uses: docker/build-push-action@v4
  with:
    context: .
    platforms: linux/amd64
    push: true
    tags: myregistry.com/sqldeployer:latest
```
