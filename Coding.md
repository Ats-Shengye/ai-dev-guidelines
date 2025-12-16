# Coding Guidelines

## Security Philosophy
**No compromise on security.** Apply all applicable security measures regardless of project scale.
Never lower standards because a project is "small" or "personal". Only skip measures that are physically inapplicable to the project.

Safe, Python-focused development. Ship small, ship fast, don't break things.
Simplicity first with 80/20 rule, but safety measures and minimal tests take priority.

## Secure Defaults

Security is enabled by default. Applied unless explicitly disabled.

- **Default Deny**: Reject operations not explicitly permitted
- **Fail-Safe**: Deny access on error (not permit)
- **Minimal Permissions**: Request only the minimum permissions required for execution
- **Secure by Default**: Operate safely even without configuration

## Basic Principles
- **File Size**: 300 lines/file is a strong guideline. Consider splitting if exceeded
- **Dependency Management**: Prefer standard library. External dependencies only when necessary, use latest stable versions
- **Language**: Code/identifiers/docstrings in English. User-facing output (CLI logs/README) can be in local language
- **Comments**: Write formally, assuming external engineers will read them

## Architecture Principles (AI Coding Era)

Architectural principles for maintaining code quality in the AI coding era.

### Layer Separation
- **Clear knowledge separation**: Limit the knowledge (responsibility) each layer holds to a minimum
- **Unidirectional dependencies**: Only upper layer → lower layer dependencies allowed. Reverse dependencies prohibited
- **Explicit boundaries**: Clearly define interfaces between layers

### Anti-Corruption Layer
- **Isolate external I/O**: Consolidate file operations, network communication, DB connections in dedicated layers
- **Isolate external packages**: Third-party library calls go through anti-corruption layer
- **Localize exception handling**: Limit try/catch, throw to anti-corruption layer (as much as possible)
- **Prefer Result types**: Use Result type pattern instead of exceptions for explicit error propagation

### Exception vs Result Type
- **Result type recommended**: Predictable errors (validation failures, business rule violations, external API failures)
- **Exception acceptable**: Unrecoverable errors (OutOfMemory, system resource exhaustion), design contract violations (program bugs)
- **Anti-corruption layer responsibility**: Catch exceptions from external libraries and convert to Result type

### Side-Effect Management
- **Prefer pure functions**: Implement as side-effect-free functions when possible
- **Explicit side effects**: Functions with side effects should be explicit in naming/documentation
- **Environment-independent layer**: Business logic layer should not depend on OS/runtime

### Logging Layer Design
- **Logger acquisition**: `logger = logging.getLogger(__name__)` in each module (regardless of layer)
- **Handler configuration**: Only at application entry point (top layer)
- **Log output**: Allowed in all layers. Lower layers: detailed, upper layers: summary
- **Log Injection Prevention**: Escape newlines/control characters in log messages
- **Log Integrity**: Append-only log files, HMAC signatures when necessary

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

# Security scanning
pip-audit                                      # Dependency vulnerability scan
safety check                                   # Alternative dependency scanner
```

## Code Style Guidelines
- **Python**: Follow PEP 8, use Black formatter (line-length=120, skip-string-normalization=true)
- **Imports**: Standard library → Third-party → Local imports
- **Variables**: snake_case for variables/functions, UPPER_CASE for constants
- **Docstrings**: Triple quotes for module/class/function documentation
- **Error Handling**: Use specific exception types, handle EACCES/EPERM/EIO appropriately

## Comment Policy

### Audience and Tone
- **Target audience**: External engineers, third-party reviewers
- **Tone**: Formal. No casual expressions or inside jokes
- **Language**: English preferred

### Prohibited (Truly "excessive")
- Explaining obvious code: `i += 1  # Increment i`
- Mere paraphrasing of code: `return result  # Return the result`

### Required (NOT "excessive")
- **Design decisions**: Why this implementation was chosen (including comparison with alternatives)
- **Boundary conditions**: Input constraints, edge case handling
- **Security decisions**: Reasoning behind security choices (can be thorough)
- **Non-obvious logic**: Logic whose intent is not immediately clear
- **External dependencies**: Points that depend on external API/library behavior

### Security Comments Guidelines
**DO**:
- Reason why a security measure was implemented
- Deliberately skipped measures and their justification

