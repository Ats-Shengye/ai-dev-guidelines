# AI-Assisted Development Workflow

## Agent Team Structure

### Claude (Manager/Architect)
- **Role**: Requirements definition, architecture decisions, final reporting
- **Reference Documents**: Coding.md + Security.md
- **Responsibilities**:
  - Determine security measure scope based on project characteristics
  - Make decisions based on Security.md guidelines
- **Deliverables**: Review result summaries, next action instructions

### Coder (Implementation)
- **Role**: Guideline-compliant implementation
- **Primary Reference**: Coding.md
- **Secondary Reference**: Security.md (security awareness)
- **Focus Areas**:
  - Universal Coding Principles (10 items)
  - Error handling, resource management
  - Type safety, input validation
- **Deliverables**: Implementation code + Pre-Submission Checklist self-check results

### Security Reviewer (Verification)
- **Role**: Security verification
- **Primary Reference**: Security.md
- **Verification Scope**:
  - Essential security measures (all projects)
  - Security Review Checklist (all items)
- **Deliverables**: Vulnerability list (Critical/High/Medium classification) + Remediation requests

## Workflow Phases

**Phase 1: Requirements Definition (Claude)**
1. Gather requirements from User
2. Analyze project characteristics
3. Determine applicable security measures
4. Create implementation instructions (requirements + security measures list)

**Phase 2: Implementation (Coder)**
1. Implement following Coding.md
2. Self-check with Pre-Submission Checklist
3. Report implementation completion (code + check results)

**Phase 3: Review (Security Reviewer)**
1. Verify all Security Review Checklist items
2. Score vulnerabilities (severity classification)
3. Generate remediation instructions

**Phase 4: Report (Claude)**
1. Integrate review results
2. Prioritize issues
3. Report to User (one loop complete)

**Phase 5: Fix or Approve (User Decision)**
- Fixes needed → Return to Phase 2
- Approved → Project complete

## Reference Documents
- **Coding Standards**: Coding.md
- **Security Standards**: Security.md

## Agent Commands

### `/coder` - Implementation Agent
**Purpose**: Implement following Coding.md guidelines
**Example**:
```
/coder Implement a task management API using Firebase and Express.
```
**Output**:
- Implementation code
- Pre-Submission Checklist self-check results
- Security considerations

### `/security-review` - Security Review Agent
**Purpose**: Verify code based on Security.md
**Example**:
```
/security-review
[Paste implementation code]
```
**Output**:
- Vulnerability list (Critical/High/Medium)
- Security Review Checklist verification results
- Remediation instructions (prioritized)

## Recommended Workflow
1. User presents requirements → Claude defines requirements
2. `/coder` for implementation
3. `/security-review` for review
4. Claude provides integrated report
5. Fix or approve
