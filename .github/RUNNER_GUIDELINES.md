# GitHub Actions Runner Selection Guidelines

This document provides guidelines for selecting appropriate GitHub Actions runner types based on workflow resource requirements to optimize costs and performance.

## Available Runner Types

### GitHub-Hosted Runners

| Runner Type | vCPUs | RAM | Storage | Use Cases |
|------------|-------|-----|---------|-----------|
| `ubuntu-latest` | 2 | 7 GB | 14 GB | Standard workflows, linting, small test suites |
| `ubuntu-latest-4-core` | 4 | 16 GB | 14 GB | Medium workloads, parallel testing, builds |
| `ubuntu-latest-8-core` | 8 | 32 GB | 14 GB | Large test suites, heavy builds, resource-intensive tasks |
| `ubuntu-latest-16-core` | 16 | 64 GB | 14 GB | Very large workloads, extensive parallel operations |

## Workflow Categorization

### Small Resource Tier (ubuntu-latest - 2 cores, 7 GB RAM)

**Characteristics:**
- Execution time: < 10 minutes
- CPU usage: Low
- Memory usage: < 4 GB
- Minimal I/O operations

**Workflow Types:**
- Code linting and style checks
- Security scans (Frogbot, Snyk, etc.)
- Documentation generation
- Small unit test suites
- Configuration validation
- Dependency audits

**Example Configuration:**
```yaml
jobs:
  lint:
    runs-on: ubuntu-latest
    timeout-minutes: 10
    steps:
      - uses: actions/checkout@v3
      - name: Run linter
        run: make lint
```

### Medium Resource Tier (ubuntu-latest-4-core - 4 cores, 16 GB RAM)

**Characteristics:**
- Execution time: 10-30 minutes
- CPU usage: Medium
- Memory usage: 4-12 GB
- Moderate parallelization

**Workflow Types:**
- Medium-sized test suites
- Application builds with asset compilation
- Docker image builds
- Integration tests
- Database migrations and seeding

**Example Configuration:**
```yaml
jobs:
  test:
    runs-on: ubuntu-latest-4-core
    timeout-minutes: 30
    strategy:
      matrix:
        test-type: [unit, integration]
    steps:
      - uses: actions/checkout@v3
      - name: Run tests
        run: make test-${{ matrix.test-type }}
```

### Large Resource Tier (ubuntu-latest-8-core - 8 cores, 32 GB RAM)

**Characteristics:**
- Execution time: 30-60 minutes
- CPU usage: High
- Memory usage: 12-28 GB
- Heavy parallelization

**Workflow Types:**
- Large test suites with extensive parallelization
- Complex builds (multi-stage Docker builds)
- End-to-end test suites
- Performance testing
- Large-scale data processing

**Example Configuration:**
```yaml
jobs:
  e2e-tests:
    runs-on: ubuntu-latest-8-core
    timeout-minutes: 60
    strategy:
      matrix:
        shard: [1, 2, 3, 4, 5, 6, 7, 8]
    steps:
      - uses: actions/checkout@v3
      - name: Run E2E tests
        run: make e2e-test-shard-${{ matrix.shard }}
```

## Best Practices

### 1. Always Set Timeouts

Prevent runaway jobs and optimize runner allocation:
```yaml
jobs:
  my-job:
    timeout-minutes: 30  # Always specify this
```

### 2. Implement Resource Monitoring

Add resource monitoring steps to track usage:
```yaml
steps:
  - name: Log resource usage
    run: |
      echo "CPU cores: $(nproc)"
      echo "Memory: $(free -h | grep Mem | awk '{print $2}')"
      echo "Started at: $(date -u +%Y-%m-%dT%H:%M:%SZ)"
```

### 3. Optimize Job Parallelization

Use matrix strategies to distribute work:
```yaml
strategy:
  matrix:
    test-suite: [unit, integration, e2e]
  fail-fast: false  # Continue running other jobs if one fails
```

### 4. Use Job Dependencies