**DON'T**:
- Location hints for credentials
- Details of known vulnerabilities or attack methods
- Internal implementation details of security mechanisms

### Test Code Comments (Especially Important)
- **Clarify test intent**: What is being tested, why that case is important
- **Boundary value reasoning**: Why that value was chosen
- **Expected behavior basis**: Why the expected value is correct

### Comment Format Example
```python
# Design Decision: Using explicit loop instead of list comprehension
# for readability when processing nested structures.
# Alternative considered: functools.reduce - rejected due to debugging difficulty.

# Security Note: Input validated at trust boundary (api_handler.py:45).
# This function assumes sanitized input per validation contract.

# Edge Case: Empty list returns default value per specification (REQ-123).
# Zero-length input is valid and expected in batch processing scenarios.
```

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

#### Secure Error Messages
- **Production**: Generic messages only (e.g., "Operation failed")
- **Development**: Detailed stack traces OK (mask sensitive info)
- **User-facing**: Hide internal details with error codes (e.g., "Error E401: Invalid request")

### 5. Concurrency Safety
- **Avoid shared mutable state** whenever possible
- **Use locks consistently** to prevent race conditions
- **Prevent deadlocks** through lock ordering or timeout mechanisms
- **Prefer immutable data structures** in concurrent contexts
- **Document thread safety** guarantees of all components

#### Concurrency Testing
- **Race Condition Tests**: Use ThreadSanitizer, pytest-xdist for parallel execution tests
- **Stress Tests**: Verify behavior under high load (detect deadlocks, leaks)

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

### Language-Specific Adaptations

**TypeScript/JavaScript**:
- Resource Management: try/finally or `using` (TS 5.2+)
- Type Safety: strict mode required, `any` type prohibited
- Error Handling: Result type libraries like neverthrow recommended
- Linting: eslint required, enum prohibited, class generally prohibited

**Go**:
- Resource Management: `defer` statement
- Error Handling: `if err != nil` pattern, error wrapping (fmt.Errorf with %w)
- Type Safety: Minimize interface{}, type assertion with ok-check

**Rust**:
- Error Handling: Result<T, E>, ? operator
- Resource Management: RAII automatically applied

### Module Docstring Template
```python
"""
Location   : path/to/module.py  # Update when moved
Purpose    : What this module does (1-2 lines)
Why        : Why this module exists / design intent (1-2 lines)
Related    : path/other_module.py, path/another.py
"""
```

## Testing Standards (AI Coding Era)

### Testing Philosophy
AI-generated code has a probability of containing "poison". Integration test verification is paramount.

### Test Hierarchy (Priority Order)
1. **Integration Tests** (Most Important): Test at API/function level with actual I/O
2. **Unit Tests** (Limited): Only for complex logic and boundary conditions
3. **E2E Tests** (As Needed): Verification of entire user flows

### Integration Test Requirements
- **Real DB/filesystem**: Test with environments closer to production than mocks
- **Error case coverage**: All patterns, not just happy paths
- **Coverage target**: 100% recommended (exceptions: view layer, barrel exports)

### Security Testing Requirements
- **Input Validation Tests**: Test all patterns for boundary values, invalid values, encoding bypasses
- **Authorization Tests**: Confirm permission checks work at all endpoints
- **Error Handling Tests**: Verify error messages don't leak sensitive information
- **Resource Exhaustion Tests**: DoS resistance (high volume requests, large files)

### Test File Organization
- **Location**: `tests/` directory mirroring source structure
  - `src/api/user.py` → `tests/api/test_user.py`
- **File naming**: `test_<module_name>.py`
- **Integration tests**: `tests/integration/`
- **Function naming**: `test_<target>_<condition>_<expected_result>`

### Test Code Quality
- **Thorough comments**: Clearly state test intent, boundary value reasoning, expected behavior basis
- **External engineer readability**: Standard is "readable by engineers outside the project"

### Destructive Operations
- NEVER execute destructive commands without explicit explanation and user confirmation

## Refactoring Rules

### Separation Principle
When refactoring, separate test changes from implementation changes.

```
Phase 1: Test changes only (don't touch implementation)
  ↓ Confirm tests pass
Phase 2: Implementation changes only (don't touch tests)
  ↓ Confirm tests pass
Phase 3: Repeat as necessary
```

