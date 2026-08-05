---
name: "technical-troubleshooting-expert"
description: "Systematic technical troubleshooting methodology for diagnosing and resolving complex technical issues. Use when analyzing performance problems, errors, security incidents, or system-related issues across any technical platform. Includes WordPress-specific troubleshooting using Grafana monitoring, server logs, and WordPress tools. Provides structured debugging approaches and root cause analysis frameworks."
---

# Technical Troubleshooting Expert

A comprehensive troubleshooting skill combining systematic diagnosis methodology with WordPress-specific tooling and diagnostic approaches.

## When to Use This Skill

- Performance degradation or slow response times
- HTTP errors (4xx, 5xx series) and connectivity issues
- Database performance issues or connection problems
- Security incidents or suspicious activity detection
- Plugin/theme conflicts causing site issues
- High resource usage or system overload situations
- User reports of accessibility or functionality problems
- System update or migration issues
- WordPress-specific errors (WSOD, failed updates, AJAX issues)

## Additional Context

- For context about the support environment, tools, and established diagnostic patterns, see [memories.md](memories.md)
- For platform-layer questions (caching, rate limiting, reserved status codes, edge/origin, logs, managed software), consult the `wp-cloud-field-guide` skill — it retrieves authoritative WP Cloud documentation via ContextA8C

## Core Troubleshooting Methodology

### 1. Initial Issue Assessment

Categorize the problem immediately:

**Performance Issues:**

- Response time degradation, resource consumption (CPU, memory, disk I/O)
- Throughput bottlenecks, network latency
- For WordPress: check Grafana Overview (HTTP Response Codes, CPU, Bandwidth)

**Functionality Issues:**

- Application errors or crashes, feature malfunctions
- Integration failures, data integrity problems
- For WordPress: review PHP Error Logs, check plugin/theme conflicts

**Security Concerns:**

- Unauthorized access attempts, suspicious traffic patterns
- Security policy violations, data breach indicators
- For WordPress: examine Mod Security Blocks, SSH Activity logs, Bot vs Not-bot traffic

### 2. Systematic Investigation

#### Step 1: Gather Initial Evidence

- **Reproduce the issue** — verify consistency, document reproduction steps
- **Check monitoring data** — review metrics, logs, and alerts around the time of the issue
- **Identify patterns** — correlate with specific times, users, or actions
- **Assess scope** — all users, specific segments, or individual cases?

#### Step 2: Form Hypotheses

Develop 5-7 potential root causes considering:

- Recent changes (deployments, configuration updates, infrastructure changes)
- Environmental factors (load increases, resource constraints)
- Architectural constraints (scaling limits, design bottlenecks)
- External dependencies (third-party services, APIs)
- Edge cases and unusual usage patterns

**Prioritize by:**

1. Likelihood based on available evidence
2. Impact severity if hypothesis is correct
3. Ease of testing/validation
4. Time required to investigate

#### Step 3: Test Hypotheses Systematically

For each hypothesis (starting with most likely):

- **Design a test** — focused experiment to validate or invalidate
- **Add instrumentation** — logging, metrics, or tracing as needed
- **Execute** — run the experiment in a controlled manner
- **Analyze** — determine if hypothesis is supported or rejected
- **Document** — record findings regardless of outcome

#### Step 4: Root Cause Analysis

Once the immediate cause is identified, apply the **5 Whys**:

1. Why did the issue occur? → [Answer]
2. Why [Answer from 1]? → [Answer]
3. Why [Answer from 2]? → [Answer]
4. Why [Answer from 3]? → [Answer]
5. Why [Answer from 4]? → [Root Cause]

### 3. Solution Development

**Immediate Remediation:**

- Quick fix to restore service or mitigate impact
- Workaround while permanent fix is developed
- Impact assessment — ensure fix doesn't introduce new issues

**Permanent Resolution:**

- Fix the root cause, not just symptoms
- Implement safeguards (checks, validations, circuit breakers)
- Update documentation and runbooks
- Add monitoring to catch similar issues early

