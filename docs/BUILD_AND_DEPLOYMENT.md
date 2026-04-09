# SQLDeployer Build and Deployment Guide

## Overview

This guide covers building, packaging, and deploying SQLDeployer for both Windows and Linux platforms, including Docker containerization.

## Prerequisites

- .NET 8.0 SDK
- Docker (for containerization)
- Git

## Build Scripts

### Windows Build

#### Batch Script (Windows)
```bash
scripts\build-windows.bat
```

#### Shell Script (Cross-platform)
```bash
scripts/build-windows.sh
```

**Output:**
- `DEPLOYMENT/windows/publish/` - Self-contained executable
- `DEPLOYMENT/windows/SQLDeployer.Windows.zip` - Deployment package

### Linux Build

#### Batch Script (Windows)
```bash
scripts\build-linux.bat
```

#### Shell Script (Cross-platform)
```bash
scripts/build-linux.sh
```

**Output:**
- `DEPLOYMENT/linux/publish/` - Self-contained executable
- `DEPLOYMENT/linux/SQLDeployer.Linux.tar.gz` - Deployment package

## Docker Containerization

### Build Docker Image

```bash
# Basic build
scripts/docker-build.sh

# With custom tag
scripts/docker-build.sh sqldeployer v1.0.0

# With registry
scripts/docker-build.sh sqldeployer v1.0.0 myregistry.com
```

### Run Docker Container

```bash
# Basic run
scripts/docker-run.sh

# With custom parameters
scripts/docker-run.sh sqldeployer:latest my-container ./data ./output ./logs
```

### Push to Registry

```bash
# Push to registry
scripts/docker-push.sh sqldeployer v1.0.0 myregistry.com
```

### Complete Workflow

```bash
# Build, test, and push
scripts/docker-complete.sh sqldeployer v1.0.0 myregistry.com
```

## Docker Compose

### Start with Docker Compose

```bash
# Start services (from project root)
docker-compose -f docker/docker-compose.yml up -d

# View logs
docker-compose -f docker/docker-compose.yml logs -f

# Stop services
docker-compose -f docker/docker-compose.yml down
```

### Environment Configuration

1. Copy `env/sqldeployer.env.sh.example` to `env/sqldeployer.env.sh`
2. Modify configuration as needed
3. Mount the file in Docker: `-v /path/to/env/sqldeployer.env.sh:/etc/env/sqldeployer.env.sh:ro`

## Directory Structure

```
scripts/
├── build-windows.bat      # Windows build (batch)
├── build-windows.sh       # Windows build (shell)
├── build-linux.bat        # Linux build (batch)
├── build-linux.sh         # Linux build (shell)
├── docker-build.sh        # Docker build
├── docker-run.sh          # Docker run
├── docker-push.sh         # Docker push
└── docker-complete.sh     # Complete workflow

env/
├── sqldeployer.env.sh.example  # Environment template
└── sqldeployer.env.sh          # Environment config

DEPLOYMENT/
├── windows/
│   ├── publish/           # Windows executable
│   └── SQLDeployer.Windows.zip
└── linux/
    ├── publish/           # Linux executable
    └── SQLDeployer.Linux.tar.gz
```

## Environment Variables

| Variable | Description | Default |
|----------|-------------|---------|
| `APP_NAME` | Application name | SQLDeployer |
| `DATABASE_TYPE` | Database type | sqlserver |
| `CONNECTION_STRING` | Database connection | - |
| `SQL_FILES_DIRECTORY` | SQL files path | /app/data |
| `OUTPUT_PATH` | Output directory | /app/output |
| `LOGS_PATH` | Logs directory | /app/logs |
| `LOG_LEVEL` | Logging level | Information |
| `TRANSACTION_HANDLING` | Enable transactions | false |

## Security Considerations

- Container runs as non-root user (sqldeployer:1001)
- Read-only filesystem where possible
- No new privileges allowed
- Resource limits applied
- Health checks configured

## Testing

### Test Categories

SqlDeployer includes two categories of tests:

#### Fast Unit Tests (Default - ~2-5 minutes)
- Pure unit tests without external dependencies
- No Docker required
- Recommended for development and PR validation

**Projects excluded from fast tests:**
- `Unit/Docker/*` - Docker integration tests (6 projects)
- `Unit/CommandLine/*` - CLI integration tests (6 projects)

**Usage:**
```bash
# Default behavior - fast tests only
./scripts/_buildAndPublishAndReleaseAll.sh

# Or explicitly
CONTAINER_TESTS=false ./scripts/_buildAndPublishAndReleaseAll.sh
```

#### Full Integration Tests (~15-30 minutes)
- All unit tests + TestContainers integration tests
- Requires Docker daemon running
- Recommended for releases and main branch merges

**Includes all tests:**
- 9 fast unit test projects
- 12 TestContainers integration test projects (Docker/CommandLine)

**Usage:**
```bash
# Run all tests including TestContainers
./scripts/_buildAndPublishAndReleaseAll.sh --container-tests

# Or via environment variable
CONTAINER_TESTS=true ./scripts/_buildAndPublishAndReleaseAll.sh
```

### Test Configuration

Default test mode is configured in `scripts/_common.sh`:

```bash
# Default: false - run only fast unit tests
CONTAINER_TESTS="${CONTAINER_TESTS:-false}"
```

To change the default behavior, edit this variable in `_common.sh`.

### CI/CD Recommendations

| Pipeline Stage | Test Mode | Duration | Docker Required |
|---------------|-----------|----------|----------------|
| Pull Request | Fast (default) | ~10-15 min | No |
| Main Branch | Full (`--container-tests`) | ~35-45 min | Yes |
| Release Build | Full (`--container-tests`) | ~35-45 min | Yes |
| Nightly Build | Full (`--container-tests`) | ~35-45 min | Yes |

### Test Mode

Test mode validates the entire workflow without publishing locally:

```bash
# Fast tests in test mode
./scripts/_buildAndPublishAndReleaseAll.sh --test

# Full tests in test mode
./scripts/_buildAndPublishAndReleaseAll.sh --test --container-tests
```

Test mode:
1. Builds for all platforms
2. Runs tests (fast or full)
3. Creates GitHub release
4. Verifies release
5. **Deletes the release**
6. Skips local publishing

For comprehensive testing documentation, see [TEST_CONFIGURATION.md](TEST_CONFIGURATION.md).

## Troubleshooting

### Build Issues

1. **Missing .NET SDK**: Install .NET 8.0 SDK
2. **Permission denied**: Ensure scripts are executable (`chmod +x scripts/*.sh`)
3. **Docker build fails**: Check docker/Dockerfile and .dockerignore

### Runtime Issues

1. **Container won't start**: Check environment variables and volume mounts
2. **Permission denied**: Ensure proper ownership of mounted directories
3. **Database connection**: Verify connection string and network access

### Logs

- Application logs: `./logs/` directory
- Docker logs: `docker logs sqldeployer`
- Compose logs: `docker-compose -f docker/docker-compose.yml logs -f`

## Performance Tuning

- Adjust `MAX_CONCURRENT_OPERATIONS` for parallel processing
- Set appropriate `COMMAND_TIMEOUT` and `CONNECTION_TIMEOUT`
- Monitor resource usage with `docker stats`

## Monitoring

- Health check endpoint: Container health status
- Logs: Structured JSON logging
- Metrics: Enable with `ENABLE_METRICS=true`
