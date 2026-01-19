# Threat Model - brpoplpush-redis_script

**Version:** 1.0
**Last Updated:** 2024-01-19
**Document Status:** Active
**Next Review Date:** 2024-07-19

## Table of Contents

1. [Executive Summary](#executive-summary)
2. [System Architecture Overview](#system-architecture-overview)
3. [Assets and Data Classification](#assets-and-data-classification)
4. [Threat Analysis (STRIDE)](#threat-analysis-stride)
5. [Attack Vectors and Entry Points](#attack-vectors-and-entry-points)
6. [Existing Security Controls](#existing-security-controls)
7. [Threat Mitigation Recommendations](#threat-mitigation-recommendations)
8. [Review and Maintenance](#review-and-maintenance)
9. [References](#references)

## Executive Summary

This document provides a comprehensive threat model for the `brpoplpush-redis_script` Ruby gem, which enables applications to load and execute Lua scripts in Redis. As a library that bridges application code with Redis server-side execution, it presents unique security considerations around script injection, file system access, and Redis command execution.

### Key Security Concerns

- **Lua Script Injection:** Malicious code execution via unsanitized script paths or content
- **File System Access:** Path traversal vulnerabilities when loading scripts
- **Redis Command Abuse:** Unauthorized Redis commands through script execution
- **ERB Template Processing:** Code injection through template rendering
- **Dependency Chain:** Security of third-party dependencies (Redis client, Concurrent Ruby)

## System Architecture Overview

### Components

The gem consists of the following key components:

1. **DSL Module (`Brpoplpush::RedisScript::DSL`)**: Configuration interface for users
2. **Client (`Brpoplpush::RedisScript::Client`)**: Orchestrates script execution
3. **Scripts Collection (`Brpoplpush::RedisScript::Scripts`)**: Manages loaded scripts
4. **Script (`Brpoplpush::RedisScript::Script`)**: Represents individual Lua scripts
5. **Template (`Brpoplpush::RedisScript::Template`)**: ERB-based partial rendering for Lua files
6. **Configuration (`Brpoplpush::RedisScript::Config`)**: Stores gem configuration

### Data Flow

```
User Application
    ↓
DSL.configure(scripts_path: "/path/to/scripts")
    ↓
DSL.execute(script_name, redis_conn, keys:, argv:)
    ↓
Client.execute → Scripts.execute → Script.load → Redis.script(:load)
    ↓
Redis.evalsha(sha, keys, argv)
    ↓
Result returned to application
```

### Trust Boundaries

1. **Application → Gem**: Trust boundary between user code and gem code
2. **Gem → File System**: File system access for reading Lua scripts
3. **Gem → Redis**: Network boundary to Redis server
4. **Template Rendering**: ERB processing of Lua scripts

### External Dependencies

- **Redis**: In-memory data store (versions 1.0 to < 6)
- **Concurrent Ruby**: Thread-safe data structures (~> 1.0)
- **ERB**: Ruby standard library for template processing

## Assets and Data Classification

### Critical Assets

| Asset | Description | Sensitivity | Impact if Compromised |
|-------|-------------|-------------|----------------------|
| Lua Scripts | Business logic executed in Redis | HIGH | Code execution, data manipulation |
| Redis Connection | Connection credentials/config | CRITICAL | Full Redis access, data breach |
| Script SHA Digests | Cached script identifiers | MEDIUM | Cache poisoning, performance impact |
| File System Paths | Script storage locations | HIGH | Path traversal, arbitrary file read |
| Application Data | Keys/argv passed to scripts | VARIES | Depends on application usage |

### Data Classification

- **CRITICAL**: Redis connection strings, authentication credentials
- **HIGH**: Lua script source code, file system paths
- **MEDIUM**: Script execution metadata, SHA digests
- **LOW**: Logging information, performance metrics

## Threat Analysis (STRIDE)

### Spoofing Identity

| ID | Threat | Description | Severity | Mitigation Status |
|----|--------|-------------|----------|-------------------|
| S-01 | Malicious Script Masquerading | Attacker replaces legitimate script with malicious one having same SHA | HIGH | ⚠️ PARTIAL - SHA verification exists but file integrity not monitored |
| S-02 | Redis Connection Spoofing | MitM attack on Redis connection | CRITICAL | ⚠️ PARTIAL - Depends on Redis client TLS configuration |

### Tampering

| ID | Threat | Description | Severity | Mitigation Status |
|----|--------|-------------|----------|-------------------|
| T-01 | Lua Script Modification | Unauthorized modification of script files | HIGH | ⚠️ PARTIAL - File system permissions only |
| T-02 | Script Cache Poisoning | Attacker injects malicious script into SHA cache | HIGH | ⚠️ PARTIAL - SHA recalculation on NOSCRIPT but no signature |
| T-03 | ERB Template Injection | Malicious ERB code in script partials | CRITICAL | ❌ NOT ADDRESSED - No input validation on template content |

### Repudiation

| ID | Threat | Description | Severity | Mitigation Status |
|----|--------|-------------|----------|-------------------|
| R-01 | Script Execution Tracking | Lack of audit trail for script execution | MEDIUM | ⚠️ PARTIAL - Debug logging only, not audit-grade |
| R-02 | Script Modification Tracking | No tracking of who modified scripts | MEDIUM | ❌ NOT ADDRESSED - No audit logging |

### Information Disclosure

| ID | Threat | Description | Severity | Mitigation Status |
|----|--------|-------------|----------|-------------------|
| I-01 | Script Source Exposure | Sensitive logic exposed through script files | MEDIUM | ⚠️ PARTIAL - File permissions dependent |
| I-02 | Error Message Leakage | Detailed error messages reveal system info | MEDIUM | ✅ ADDRESSED - LuaError provides controlled error handling |
| I-03 | Path Traversal | Reading arbitrary files via script path manipulation | CRITICAL | ❌ NOT ADDRESSED - No path validation |
| I-04 | Logging Sensitive Data | Redis credentials or data logged | HIGH | ⚠️ PARTIAL - Logging exists but sanitization unclear |

### Denial of Service

| ID | Threat | Description | Severity | Mitigation Status |
|----|--------|-------------|----------|-------------------|
| D-01 | Infinite Loop Scripts | Malicious script with infinite loop | HIGH | ⚠️ PARTIAL - BUSY error handled but no timeout |
| D-02 | Resource Exhaustion | Script consumes excessive memory/CPU | HIGH | ❌ NOT ADDRESSED - No resource limits |
| D-03 | Script Flooding | Rapid script loads exhaust Redis memory | MEDIUM | ❌ NOT ADDRESSED - No rate limiting |
| D-04 | File System Exhaustion | Attempting to load massive files | MEDIUM | ❌ NOT ADDRESSED - No file size limits |

### Elevation of Privilege

| ID | Threat | Description | Severity | Mitigation Status |
|----|--------|-------------|----------|-------------------|
| E-01 | Redis Command Escalation | Script executes privileged Redis commands | HIGH | ❌ NOT ADDRESSED - No command allowlist |
| E-02 | File System Access | Reading sensitive files via template includes | CRITICAL | ❌ NOT ADDRESSED - No path restrictions |
| E-03 | Code Injection via ERB | Arbitrary Ruby code execution through ERB | CRITICAL | ❌ NOT ADDRESSED - ERB.new uses binding without sanitization |

## Attack Vectors and Entry Points

### Primary Attack Vectors

1. **Script Path Injection**
   - **Entry Point**: `config.scripts_path` configuration
   - **Attack**: `../../../etc/passwd` style path traversal
   - **Impact**: Arbitrary file read, potential code execution

2. **Script Name Injection**
   - **Entry Point**: `execute(script_name, ...)` method
   - **Attack**: Pass malicious script name with path traversal
   - **Impact**: Load and execute arbitrary Lua files

3. **ERB Template Injection**
   - **Entry Point**: `include_partial(relative_path)` in Lua scripts
   - **Attack**: Inject ERB code into Lua scripts or partials
   - **Impact**: Arbitrary Ruby code execution on server

4. **Lua Script Injection**
   - **Entry Point**: Lua script file contents
   - **Attack**: Malicious Lua code in script files
   - **Impact**: Arbitrary Redis commands, data access/modification

5. **Keys/Argv Injection**
   - **Entry Point**: `keys:` and `argv:` parameters
   - **Attack**: Inject malicious data through script parameters
   - **Impact**: Depends on script implementation

6. **Dependency Vulnerabilities**
   - **Entry Point**: Third-party gems (redis, concurrent-ruby)
   - **Attack**: Exploit known CVEs in dependencies
   - **Impact**: Varies by vulnerability

### Attack Scenarios

#### Scenario 1: Path Traversal Attack
```ruby
# Attacker controls configuration
configure do |config|
  config.scripts_path = "/app/scripts/../../../../etc"
end

# Executes: reads /etc/passwd.lua if it exists
execute(:passwd, redis_conn)
```

#### Scenario 2: ERB Code Injection
```lua
-- malicious_script.lua with ERB injection
<%= `rm -rf /` %>
redis.call("SET", "key", "value")
```

#### Scenario 3: Lua Command Escalation
```lua
-- Attacker gains access to script directory
-- Creates privileged_script.lua
redis.call("CONFIG", "SET", "dir", "/tmp")
redis.call("CONFIG", "SET", "dbfilename", "evil.so")
-- Further exploitation possible
```

## Existing Security Controls

### Implemented Controls

| Control | Type | Effectiveness | Location |
|---------|------|---------------|----------|
| SHA Digest Verification | Integrity | MEDIUM | `Script#compiled_sha` |
| Script Reload on NOSCRIPT | Availability | HIGH | `Client#handle_noscript` |
| BUSY Error Handling | Availability | MEDIUM | `Client#handle_busy` |
| Structured Error Handling | Security | MEDIUM | `LuaError` class |
| Debug Logging | Monitoring | LOW | Throughout codebase |
| Connection Abstraction | Defense in Depth | MEDIUM | Supports Redis namespacing |

### Control Gaps

- ❌ **No input validation** on script names or paths
- ❌ **No file path sanitization** preventing traversal
- ❌ **No ERB template sandboxing** allowing code injection
- ❌ **No resource limits** on script execution
- ❌ **No audit logging** for security events
- ❌ **No script content validation** before loading
- ❌ **No allowlist** for Redis commands
- ❌ **No rate limiting** on script operations

## Threat Mitigation Recommendations

### Priority 1 (CRITICAL - Address Immediately)

#### 1.1 Path Traversal Prevention
**Threat**: E-02, I-03
**Recommendation**: Implement strict path validation and canonicalization

```ruby
# In Script#initialize
def initialize(name:, root_path:)
  @name = validate_script_name(name)
  @root_path = Pathname.new(root_path).realpath
  @path = build_safe_path(@name)
  # ... rest of initialization
end

private

def validate_script_name(name)
  # Only allow alphanumeric, underscore, hyphen
  unless name.to_s =~ /\A[a-zA-Z0-9_-]+\z/
    raise ArgumentError, "Invalid script name: #{name}"
  end
  name
end

def build_safe_path(name)
  path = @root_path.join("#{name}.lua")
  # Ensure path is within root_path
  unless path.realpath.to_s.start_with?(@root_path.to_s)
    raise SecurityError, "Path traversal detected: #{name}"
  end
  path.to_s
rescue Errno::ENOENT
  # File doesn't exist yet, build expected path
  @root_path.join("#{name}.lua").to_s
end
```

#### 1.2 ERB Template Sandboxing
**Threat**: T-03, E-03
**Recommendation**: Restrict ERB evaluation or eliminate dynamic templating

**Option A: Remove ERB (Recommended)**
```ruby
# Replace ERB with simple string inclusion
def render(pathname)
  content = File.read(pathname)
  # Simple regex-based inclusion without eval
  content.gsub(/--\s*INCLUDE:\s*(\S+)/) do |match|
    include_partial($1)
  end
end
```

**Option B: Sandbox ERB**
```ruby
# Use safe_level or restricted binding
def render(pathname)
  template_code = File.read(pathname)
  # Create restricted binding without dangerous methods
  safe_binding = create_safe_binding
  ERB.new(template_code, safe_level: 4).result(safe_binding)
end
```

#### 1.3 File Size Limits
**Threat**: D-04
**Recommendation**: Enforce maximum file size for scripts

```ruby
MAX_SCRIPT_SIZE = 1_048_576 # 1MB

def render_file
  file_size = File.size(path)
  if file_size > MAX_SCRIPT_SIZE
    raise SecurityError, "Script file too large: #{file_size} bytes"
  end
  Template.new(root_path).render(path)
end
```

### Priority 2 (HIGH - Address Soon)

#### 2.1 Script Content Validation
**Threat**: E-01
**Recommendation**: Validate Lua scripts for dangerous patterns

```ruby
DANGEROUS_PATTERNS = [
  /redis\.call\s*\(\s*["'](?:CONFIG|SHUTDOWN|DEBUG)/i,
  /redis\.call\s*\(\s*["'](?:SCRIPT\s+KILL|SCRIPT\s+FLUSH)/i,
  /redis\.call\s*\(\s*["'](?:EVAL|EVALSHA)/i  # Prevent nested eval
].freeze

def validate_script_content
  DANGEROUS_PATTERNS.each do |pattern|
    if source =~ pattern
      logger.warn("Potentially dangerous command in #{name}: #{pattern}")
    end
  end
end
```

#### 2.2 Audit Logging
**Threat**: R-01, R-02
**Recommendation**: Implement security event logging

```ruby
def execute(script_name, conn, keys: [], argv: [])
  audit_log("script_execution", {
    script: script_name,
    keys_count: keys.length,
    argv_count: argv.length,
    timestamp: Time.now.utc
  })

  result, elapsed = timed do
    scripts.execute(script_name, conn, keys: keys, argv: argv)
  end

  audit_log("script_completed", {
    script: script_name,
    elapsed_ms: elapsed,
    success: true
  })

  result
rescue => ex
  audit_log("script_failed", {
    script: script_name,
    error: ex.class.name,
    message: ex.message
  })
  raise
end
```

#### 2.3 Resource Limits
**Threat**: D-01, D-02
**Recommendation**: Document timeout configuration for users

```markdown
# In README.md - Security Best Practices section

### Redis Timeouts
Configure Redis with appropriate script timeouts:

```redis.conf
lua-time-limit 5000  # 5 seconds max script execution
```

In your application:
```ruby
redis = Redis.new(timeout: 5)
```
```

### Priority 3 (MEDIUM - Address in Next Release)

#### 3.1 Rate Limiting
**Threat**: D-03
**Recommendation**: Add optional rate limiting for script loading

```ruby
# Add to Config
class Config
  attr_accessor :max_scripts_per_minute

  def initialize
    @max_scripts_per_minute = 60 # default
  end
end

# Add rate limiter
class RateLimiter
  def initialize(limit:)
    @limit = limit
    @timestamps = []
  end

  def allow?
    now = Time.now
    @timestamps.reject! { |t| t < now - 60 }
    @timestamps.size < @limit
  end

  def record
    @timestamps << Time.now
  end
end
```

#### 3.2 Dependency Scanning
**Threat**: Attack Vector #6
**Recommendation**: Implement automated dependency scanning

```yaml
# .github/workflows/security.yml
name: Security Scan
on: [push, pull_request]
jobs:
  dependency-scan:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v2
      - uses: ruby/setup-ruby@v1
      - run: gem install bundler-audit
      - run: bundle-audit check --update
```

#### 3.3 TLS Enforcement Documentation
**Threat**: S-02
**Recommendation**: Document secure Redis connection setup

```ruby
# Add to README.md
### Secure Redis Connections

Always use TLS for production Redis connections:

```ruby
redis = Redis.new(
  url: "rediss://redis.example.com:6380",
  ssl_params: {
    verify_mode: OpenSSL::SSL::VERIFY_PEER,
    ca_file: "/path/to/ca.crt"
  }
)
```
```

### Priority 4 (LOW - Future Enhancement)

#### 4.1 Script Signing
**Threat**: S-01, T-01
**Recommendation**: Implement optional script signature verification

#### 4.2 Command Allowlist
**Threat**: E-01
**Recommendation**: Optional configuration to restrict Redis commands

#### 4.3 Security Headers in Documentation
**Threat**: General
**Recommendation**: Add comprehensive security section to README

## Review and Maintenance

### Review Cadence

- **Quarterly Reviews**: Every 3 months, review threat model for new threats
- **Post-Incident**: Update immediately after security incidents
- **Major Releases**: Review before each major version release
- **Dependency Updates**: Review when updating Redis or Ruby versions

### Triggers for Review

1. New attack vectors discovered in Lua or Redis
2. CVEs published affecting dependencies
3. Significant architectural changes
4. New features that handle user input
5. Security incidents or bug reports
6. Changes to Redis command capabilities

### Review Checklist

- [ ] Review all STRIDE categories for new threats
- [ ] Verify existing mitigations are still effective
- [ ] Check for new CVEs in dependencies
- [ ] Update attack scenarios based on real-world incidents
- [ ] Review and update mitigation priorities
- [ ] Ensure documentation reflects current state
- [ ] Update security recommendations

### Stakeholders

- **Security Team**: Primary reviewers
- **Development Team**: Implementation and testing
- **DevOps Team**: Deployment and monitoring
- **Open Source Maintainers**: Community security concerns

## References

### Security Documentation

- [Redis Security Documentation](https://redis.io/topics/security)
- [Lua Security Considerations](https://www.lua.org/manual/5.1/)
- [OWASP Code Injection Prevention](https://cheatsheetseries.owasp.org/cheatsheets/Injection_Prevention_Cheat_Sheet.html)
- [Ruby Security Guide](https://docs.ruby-lang.org/en/master/doc/security_rdoc.html)

### Related Documents

- [CODE_OF_CONDUCT.md](CODE_OF_CONDUCT.md) - Community guidelines
- [CHANGELOG.md](CHANGELOG.md) - Version history
- [README.md](README.md) - Usage documentation
- [docs/THREAT_MODEL_TEMPLATE.md](docs/THREAT_MODEL_TEMPLATE.md) - Template for future threat models

### Incident Response

For security issues, please follow responsible disclosure:
1. **DO NOT** open a public GitHub issue
2. Email security concerns to project maintainers
3. Provide detailed information about the vulnerability
4. Allow reasonable time for patch development
5. Coordinate disclosure timeline

### Version History

| Version | Date | Author | Changes |
|---------|------|--------|---------|
| 1.0 | 2024-01-19 | Security Team | Initial threat model creation |

---

**Document Classification**: INTERNAL
**Review Status**: APPROVED
**Next Review**: 2024-07-19
