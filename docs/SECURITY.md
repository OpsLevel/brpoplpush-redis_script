# Security Documentation

This directory contains security-related documentation for the brpoplpush-redis_script project.

## Available Documents

### [Threat Model](../THREAT_MODEL.md)

The comprehensive threat model for this project, including:
- System architecture and trust boundaries
- Asset classification
- STRIDE threat analysis
- Attack vectors and mitigation recommendations
- Security review cadence

**Location:** `/THREAT_MODEL.md` (repository root)

### [Threat Model Template](THREAT_MODEL_TEMPLATE.md)

A reusable template for creating threat models for other projects or future updates.

**Location:** `/docs/THREAT_MODEL_TEMPLATE.md`

## Security Best Practices

When using brpoplpush-redis_script in your application:

### 1. Script Path Security
- Store Lua scripts in a dedicated, restricted directory
- Use absolute paths for `scripts_path` configuration
- Ensure proper file system permissions (read-only for application user)
- Never allow user input to influence script paths

### 2. Redis Connection Security
- Always use TLS (rediss://) for production Redis connections
- Implement proper authentication (Redis AUTH or ACLs)
- Use network isolation (VPC, private networks)
- Configure Redis with `lua-time-limit` to prevent runaway scripts

### 3. Script Content Review
- Review all Lua scripts for dangerous Redis commands
- Avoid using CONFIG, SHUTDOWN, DEBUG commands in scripts
- Validate all script inputs (keys and argv)
- Use version control for script files
- Implement code review for script changes

### 4. Dependency Management
- Regularly update dependencies (use `bundle update`)
- Monitor for security advisories (GitHub Dependabot, bundle-audit)
- Pin dependency versions in production
- Review dependency changes before updating

### 5. Monitoring and Logging
- Monitor Redis for unusual script execution patterns
- Log script execution with appropriate detail level
- Set up alerts for script errors or timeouts
- Track script performance metrics

## Reporting Security Issues

If you discover a security vulnerability in brpoplpush-redis_script:

### DO NOT
- Open a public GitHub issue
- Disclose the vulnerability publicly before a fix is available
- Share exploit code publicly

### DO
1. Email the project maintainers privately with:
   - Description of the vulnerability
   - Steps to reproduce
   - Potential impact assessment
   - Any suggested fixes (optional)

2. Allow reasonable time for:
   - Vulnerability verification
   - Fix development and testing
   - Security advisory preparation
   - Coordinated disclosure

3. Follow responsible disclosure practices:
   - Provide 90 days for fix development (or agreed timeline)
   - Coordinate disclosure timing with maintainers
   - Credit will be provided for responsible disclosure

## Security Resources

### External References
- [Redis Security Guide](https://redis.io/topics/security)
- [Lua Security Considerations](https://www.lua.org/manual/5.1/)
- [OWASP Injection Prevention](https://cheatsheetseries.owasp.org/cheatsheets/Injection_Prevention_Cheat_Sheet.html)
- [Ruby Security Guide](https://docs.ruby-lang.org/en/master/doc/security_rdoc.html)

### Project Documentation
- [README](../README.md) - General usage documentation
- [Code of Conduct](../CODE_OF_CONDUCT.md) - Community guidelines
- [Changelog](../CHANGELOG.md) - Version history and changes

## Security Maintenance

This documentation is reviewed quarterly and updated as needed. Last review: 2024-01-19

For questions about security documentation, please contact the project maintainers.
