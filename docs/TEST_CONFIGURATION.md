# Test Configuration Guide

## Overview

SqlDeployer has two categories of tests:

1. **Fast Unit Tests** - Pure unit tests without external dependencies (default)
2. **TestContainers Integration Tests** - Integration tests that require Docker and spin up real database containers (optional)

## Test Categories

### Fast Unit Tests (Default)
Located in:
- `Unit/Basic_tests/` - Basic functionality tests
- `Unit/SqlServer/` - SQL Server specific tests (without containers)
- `Unit/PostgreSQL/` - PostgreSQL specific tests (without containers)
- `Unit/MariaDB/` - MariaDB specific tests (without containers)
- `Unit/Oracle/` - Oracle specific tests (without containers, skipped by default)
- `Unit/Sqlite/` - SQLite specific tests

**Execution time:** ~2-5 minutes  
**Docker required:** No

### TestContainers Integration Tests (Optional)
Located in:
- `Unit/Docker/*` - Docker-based integration tests with TestContainers
  - `Docker.SqlServer/`
  - `Docker.PostgreSQL/`
  - `Docker.MariaDB/`
  - `Docker.Oracle/`
  - `Docker.Sqlite/`
- `Unit/CommandLine/*` - CLI integration tests with TestContainers
  - `CommandLine.SqlServer/`
  - `CommandLine.PostgreSQL/`
  - `CommandLine.MariaDB/`
  - `CommandLine.Oracle/`
  - `CommandLine.Sqlite/`

**Execution time:** ~15-30 minutes  
**Docker required:** Yes  
**Requirements:**
- Docker Desktop or Docker Engine running
- Sufficient memory (minimum 4GB RAM allocated to Docker)
- Internet connection to pull container images

## Configuration

### Global Configuration (scripts/_common.sh)

The `CONTAINER_TESTS` variable controls which tests are executed:

```bash
# scripts/_common.sh

# Controls whether to run integration tests that require TestContainers (Docker)
# Values:
#   false (default) - Run only fast unit tests, skip TestContainers integration tests
#   true            - Run all tests including TestContainers integration tests
CONTAINER_TESTS="${CONTAINER_TESTS:-false}"
```

### Changing Default Behavior

To change the default behavior for all script invocations, edit `scripts/_common.sh`:

```bash
# Enable TestContainers tests by default
CONTAINER_TESTS="${CONTAINER_TESTS:-true}"
```

## Usage

### Running Fast Tests Only (Default)

```bash
# Build, test (fast only), and release
./scripts/_buildAndPublishAndReleaseAll.sh
```

This will:
- ✅ Run unit tests (~2-5 minutes)
- ⊘ Skip TestContainers integration tests
- ⊘ Skip tests requiring Docker

### Running All Tests (Including TestContainers)

```bash
# Build, test (all including TestContainers), and release
./scripts/_buildAndPublishAndReleaseAll.sh --container-tests
```

This will:
- ✅ Run unit tests
- ✅ Run TestContainers integration tests (~15-30 minutes)
- ✅ Require Docker to be running

### Running Only Tests (No Build/Release)

```bash
# Fast tests only
dotnet test --configuration Release --verbosity normal --filter "Category!=Oracle"

# With TestContainers (requires Docker)
CONTAINER_TESTS=true dotnet test --configuration Release --verbosity normal --filter "Category!=Oracle"
```

### Environment Variable Override

```bash
# Temporary override for single execution
CONTAINER_TESTS=true ./scripts/_buildAndPublishAndReleaseAll.sh

# Or export for session
export CONTAINER_TESTS=true
./scripts/_buildAndPublishAndReleaseAll.sh
```

## CI/CD Recommendations

### Fast CI Pipeline (Pull Requests)

```yaml
# GitHub Actions / Azure DevOps
- name: Run Fast Tests
  run: ./scripts/_buildAndPublishAndReleaseAll.sh
  # CONTAINER_TESTS defaults to false
```

**Use case:** PR validation, quick feedback loops  
**Time:** ~10-15 minutes total (build + tests)

### Full CI Pipeline (Main Branch / Releases)

```yaml
# GitHub Actions / Azure DevOps
- name: Run All Tests
  run: ./scripts/_buildAndPublishAndReleaseAll.sh --container-tests
```

**Use case:** Main branch merges, release builds, nightly builds  
**Time:** ~30-45 minutes total (build + all tests)

### Hybrid Approach

```yaml
# Run fast tests on PR
on:
  pull_request:
    branches: [ main, develop ]

jobs:
  fast-tests:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - name: Fast Tests
        run: ./scripts/_buildAndPublishAndReleaseAll.sh

# Run full tests on merge to main
on:
  push:
    branches: [ main ]

jobs:
  full-tests:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - name: Full Tests with TestContainers
        run: ./scripts/_buildAndPublishAndReleaseAll.sh --container-tests
```

## Test Mode

Test mode (`--test`) executes the entire workflow including release creation and deletion for validation:

```bash
# Fast tests in test mode
./scripts/_buildAndPublishAndReleaseAll.sh --test

# All tests in test mode
./scripts/_buildAndPublishAndReleaseAll.sh --test --container-tests
```

Test mode:
1. Builds for all platforms
2. Runs tests (fast or all, depending on flag)
3. Creates GitHub release
4. Verifies release
5. **Deletes the release**
6. Skips local publishing

## Troubleshooting

### TestContainers Tests Fail

**Problem:** Tests fail with "Cannot connect to Docker daemon"

**Solution:**
1. Ensure Docker Desktop is running
2. Check Docker daemon is accessible: `docker ps`
3. Verify Docker resource limits (minimum 4GB RAM)
4. If using Docker Desktop on macOS, ensure it's fully started

### Tests Take Too Long

**Problem:** Full test suite takes 30+ minutes

**Solution:**
1. Use `--container-tests` only when necessary (releases, nightly builds)
2. For PR validation, use fast tests only (default)
3. Consider running TestContainers tests only on main branch in CI/CD

### Oracle Tests Skipped

**Problem:** Oracle tests are always skipped

**Reason:** Oracle TestContainers require special licensing and large container images (~2GB)

**Solution:** Oracle tests are marked with `Category=Oracle` and skipped by default. To enable:
```bash
# Remove the Skip trait from Oracle test files
# Or run with custom filter
dotnet test --filter "Category!=None"  # Will include Oracle
```

## Summary

| Mode | Command | Tests Run | Duration | Docker Required |
|------|---------|-----------|----------|----------------|
| **Default (Fast)** | `./scripts/_buildAndPublishAndReleaseAll.sh` | Unit tests only | ~2-5 min | No |
| **Full Tests** | `./scripts/_buildAndPublishAndReleaseAll.sh --container-tests` | All tests | ~15-30 min | Yes |
| **Test Mode Fast** | `./scripts/_buildAndPublishAndReleaseAll.sh --test` | Unit tests + release validation | ~10-15 min | No |
| **Test Mode Full** | `./scripts/_buildAndPublishAndReleaseAll.sh --test --container-tests` | All tests + release validation | ~30-45 min | Yes |

## Best Practices

1. **Development:** Use fast tests only for quick iteration
2. **PR Validation:** Use fast tests for quick feedback
3. **Main Branch:** Use full tests to ensure integration quality
4. **Releases:** Always use full tests (`--container-tests`)
5. **Nightly Builds:** Use full tests to catch integration issues early
6. **Local Testing:** Use fast tests unless debugging integration issues