Organize workflows efficiently:
```yaml
jobs:
  lint:
    runs-on: ubuntu-latest

  test:
    needs: lint  # Only run after lint succeeds
    runs-on: ubuntu-latest-4-core

  deploy:
    needs: [lint, test]  # Wait for both to complete
    runs-on: ubuntu-latest
```

### 5. Cache Dependencies

Reduce build times and resource usage:
```yaml
steps:
  - uses: actions/checkout@v3

  - name: Cache dependencies
    uses: actions/cache@v3
    with:
      path: vendor/bundle
      key: ${{ runner.os }}-gems-${{ hashFiles('**/Gemfile.lock') }}

  - name: Install dependencies
    run: bundle install --jobs 4 --retry 3
```

### 6. Fail Fast When Appropriate

For blocking workflows, fail fast to save resources:
```yaml
strategy:
  fail-fast: true  # Stop all jobs if one fails
```

## Runner Selection Decision Tree

```
Start
  |
  ├─ Linting/Style checks? → ubuntu-latest (2-core)
  │
  ├─ Security scanning? → ubuntu-latest (2-core)
  │
  ├─ Small test suite (< 5 min)? → ubuntu-latest (2-core)
  │
  ├─ Medium test suite (5-20 min)? → ubuntu-latest-4-core
  │  └─ Can parallelize? → Yes: Use matrix strategy
  │
  ├─ Large test suite (20-45 min)? → ubuntu-latest-8-core
  │  └─ Use matrix strategy with 4-8 shards
  │
  ├─ Heavy build process? → ubuntu-latest-4-core or -8-core
  │  └─ Use caching aggressively
  │
  └─ E2E tests or performance tests? → ubuntu-latest-8-core
     └─ Use matrix strategy for parallel execution
```

## Monitoring and Optimization

### Metrics to Track

1. **Job Duration**: Track how long each job takes
2. **CPU Usage**: Monitor peak and average CPU utilization
3. **Memory Usage**: Track peak memory consumption
4. **Queue Time**: Monitor how long jobs wait for runners
5. **Cost**: Calculate runner costs based on execution time

### Optimization Cycle

1. **Audit**: Run workflows with resource monitoring enabled
2. **Analyze**: Review logs to identify over/under-provisioned runners
3. **Adjust**: Update runner types based on actual resource usage
4. **Validate**: Verify improvements in execution time and cost
5. **Repeat**: Continuously monitor and optimize

## Examples from This Repository

### Frogbot Security Scan
- **Runner**: `ubuntu-latest` (2-core)
- **Timeout**: 15 minutes
- **Rationale**: Security scanning is I/O bound with low CPU/memory needs

### RSpec/Cucumber Tests
- **Runner**: `ubuntu-latest` (2-core)
- **Timeout**: 30 minutes
- **Rationale**: Test suite size is small-to-medium; 2 cores sufficient
- **Optimization**: Parallelized using matrix strategy

### Asset Compilation
- **Runner**: `ubuntu-latest` (2-core)
- **Timeout**: 20 minutes
- **Rationale**: Asset precompilation is moderately CPU intensive but short-lived

## Additional Resources

- [GitHub Actions Runner Images](https://github.com/actions/runner-images)
- [GitHub Actions Billing](https://docs.github.com/en/billing/managing-billing-for-github-actions/about-billing-for-github-actions)
- [Optimizing GitHub Actions Workflows](https://docs.github.com/en/actions/using-workflows/workflow-syntax-for-github-actions)

## Deprecation Policy

Workflows that have not run in the past 90 days should be reviewed and considered for deprecation. To identify unused workflows:

```bash
# Review workflow run history in GitHub Actions UI
# Or use GitHub CLI:
gh run list --workflow=<workflow-name> --limit=100
```

Workflows to deprecate should be:
1. Commented with deprecation notice
2. Disabled in repository settings
3. Removed after 30-day grace period

---

**Last Updated**: 2024
**Owner**: DevOps Team
**Review Cycle**: Quarterly