### Rationale
- AI tends to break consistency during simultaneous changes
- Separation makes problem isolation easier
- Each phase guarantees tests pass

### Cases Where Simultaneous Changes Are Acceptable
- **Interface changes**: Test changes required when changing function signatures
- **Type definition changes**: Caller modifications needed with type changes
- **Breaking changes**: When backward compatibility is impossible

**Condition**: Document reason in commit message, User approval required

## Dependency Management
- **Standard Library First**: Prefer standard library over external dependencies
- **Security Validation**: Run security checks on all dependencies before use
- **Version Strategy**: Use latest stable versions, avoid strict pinning unless necessary
- **Dependency Files**: Implement `requirements.txt` or `pyproject.toml`
- **Update Verification**: Check for tool/library updates before project execution
- **Warnings as Errors**: Configure linters to fail on warnings

## Secrets Management
- **Storage Location**: Environment variables or `~/.config/<app>/` (mode 600) - encryption recommended
- **Loading Pattern**: Load with permission checks
- **Never**: commit to git, log, include in error messages
- **Rotation**: Establish regular rotation schedule

## Git & Version Control
- **Branch Strategy**: Work directly on `main` branch (personal development)
- **Commit Messages**: Concise description (e.g., "fix(auth): add input validation")
- **Commit Frequency**: Commit when features are complete and functional
- **Privacy Protection**: Ensure no personal information in commit history
- **.gitignore**: Exclude personal data, temporary files, cache, secrets

## Performance Guidelines
- **Development Priority**: Functionality first, optimization as needed
- **Resource Awareness**: Consider memory limitations
- **Optimization Approach**: Profile before optimizing

## Development Workflow
- **Planning**: Use task tracking for multi-step tasks (3+ steps)
- **Error Handling**: Root cause analysis required for all failures
- **Quality Gates**: Run lint/typecheck before marking tasks complete
- **Validation**: Test functionality before delivery

## Pre-Submission Checklist

### Automated Checks (pre-commit hook recommended)
- [ ] Black formatting
- [ ] `mypy --strict .`
- [ ] `ruff check .`
- [ ] `bandit -r . -ll`
- [ ] `pip-audit` (dependency scan)
- [ ] `python3 -m py_compile` (all modules)

### Manual Review Required
- [ ] Module docstring contains 4 required elements (Location, Purpose, Why, Related)
- [ ] Integration tests cover API/function-level behavior with real I/O
- [ ] Security tests cover input validation, authorization, error handling
- [ ] Comments are formal, external-engineer-readable, explain design intent
- [ ] Layer boundaries respected (no reverse dependencies)
- [ ] All external inputs validated at trust boundaries
- [ ] No hardcoded secrets, credentials, or personal information
- [ ] Resource management uses context managers consistently
- [ ] Error handling catches specific exceptions with proper logging
- [ ] Concurrency safety verified for shared mutable state
- [ ] Security-sensitive operations logged for audit trail
- [ ] `.gitignore` excludes personal data, temporary files, and cache
- [ ] `sudo` and destructive operations documented and approved
- [ ] Code generation tool output thoroughly reviewed if used

---

## Known Limitations (Future Improvements)

This guideline has the following known limitations. Be aware when using.

### 1. DAST (Dynamic Security Scanning) Not Automated
- **Current**: SAST (bandit, ruff) and dependency scanning (pip-audit) are covered, but no mention of dynamic scanners like OWASP ZAP
- **Risk**: May miss untested attack patterns (especially Web/API)
- **Interim**: Thorough integration testing with real I/O
- **Future**: Add DAST criteria for Web/API projects

### 2. Python-Specific Concurrency Testing Constraints
- **Current**: ThreadSanitizer, pytest-xdist recommended, but Python GIL constraints make true parallel execution testing difficult in some cases
- **Interim**: Consider using multiprocessing, strengthen manual review for suspected race conditions

### 3. Secrets Management Encryption Details Lacking
- **Current**: "Encryption recommended" stated but no selection criteria for specific methods (age/gpg/python-keyring)
- **Interim**: Decide based on project characteristics in consultation with User

### 4. pre-commit hook Implementation Guide Missing
- **Current**: "pre-commit hook recommended" but no specific configuration examples
- **Interim**: Manual execution achieves equivalent checks

---
Keep implementation thin, reasoning brief, and ship without breaking.
