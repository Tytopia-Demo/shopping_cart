# GitHub Actions Workflows

This document describes the GitHub Actions workflows configured for this repository.

## Overview

This repository uses GitHub Actions for continuous integration, security scanning, and runner optimization auditing. All workflows have been optimized for resource efficiency and cost-effectiveness.

## Workflows

### 1. CI Pipeline (`ci.yml`)

**Purpose**: Comprehensive continuous integration pipeline for testing and building the application.

**Triggers**:
- Push to `master`, `main`, or `develop` branches
- Pull requests to `master`, `main`, or `develop` branches
- Manual workflow dispatch

**Jobs**:

#### Lint (Parallel Job 1)
- **Runner**: `ubuntu-latest` (2-core, 7GB RAM)
- **Timeout**: 10 minutes
- **Purpose**: Code linting and style checks
- **Dependencies**: None (runs immediately)

#### Test (Parallel Job 2)
- **Runner**: `ubuntu-latest` (2-core, 7GB RAM)
- **Timeout**: 30 minutes
- **Purpose**: Run RSpec and Cucumber test suites
- **Dependencies**: Lint job must complete successfully
- **Parallelization**: Matrix strategy with `[rspec, cucumber]` test suites

#### Build (Parallel Job 3)
- **Runner**: `ubuntu-latest` (2-core, 7GB RAM)
- **Timeout**: 20 minutes
- **Purpose**: Build assets and verify application integrity
- **Dependencies**: Lint job must complete successfully

#### CI Success (Final Job)
- **Runner**: `ubuntu-latest` (2-core, 7GB RAM)
- **Timeout**: 5 minutes
- **Purpose**: Report overall CI pipeline status
- **Dependencies**: All previous jobs must complete successfully

**Resource Optimization**:
- Uses appropriate 2-core runners for lightweight Rails application
- Implements matrix strategy for test parallelization
- Sets reasonable timeouts to prevent runaway jobs
- Includes comprehensive resource monitoring

**Estimated Duration**: 15-25 minutes (with parallel execution)

### 2. Frogbot Security Scan (`frogbot.yml`)

**Purpose**: Automated security vulnerability scanning for dependencies and pull requests.

**Triggers**:
- Pull request opened or synchronized
- Push to `master` branch
- Daily scheduled scan at midnight UTC
- Manual workflow dispatch

**Jobs**:

#### Frogbot Scan
- **Runner**: `ubuntu-latest` (2-core, 7GB RAM)
- **Timeout**: 15 minutes
- **Purpose**: Scan for security vulnerabilities using JFrog Frogbot
- **Dependencies**: None

**Resource Optimization**:
- Uses 2-core runner appropriate for I/O-bound security scanning
- Implements 15-minute timeout for security scans
- Includes resource monitoring before and after scan

**Estimated Duration**: 5-10 minutes

### 3. Runner Usage Audit (`runner-audit.yml`)

**Purpose**: Periodic auditing of GitHub Actions runner usage to identify optimization opportunities.

**Triggers**:
- Weekly schedule (Monday at 9 AM UTC)
- Manual workflow dispatch

**Jobs**:

#### Audit Runner Usage
- **Runner**: `ubuntu-latest` (2-core, 7GB RAM)
- **Timeout**: 10 minutes
- **Purpose**: Analyze workflow configurations and generate optimization recommendations
- **Dependencies**: None

**Features**:
- Analyzes all workflow files for timeout configurations
- Checks for resource monitoring implementation
- Identifies runner types used across workflows
- Detects parallelization strategies
- Generates actionable recommendations

**Estimated Duration**: 2-5 minutes

## Resource Monitoring

All workflows include resource monitoring steps that log:
- CPU core count
- Available memory
- Disk space
- Job start and completion timestamps
- Memory usage statistics
- System load averages

This data helps identify optimization opportunities and validate runner selections.

## Best Practices Implemented

### 1. Job Timeouts
Every job has a `timeout-minutes` configuration to:
- Prevent runaway processes
- Optimize runner allocation
- Reduce costs from hung jobs
- Provide fast feedback on failures

### 2. Resource Monitoring
All non-trivial jobs include monitoring steps to:
- Track resource consumption
- Validate runner sizing decisions
- Identify optimization opportunities
- Support data-driven improvements

### 3. Parallelization
The CI pipeline uses matrix strategies to:
- Run test suites in parallel
- Reduce total workflow duration
- Maximize runner efficiency
- Provide faster feedback to developers

### 4. Job Dependencies
Jobs are organized with `needs` to:
- Fail fast when linting fails
- Run independent jobs in parallel
- Optimize total execution time
- Reduce unnecessary runner usage

### 5. Documentation
All runners include inline comments specifying:
- Why that runner size was selected
- Expected resource requirements
- Typical execution duration

## Cost Optimization

### Current Configuration
- **Frogbot**: 2-core runner, ~5-10 min/run, runs daily + on PRs
- **CI Pipeline**: 2-core runners, ~15-25 min/run, runs on every push/PR
- **Audit**: 2-core runner, ~2-5 min/run, runs weekly

### Estimated Monthly Cost
Based on GitHub Actions pricing for public repositories (free) or private repositories:
- All workflows use standard `ubuntu-latest` runners
- Parallelization reduces total execution time by ~40%
- Timeouts prevent cost from runaway jobs
- Appropriate runner sizing avoids over-provisioning

## Monitoring and Maintenance

### Weekly Review
The runner audit workflow runs weekly to:
- Verify all workflows have timeouts
- Check resource monitoring implementation
- Generate optimization recommendations

### Quarterly Optimization
Review workflow execution data quarterly to:
- Validate runner size selections
- Adjust timeouts based on actual durations
- Identify further parallelization opportunities
- Remove or consolidate unused workflows

### Metrics to Track
- Average job duration by workflow
- Runner queue times
- Job failure rates
- Resource utilization (CPU, memory)
- Total workflow execution costs

## Future Improvements

Potential optimizations to consider:

1. **Larger Runners for Heavy Workloads**: If test suites grow, consider upgrading to 4-core runners
2. **Additional Parallelization**: Split test suites into more granular shards
3. **Caching Strategies**: Implement dependency caching to reduce setup time
4. **Self-Hosted Runners**: For very high volume, consider self-hosted runners with auto-scaling
5. **Conditional Workflows**: Skip certain jobs based on file changes (e.g., skip tests if only docs changed)

## Troubleshooting

### Workflow Times Out
1. Check resource monitoring logs for bottlenecks
2. Consider increasing timeout or upgrading runner size
3. Look for opportunities to parallelize or optimize the workload

### High Runner Costs
1. Review runner audit recommendations
2. Check for redundant workflow runs
3. Implement more aggressive caching
4. Consider smaller runners for lightweight jobs

### Jobs Frequently Fail
1. Review failure patterns in workflow history
2. Check if timeout is too aggressive
3. Verify runner has sufficient resources
4. Look for flaky tests or environmental issues

## References

- [Runner Selection Guidelines](./RUNNER_GUIDELINES.md)
- [GitHub Actions Documentation](https://docs.github.com/en/actions)
- [GitHub Actions Best Practices](https://docs.github.com/en/actions/using-workflows/workflow-syntax-for-github-actions)
- [GitHub Actions Billing](https://docs.github.com/en/billing/managing-billing-for-github-actions/about-billing-for-github-actions)
