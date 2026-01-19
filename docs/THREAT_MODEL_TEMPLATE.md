# Threat Model Template

**Version:** [Version Number]
**Last Updated:** [YYYY-MM-DD]
**Document Status:** [Draft/Active/Archived]
**Next Review Date:** [YYYY-MM-DD]

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

### Purpose
[Brief description of what this threat model covers and why it exists]

### Scope
[Define what is included and excluded from this threat model]

### Key Security Concerns
[List 3-5 primary security concerns for this system]

- **Concern 1:** [Description]
- **Concern 2:** [Description]
- **Concern 3:** [Description]

## System Architecture Overview

### Components

[List and describe the major components of the system]

1. **Component 1**: [Description]
2. **Component 2**: [Description]
3. **Component 3**: [Description]

### Data Flow

```
[Describe or diagram the data flow through the system]

User/Client
    ↓
Component A
    ↓
Component B
    ↓
External Service/Database
```

### Trust Boundaries

[Identify where trust boundaries exist in the system]

1. **Boundary 1**: [Description - e.g., "Application → Database"]
2. **Boundary 2**: [Description - e.g., "User Input → Application"]
3. **Boundary 3**: [Description - e.g., "Application → External API"]

### External Dependencies

[List external systems, libraries, or services the system depends on]

- **Dependency 1**: [Name and version]
- **Dependency 2**: [Name and version]

## Assets and Data Classification

### Critical Assets

| Asset | Description | Sensitivity | Impact if Compromised |
|-------|-------------|-------------|----------------------|
| [Asset Name] | [What it is] | [LOW/MEDIUM/HIGH/CRITICAL] | [Impact description] |
| [Asset Name] | [What it is] | [LOW/MEDIUM/HIGH/CRITICAL] | [Impact description] |

### Data Classification

- **CRITICAL**: [Examples - credentials, encryption keys, PII]
- **HIGH**: [Examples - business logic, proprietary data]
- **MEDIUM**: [Examples - configuration, metadata]
- **LOW**: [Examples - public documentation, logs]

## Threat Analysis (STRIDE)

### Spoofing Identity

| ID | Threat | Description | Severity | Mitigation Status |
|----|--------|-------------|----------|-------------------|
| S-01 | [Threat name] | [Description] | [LOW/MEDIUM/HIGH/CRITICAL] | [✅/⚠️/❌] [Status] |
| S-02 | [Threat name] | [Description] | [LOW/MEDIUM/HIGH/CRITICAL] | [✅/⚠️/❌] [Status] |

### Tampering

| ID | Threat | Description | Severity | Mitigation Status |
|----|--------|-------------|----------|-------------------|
| T-01 | [Threat name] | [Description] | [LOW/MEDIUM/HIGH/CRITICAL] | [✅/⚠️/❌] [Status] |
| T-02 | [Threat name] | [Description] | [LOW/MEDIUM/HIGH/CRITICAL] | [✅/⚠️/❌] [Status] |

### Repudiation

| ID | Threat | Description | Severity | Mitigation Status |
|----|--------|-------------|----------|-------------------|
| R-01 | [Threat name] | [Description] | [LOW/MEDIUM/HIGH/CRITICAL] | [✅/⚠️/❌] [Status] |
| R-02 | [Threat name] | [Description] | [LOW/MEDIUM/HIGH/CRITICAL] | [✅/⚠️/❌] [Status] |

### Information Disclosure

| ID | Threat | Description | Severity | Mitigation Status |
|----|--------|-------------|----------|-------------------|
| I-01 | [Threat name] | [Description] | [LOW/MEDIUM/HIGH/CRITICAL] | [✅/⚠️/❌] [Status] |
| I-02 | [Threat name] | [Description] | [LOW/MEDIUM/HIGH/CRITICAL] | [✅/⚠️/❌] [Status] |

### Denial of Service

| ID | Threat | Description | Severity | Mitigation Status |
|----|--------|-------------|----------|-------------------|
| D-01 | [Threat name] | [Description] | [LOW/MEDIUM/HIGH/CRITICAL] | [✅/⚠️/❌] [Status] |
| D-02 | [Threat name] | [Description] | [LOW/MEDIUM/HIGH/CRITICAL] | [✅/⚠️/❌] [Status] |

### Elevation of Privilege

| ID | Threat | Description | Severity | Mitigation Status |
|----|--------|-------------|----------|-------------------|
| E-01 | [Threat name] | [Description] | [LOW/MEDIUM/HIGH/CRITICAL] | [✅/⚠️/❌] [Status] |
| E-02 | [Threat name] | [Description] | [LOW/MEDIUM/HIGH/CRITICAL] | [✅/⚠️/❌] [Status] |

## Attack Vectors and Entry Points

### Primary Attack Vectors

1. **Attack Vector 1**
   - **Entry Point**: [Where the attack enters the system]
   - **Attack**: [Description of the attack method]
   - **Impact**: [What happens if successful]

2. **Attack Vector 2**
   - **Entry Point**: [Where the attack enters the system]
   - **Attack**: [Description of the attack method]
   - **Impact**: [What happens if successful]