## WordPress-Specific Diagnostics

### Available Tools and Capabilities

**Monitoring:** Grafana dashboards, Kibana/Elastic logs, Jetpack Debugger
**Server Access:** SSH with Bash/WP-CLI, SFTP, phpMyAdmin (MariaDB)
**WordPress:** Dashboard access, direct file access

### Grafana-Based Performance Analysis

**Primary Dashboard — Overview (check first):**

1. HTTP Response Codes (track error rates)
2. PHP CPU usage (application performance)
3. HTTP Requests per minute (traffic patterns)
4. Average HTTP Request Time (user experience)

**Secondary Dashboard — Nginx (drill down when issues detected):**

1. Requests by URI (identify problem pages)
2. wp-admin/admin-ajax.php activity (backend load)
3. Top Traffic Type (understand traffic sources)
4. Bot vs Not-bot ratio (security analysis)
5. Top AS Numbers per 1m (traffic origin)
6. Limited Traffic By Reason per 1m

**Tertiary Dashboard — MySQL (database investigation):**

1. MySQL CPU cores (database performance)
2. Changed Rows per minute (write operations)
3. Binary Log Bytes (replication load)

**Log Sources — prioritize by issue type:**

| Issue Type    | Primary Logs                              | Secondary Logs                |
| ------------- | ----------------------------------------- | ----------------------------- |
| Performance   | Nginx Logs, PHP Error Logs                | API Nginx Logs                |
| Security      | Mod Security Blocks/Details, SSH Activity | Bash Logs                     |
| Functionality | PHP Error Logs                            | WordPress debug.log (via SSH) |

### SSH/WP-CLI Diagnostic Commands

**WordPress Health Check:**

```bash
wp core verify-checksums
wp plugin status
wp theme status
wp db check
wp config list
wp option get active_plugins
```

**Performance Profiling:**

```bash
# Enable debugging
wp config set WP_DEBUG true
wp config set WP_DEBUG_LOG true
wp config set WP_DEBUG_DISPLAY false

# Check memory limits
wp eval "echo WP_MEMORY_LIMIT;"
wp eval "echo ini_get('memory_limit');"

# Check cron jobs
wp cron event list
wp cron test

# Monitor real-time errors
tail -f /path/to/wp-content/debug.log
```

**Plugin/Theme Conflict Resolution:**

```bash
# Backup plugin list
wp plugin list --format=json > plugin_backup.json

# Backup database
wp db export backup_$(date +%Y%m%d_%H%M%S).sql

# Test with defaults
wp theme activate twentytwentyfour
wp plugin deactivate --all

# Reactivate strategically, one at a time
wp plugin activate plugin-name
```

### Database Investigation via phpMyAdmin

**Diagnostic SQL Queries:**

```sql
-- Check autoload option sizes
SELECT option_name, LENGTH(option_value)
FROM wp_options
WHERE autoload = 'yes'
ORDER BY LENGTH(option_value) DESC
LIMIT 20;

-- Identify large tables
SELECT table_name,
       ROUND(((data_length + index_length) / 1024 / 1024), 2) AS 'Size (MB)'
FROM information_schema.tables
WHERE table_schema = 'database_name'
ORDER BY (data_length + index_length) DESC;

-- Check active database connections
SHOW PROCESSLIST;
```

## Issue-Specific Troubleshooting Guides

### High wp-admin/admin-ajax.php Activity

1. **Grafana**: Check "wp-admin/admin-ajax.php actions per 1m"
2. **SSH**: `wp config set WP_DEBUG_AJAX true`
3. **Logs**: Monitor PHP error logs for AJAX-specific errors
4. **Nginx**: `tail -f /var/log/nginx/access.log | grep admin-ajax.php`

### White Screen of Death (WSOD)

1. Enable WordPress debugging via WP-CLI
2. Check PHP error logs for fatal errors
3. Test with default theme to rule out theme issues
4. Deactivate plugins to identify conflicts
5. Increase PHP memory limit if needed

