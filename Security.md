# Security Standards

## Security Philosophy
**Maximum security by default.** Apply all security measures that are technically applicable, regardless of project scale or purpose.

### Core Principles
- **No compromise on security**: Personal projects, PoCs, and prototypes follow the same standards
- **Apply all applicable measures**: Never skip because it's "tedious" or "small-scale"
- **When uncertain, apply**: If in doubt, implement the security measure

### Essential Security Measures
- Input validation at all boundaries
- No hardcoded secrets/credentials
- Specific exception handling (no bare except)
- Context managers for resource management
- Type hints and static type checking
- Security scanning (bandit, ruff)
- Proper file permissions (600 for config, 700 for private dirs, 755 for executables)
- Error messages without sensitive data
- Audit logging for security-relevant events

## Fundamental Security Principles

### 1. Principle of Least Privilege
- **Run with minimum necessary permissions** for each process and component
- **Avoid sudo unless absolutely required**: document justification and seek approval
- **Drop privileges immediately** after privileged operations complete
- **Use capability-based security** where available
- **Separate privileged and unprivileged code paths** clearly

### 2. Defense in Depth
- **Implement multiple security layers**: no single point of failure
- **Assume breach mentality**: design for containment and detection
- **Validate at every boundary**: don't trust internal components implicitly
- **Redundant verification mechanisms** for critical operations

### 3. Secure by Default
- **Default deny**: explicitly allow only necessary access
- **Safe initial configuration**: require opt-out of security features, not opt-in
- **Fail securely**: on error, deny access rather than granting it
- **Minimal attack surface**: disable unnecessary features and endpoints
- **Secure templates**: provide safe starting points for common patterns

### 4. Privacy and Data Protection
- **NEVER include personal information** in code, comments, or logs
- **Assume public disclosure**: all code should be safe for publication
- **Mask sensitive data** in error messages and logs
- **Encrypt data at rest and in transit** for sensitive information
- **Data minimization**: collect and retain only necessary information
- **Proper file permissions**: 600 for config files, 700 for private directories

### 5. Cryptographic Standards
- **Use established algorithms only**: AES-256, ChaCha20, RSA-4096, Ed25519
- **Never implement custom crypto**: use well-tested libraries
- **Secure key management**: keys in environment variables or key stores, never in code
- **Cryptographically secure random**: use `secrets` module, not `random`
- **Hash passwords properly**: use bcrypt, argon2, or scrypt with salt
- **Validate certificates**: never disable TLS verification

### 6. Input Validation and Sanitization
- **Whitelist validation**: define allowed inputs, reject everything else
- **Normalize before validation**: prevent bypass through encoding tricks
- **Validate at multiple layers**: syntax → semantics → business logic
- **Context-specific encoding**: SQL escaping ≠ HTML escaping ≠ shell escaping
- **Reject, don't sanitize**: removal can be bypassed, rejection cannot

### 7. Attack Surface Reduction
- **Minimize exposed endpoints**: only expose what's necessary
- **Remove unused dependencies**: each dependency is a potential vulnerability
- **Disable debug features in production**: no debug endpoints, verbose errors, or test accounts
- **Remove sensitive comments**: no TODOs with security implications

### 8. Auditability and Monitoring
- **Log all security events**: authentication, authorization, privilege changes
- **Tamper-proof logging**: append-only, integrity-protected logs
- **Include correlation IDs** for request tracing across services
- **Alert on anomalies**: failed auth attempts, privilege escalations, unusual patterns
- **NEVER log sensitive data**: passwords, tokens, credit cards, PII
- **Retain logs appropriately**: balance security needs with privacy requirements

## Code-Level Security Practices

### 9. Injection Attack Prevention

#### SQL Injection
- Always use parameterized queries, never string concatenation

#### Command Injection
- Never use shell=True with user input; use list arguments

#### Path Traversal
- Validate paths, use Path().name to strip directory components
- Reject absolute paths and parent directory references (..)

#### Archive Extraction Safety
**Critical**: Extraction filters alone are insufficient.

**Required Multi-Layer Defense**:

1. **Use Extraction Filter** (Python 3.12+):
   ```python
   import tarfile
   with tarfile.open("archive.tar.gz") as tar:
       tar.extractall(path="/safe/dest", filter='data')
   ```