### Attack Scenarios

#### Scenario 1: [Scenario Name]
```
[Describe a realistic attack scenario with code examples or step-by-step description]

Example:
1. Attacker performs action X
2. System responds with Y
3. Attacker exploits Z
4. Impact: ...
```

#### Scenario 2: [Scenario Name]
```
[Another attack scenario]
```

## Existing Security Controls

### Implemented Controls

| Control | Type | Effectiveness | Location |
|---------|------|---------------|----------|
| [Control Name] | [Preventive/Detective/Corrective] | [LOW/MEDIUM/HIGH] | [Where implemented] |
| [Control Name] | [Preventive/Detective/Corrective] | [LOW/MEDIUM/HIGH] | [Where implemented] |

### Control Gaps

- ❌ **[Gap Description]**: [What's missing]
- ❌ **[Gap Description]**: [What's missing]
- ⚠️ **[Partial Gap]**: [What needs improvement]

## Threat Mitigation Recommendations

### Priority 1 (CRITICAL - Address Immediately)

#### 1.1 [Mitigation Name]
**Threat**: [Threat IDs this addresses]
**Recommendation**: [Description of what to do]

```[language]
[Code example or implementation guidance]
```

**Effort**: [LOW/MEDIUM/HIGH]
**Impact**: [Description of security improvement]

### Priority 2 (HIGH - Address Soon)

#### 2.1 [Mitigation Name]
**Threat**: [Threat IDs]
**Recommendation**: [Description]

### Priority 3 (MEDIUM - Address in Next Release)

#### 3.1 [Mitigation Name]
**Threat**: [Threat IDs]
**Recommendation**: [Description]

### Priority 4 (LOW - Future Enhancement)

#### 4.1 [Mitigation Name]
**Threat**: [Threat IDs]
**Recommendation**: [Description]

## Review and Maintenance

### Review Cadence

- **[Frequency] Reviews**: [Description - e.g., "Quarterly, every 3 months"]
- **Post-Incident**: Update immediately after security incidents
- **Major Releases**: Review before each major version release
- **[Other trigger]**: [Description]

### Triggers for Review

1. [Trigger 1 - e.g., "New attack vectors discovered"]
2. [Trigger 2 - e.g., "CVEs in dependencies"]
3. [Trigger 3 - e.g., "Architectural changes"]
4. [Trigger 4 - e.g., "Security incidents"]

### Review Checklist

- [ ] Review all STRIDE categories for new threats
- [ ] Verify existing mitigations are still effective
- [ ] Check for new CVEs in dependencies
- [ ] Update attack scenarios based on real-world incidents
- [ ] Review and update mitigation priorities
- [ ] Ensure documentation reflects current state
- [ ] Update security recommendations

### Stakeholders

- **[Role/Team 1]**: [Responsibility]
- **[Role/Team 2]**: [Responsibility]
- **[Role/Team 3]**: [Responsibility]

## References

### Security Documentation

- [Link to relevant security standard or guideline]
- [Link to industry best practices]
- [Link to security policies]

### Related Documents

- [Link to related internal documentation]
- [Link to runbooks]
- [Link to incident response procedures]

### Incident Response

[Describe how to report security issues]

For security issues, please:
1. [Step 1 - e.g., "DO NOT open a public issue"]
2. [Step 2 - e.g., "Email security@example.com"]
3. [Step 3 - e.g., "Provide detailed information"]
4. [Step 4 - e.g., "Allow time for patch development"]

### Version History

| Version | Date | Author | Changes |
|---------|------|--------|---------|
| [X.X] | [YYYY-MM-DD] | [Name] | [Description of changes] |

---

## How to Use This Template

1. **Copy this template** to create a new threat model document
2. **Fill in all sections** with information specific to your system
3. **Customize STRIDE categories** - add or remove threats as needed
4. **Prioritize mitigations** based on your risk tolerance and resources
5. **Review regularly** according to your defined cadence
6. **Keep it updated** as the system evolves

### Tips for Effective Threat Modeling

- **Be specific**: Vague threats lead to ineffective mitigations
- **Consider context**: What's high risk for one system may be low for another
- **Involve stakeholders**: Get input from developers, ops, and security teams
- **Think like an attacker**: Consider unconventional attack paths
- **Document assumptions**: Note what you assume is secure and why
- **Be practical**: Focus on realistic threats, not theoretical extremes
- **Update regularly**: Threat landscapes change constantly

### Severity Definitions

- **CRITICAL**: Immediate risk of complete system compromise or data breach
- **HIGH**: Significant risk of unauthorized access or data exposure
- **MEDIUM**: Moderate risk with potential for escalation
- **LOW**: Minimal risk with limited impact

### Status Indicators

- ✅ **ADDRESSED**: Mitigation fully implemented and tested
- ⚠️ **PARTIAL**: Some mitigations in place but gaps remain
- ❌ **NOT ADDRESSED**: No mitigation currently implemented

---

**Document Classification**: [INTERNAL/CONFIDENTIAL/PUBLIC]
**Review Status**: [DRAFT/APPROVED/ARCHIVED]
**Next Review**: [YYYY-MM-DD]