### Slow Loading Pages

1. **Grafana**: Check Average HTTP Request Time by page
2. **Enable Query Monitor** plugin for detailed profiling
3. **Review wp-cron** for scheduled task issues
4. **Check external API calls** and timeouts
5. **Analyze database query performance**: `wp db query "SHOW PROCESSLIST;"`

### Failed WordPress Updates

1. Check file permissions via SSH/SFTP
2. Verify disk space: `df -h`
3. Review PHP error logs for update failures
4. Attempt manual update: `wp core update --force` / `wp plugin update --all`

### Security Incident Response

1. **Grafana**: Review Bot vs Not-bot traffic patterns
2. **Logs**: Analyze Mod Security Blocks for attack patterns
3. **SSH audit**:

```bash
wp user list --role=administrator
wp core verify-checksums
wp plugin verify-checksums --all
find /path/to/wordpress -name "*.php" -mtime -7 -exec ls -la {} \;
```

## Logging and Instrumentation

### Effective Logging Strategy

- **Log at appropriate levels** — DEBUG, INFO, WARN, ERROR, FATAL
- **Include context** — User ID, request ID, session ID
- **Timestamp everything** — use consistent timezone (UTC recommended)
- **Structure logs** — JSON or structured format for easy parsing
- **Avoid sensitive data** — never log passwords, tokens, or PII

### Key Metrics to Track

- **Response times** — average, median, 95th/99th percentile
- **Error rates** — by type, endpoint, and user segment
- **Resource utilization** — CPU, memory, disk, network
- **Throughput** — requests per second, transactions per minute

## Communication and Reporting

### Structured Issue Analysis Format

**Initial Assessment:**

- **Issue Description**: Brief summary of reported problem
- **Grafana Baseline**: Current metrics snapshot
- **Severity Level**:
    - Critical: Site down, 500 errors >10%
    - High: Significant slowdown, 500 errors 5-10%
    - Medium: Intermittent issues, 500 errors 1-5%
    - Low: Minor issues, <1% error rate

**Investigation Findings:**

- **Primary Indicators**: Key metrics showing the problem (with specific values and timestamps)
- **Correlating Data**: Supporting evidence from multiple sources
- **Timeline Analysis**: When the issue started and pattern evolution

**Root Cause Analysis:**

- **Identified Cause**: Specific component or configuration issue
- **Supporting Evidence**: Data from monitoring tools and logs
- **Impact Scope**: Affected users, pages, or functionality (with percentages)

**Resolution Plan:**

- **Immediate Actions**: Quick fixes to restore service
- **Permanent Solution**: Long-term fix to prevent recurrence
- **Monitoring Strategy**: Specific metrics to watch post-fix

### Escalation Criteria

**Immediate Escalation Triggers:**

- HTTP error rates > 10% (from Grafana)
- PHP CPU usage > 80% sustained
- Security blocks increasing exponentially
- Database connection failures
- Site completely unavailable for >5 minutes
- Data loss or corruption suspected

**Escalation Information to Provide:**

- Complete timeline and chronology
- All hypotheses tested and results
- Current impact and scope
- Immediate actions taken
- Recommended next steps

## WordPress Maintenance Best Practices

### Optimization Strategies

**Caching Layers:** Page cache → Object cache → Opcode cache → CDN
**Database Optimization:** Regular table optimization, revision cleanup, autoload audit, orphaned data removal
**Plugin Management:** Minimal count, well-maintained plugins, staging testing, complete removal of unused plugins

### Security Hardening

1. Keep WordPress core, plugins, and themes updated
2. Strong passwords and 2FA for admin accounts
3. Limit login attempts
4. Regular security scans and file integrity monitoring
5. Web application firewall (WAF)

### Post-Resolution Follow-up

1. **Monitor Grafana** for 30-60 minutes after fix — verify metrics return to baseline
2. **Update monitoring alerts** based on lessons learned
3. **Document findings** for future reference
4. **Implement preventive measures** (caching, security hardening, query optimization)
