# Coding Guidelines

## Security Philosophy
**No compromise on security.** Apply all applicable security measures regardless of project scale.
Never lower standards because a project is "small" or "personal". Only skip measures that are physically inapplicable to the project.

Safe, Python-focused development. Ship small, ship fast, don't break things.
Simplicity first with 80/20 rule, but safety measures and minimal tests take priority.

## Basic Principles
- **File Size**: 300 lines/file is a strong guideline. Consider splitting if exceeded
- **Dependency Management**: Prefer standard library. External dependencies only when necessary, use latest stable versions
- **Language**: Code/identifiers/docstrings in English. User-facing output (CLI logs/README) can be in local language
- **Comments**: Minimal. Only add brief context for complex sections

## Development Flow
- **Branching**: Direct `main` commits OK for personal development. Keep commits small and functional
- **GitHub Push**: Prohibited without explicit approval
- **DB Changes**: Human-only (AI suggests, human executes)

## Project Setup

### Python Version Verification
**Rationale**: Older Python versions may lack security patches for known CVEs.

**Before starting a new project**:
```bash
# Verify Python version
python --version
python3 --version

# Check for known vulnerabilities
# 1. Visit https://www.python.org/downloads/ for security releases
# 2. Search CVE databases: https://nvd.nist.gov/vuln/search
# 3. Review Python security advisories: https://www.python.org/dev/security/

# Automated check:
pip-audit  # Checks installed packages and Python itself for known CVEs

# Recommended: Use latest stable Python 3.x (currently 3.12+ for tarfile filters)
```

**If older Python version is required**:
1. Document the requirement with justification
2. Evaluate CVE impact on project-specific code paths
3. Implement mitigations if upgrading is not feasible
4. Plan for future upgrade

## Build/Test Commands
```bash
# Quick syntax checks
python3 -m py_compile path/to/module.py
python3 - <<'PY'
import ast
ast.parse(open('path/to/module.py').read())
print('AST OK')
PY

# Static analysis (required before commit)
mypy --strict .                                # Type checking
ruff check . --config pyproject.toml           # Fast linting
bandit -r . -ll                                # Security linting

# Configure in pyproject.toml:
# [tool.mypy]: strict = true, warn_return_any = true
# [tool.ruff]: select = ["E", "F", "W", "I", "N", "S"], ignore = []
# [tool.bandit]: exclude_dirs = ["tests/", "venv/"]
```

## Code Style Guidelines
- **Python**: Follow PEP 8, use Black formatter (line-length=120, skip-string-normalization=true)
- **Imports**: Standard library → Third-party → Local imports
- **Variables**: snake_case for variables/functions, UPPER_CASE for constants
- **Docstrings**: Triple quotes for module/class/function documentation
- **Error Handling**: Use specific exception types, handle EACCES/EPERM/EIO appropriately
- **Comments**: Minimal inline comments, focus on docstrings and clear variable names
- **Exception Documentation**: Document edge cases, boundary conditions, and security decisions

## Universal Coding Principles

### 1. Trust Boundary Management
- **Validate all external inputs** at system boundaries
- **Sanitize data** before use in commands, queries, or file operations
- **Use type systems** to distinguish trusted vs untrusted data
- **Normalize input** before validation to prevent bypass attacks

### 2. Defensive Programming
- **Explicit precondition checks** at function entry points
- **Maintain invariants** throughout object lifecycle
- **Fail-safe design**: system should fail to a secure state
- **Validate assumptions** rather than relying on implicit behavior

### 3. Strict Resource Management
- **Always use context managers** (with statements) for resources
- **Implement RAII patterns** for automatic cleanup
- **Set resource limits** to prevent exhaustion attacks
- **Monitor for leaks** in long-running processes

### 4. Comprehensive Error Handling
- **Catch specific exceptions** only, never use bare except
- **Log errors with context** but never expose sensitive data
- **Implement retry logic** with exponential backoff for transient failures
- **Design for graceful degradation** in partial failure scenarios
- **Propagate errors explicitly** rather than silencing them

### 5. Concurrency Safety
- **Avoid shared mutable state** whenever possible
- **Use locks consistently** to prevent race conditions
- **Prevent deadlocks** through lock ordering or timeout mechanisms
- **Prefer immutable data structures** in concurrent contexts
- **Document thread safety** guarantees of all components

### 6. Time and State Management
- **Always use timezone-aware datetime** objects
- **Use monotonic clocks** for intervals and timeouts
- **Avoid time-of-check-time-of-use** (TOCTOU) vulnerabilities
- **Handle clock skew** in distributed systems