2. **Pre-Extraction Validation**:
   - Reject absolute paths (starts with `/`)
   - Reject parent directory references (contains `..`)
   - Validate symlink targets
   - Whitelist filename characters
   - Enforce file count and size limits

3. **Isolation and Resource Limits**:
   - Extract to empty temporary directory
   - Set OS-level resource limits
   - Monitor extraction progress

**Manual Validation Example (Python <3.12)**:
```python
import tarfile
from pathlib import Path

def safe_extract(tar_path: str, dest_dir: str) -> None:
    dest = Path(dest_dir).resolve()

    with tarfile.open(tar_path) as tar:
        for member in tar.getmembers():
            if member.name.startswith('/'):
                raise ValueError(f"Absolute path rejected: {member.name}")
            if '..' in Path(member.name).parts:
                raise ValueError(f"Parent reference rejected: {member.name}")
            member_path = (dest / member.name).resolve()
            if not member_path.is_relative_to(dest):
                raise ValueError(f"Path outside destination: {member.name}")
        tar.extractall(path=dest_dir)
```

#### XML/XXE
- Disable external entity processing in XML parsers

#### Template Injection
- Use auto-escaping template engines

### 10. Authentication and Authorization
- **Strong authentication**: multi-factor where possible, secure password policies
- **Session management**: secure tokens, proper timeout, regenerate on privilege change
- **Authorization checks**: verify on every request, not just first access
- **NEVER hardcode credentials**: use environment variables or secret management
- **Implement account lockout** after failed authentication attempts

### 11. Secure Communication
- **HTTPS/TLS everywhere**: no plaintext protocols for sensitive data
- **Certificate validation**: never disable, pin certificates for critical connections
- **Forward secrecy**: use ephemeral key exchange (ECDHE)
- **Secure WebSocket**: use wss:// not ws://
- **API authentication**: OAuth2, JWT with proper validation, API keys over HTTPS only

### 12. Supply Chain Security
- **Dependency verification**: check signatures, use lock files with hashes
- **Vulnerability scanning**: pre-commit and weekly scans
- **Scanning tools**: bandit (SAST), pip-audit (CVE database), ruff (general linting)
- **Verify package integrity**: compare checksums before installation
- **Avoid typosquatting**: carefully check package names before installing
- **Review dependency updates**: don't blindly auto-merge security patches
- **Minimize transitive dependencies**: fewer dependencies = smaller attack surface

### 13. Secure Development Lifecycle
- **Threat modeling**: identify attack vectors before implementation
- **Security testing**: SAST, DAST, dependency scanning
- **Code review with security focus**: dedicated security review for critical code
- **Secrets scanning**: automated detection of committed credentials
- **Incident response procedures**:
  1. Identify: detect anomaly through logs or alerts
  2. Contain: isolate affected component, revoke compromised credentials
  3. Analyze: determine root cause and scope
  4. Remediate: patch vulnerability, restore from clean backup if needed
  5. Document: record timeline, impact, and preventive measures
  6. Review: update security measures to prevent recurrence

### 14. Deployment and Operations Security
- **Environment separation**: dev/staging/prod with different credentials
- **Configuration management**: secure storage, version control (encrypted), audit access
- **Update strategy**: timely patching with rollback capability
- **Backup security**: encrypted backups, tested restoration, offsite storage
- **Monitoring and alerting**: intrusion detection, anomaly detection

## Security Review Checklist
- [ ] All external inputs validated with whitelist approach
- [ ] No hardcoded credentials, API keys, or secrets in code
- [ ] Parameterized queries used for all database operations
- [ ] Subprocess calls use list arguments, never shell=True with user input
- [ ] File operations validate and restrict paths to safe directories
- [ ] Archive extraction uses filter='data' AND multi-layer validation
- [ ] Cryptographic operations use established libraries and algorithms
- [ ] Error messages reveal no sensitive information
- [ ] All privileged operations require explicit authorization checks
- [ ] Sensitive data encrypted at rest and in transit
- [ ] Security events logged without exposing sensitive data
- [ ] Dependencies verified and scanned for known vulnerabilities
- [ ] No debug features or verbose errors in production
- [ ] sudo usage documented, justified, and approved
- [ ] Automated security scanning passed
- [ ] Code generation tool output thoroughly reviewed for security issues