### 7. Dependency Verification
- **Pin dependency versions** with hash verification
- **Verify package signatures** when available
- **Monitor for supply chain attacks** (typosquatting, package takeover)
- **Audit dependencies regularly** for known vulnerabilities
- **Minimize dependency count** to reduce attack surface

### 8. Code Generation Tool Usage
- **Treat generated code as untrusted**: always review before use
- **Verify context understanding**: check if assumptions match reality
- **Maintain architectural control**: humans decide structure, AI assists implementation
- **Test generated code thoroughly**: do not assume correctness
- **Document generation prompts**: enable reproducibility and debugging

### 9. Logging and Observability
- **Configure logging at module level**, not within functions
- **Use lazy string formatting** (%s) in log messages for performance
- **Never log sensitive data**: passwords, tokens, PII
- **Include correlation IDs** for request tracing
- **Log security-relevant events** for audit trails

### 10. Type Safety and Validation
- **Use type hints** and static type checkers
- **Validate data at boundaries**, not just at storage
- **Avoid mutable default arguments** in function signatures
- **Use float comparison with tolerance** (math.isclose) not equality
- **Prefer explicit type conversions** over implicit coercion

### Module Docstring Template
```python
"""
Location   : path/to/module.py  # Update when moved
Purpose    : What this module does (1-2 lines)
Why        : Why this module exists / design intent (1-2 lines)
Related    : path/other_module.py, path/another.py
"""
```

## Testing Standards
- **Destructive Operations**: NEVER execute without explicit explanation and user confirmation
- **Basic Testing**: Minimum viable functionality tests + security validation required
- **Framework Selection**: Use Python `unittest` by default, consider `pytest` when necessary
- **Test Timing**: Execute at sprint/milestone completion and before final delivery
- **Coverage Priority**: Focus on critical functionality and security-related code

## Dependency Management
- **Standard Library First**: Prefer Python standard library over external dependencies
- **Security Validation**: Run security checks on all dependencies before use
- **Version Strategy**: Use latest stable versions, avoid strict pinning unless necessary
- **Dependency Files**: Implement `requirements.txt` or `pyproject.toml` when external libraries needed
- **Update Verification**: Check for tool/library updates before project execution
- **Configuration Files**: Use `pyproject.toml` for tool configuration, commit to repository root
- **Warnings as Errors**: Configure linters to fail on warnings in CI/pre-commit checks

## Git & Version Control
- **Branch Strategy**: Work directly on `main` branch (personal development)
- **Commit Messages**: Concise description (e.g., "fix(auth): add input validation")
- **Commit Frequency**: Commit when features are complete and functional
- **Privacy Protection**: Ensure no personal information in commit history or files
- **GitHub Preparation**: All code should be ready for public publication
- **.gitignore**: Exclude personal data, temporary files, cache files, sensitive configurations

## Performance Guidelines
- **Development Priority**: Functionality first, optimization as needed
- **Resource Awareness**: Consider memory limitations, avoid memory-intensive operations
- **Optimization Approach**: Profile before optimizing, focus on user-visible performance impacts

## Development Workflow
- **Planning**: Use task tracking for multi-step tasks (3+ steps)
- **Error Handling**: Root cause analysis required for all failures
- **Quality Gates**: Run lint/typecheck before marking tasks complete
- **Documentation**: Code should be self-documenting, avoid excessive comments
- **Validation**: Test functionality before delivery, verify security measures

## Pre-Submission Checklist
- [ ] Code formatted with Black (120 columns)
- [ ] Module docstring contains all 4 required elements (Location, Purpose, Why, Related)
- [ ] `python3 -m py_compile` and `ast.parse` pass without errors
- [ ] Static analysis passed: `mypy --strict .`, `ruff check .`, `bandit -r . -ll`
- [ ] Tool configuration in `pyproject.toml` with warnings-as-errors enabled
- [ ] Unit tests added/updated for critical changes
- [ ] All external inputs validated at trust boundaries
- [ ] No hardcoded secrets, credentials, or personal information
- [ ] Resource management uses context managers consistently
- [ ] Error handling catches specific exceptions with proper logging
- [ ] Concurrency safety verified for shared mutable state
- [ ] Dependencies pinned with hash verification where possible
- [ ] Security-sensitive operations logged for audit trail
- [ ] Edge cases and security decisions documented where non-obvious
- [ ] `.gitignore` excludes personal data, temporary files, and cache
- [ ] `sudo` and destructive operations documented and approved
- [ ] Code generation tool output thoroughly reviewed if used

---
Keep implementation thin, reasoning brief, and ship without breaking.
