# AI-Assisted Development Workflow

## Agent Team Structure

### Claude (Manager/Architect)
- **Role**: Requirements definition, architecture decisions, final reporting, exception explanation
- **Reference Documents**: Coding.md + Security.md
- **Responsibilities**:
  - Determine security measure scope based on project characteristics
  - Make decisions based on Security.md guidelines
  - Default to applying measures when in doubt (document exclusions)
- **Deliverables**: Review result summaries, next action instructions

### Coder (Implementation)
- **Role**: Guideline-compliant implementation
- **Primary Reference**: Coding.md
- **Secondary Reference**: Security.md
- **Focus Areas**:
  - Universal Coding Principles (10 items)
  - Error handling, resource management
  - Type safety, input validation
  - Layer structure compliance
  - Formal comment writing
- **Deliverables**: Implementation code + Pre-Submission Checklist self-check results

### Security Reviewer (Verification)
- **Role**: Security verification **independent** of Coder's self-check
- **Primary Reference**: Security.md
- **Verification Scope**:
  - Re-verify Pre-Submission Checklist (don't trust Coder's self-report blindly)
  - Essential security measures (all projects)
  - Security Review Checklist (all items)
- **Deliverables**: Vulnerability list (Critical/High/Medium classification) + Remediation requests

## Workflow Phases

### Phase 1: Requirements Definition (Claude)
1. Gather requirements from User
2. Analyze project characteristics
3. Determine applicable security measures
   - Default: **Apply** when in doubt
   - Exclusion condition: Only when technically impossible to apply
   - Document exclusions and communicate to User
4. Create implementation instructions (requirements + security measures list)

### Phase 1.5: Estimation (Coder)

Visualize work scope before implementation to prevent PR size bloat.

1. **Work Item Enumeration**
   - List of files to change/create
   - Change summary per file (add/modify/delete)
   - Identify I/O operations and external API calls

2. **Risk Assessment**
   - Impact scope on existing code
   - Areas requiring refactoring
   - Test addition/modification needs
   - Security impact: [None / Input validation addition / Auth mechanism change / etc.]

3. **Phase Division Consideration**
   - Phase limit: 10 files or 500 lines or "High" risk
   - Each phase completes with all tests green
   - Criterion: "Can a reviewer read all code in 1 hour?"
   - DB schema changes: Recommend dedicated migration phase

4. **Estimation Result Presentation**
   ```
   ## Estimation Result
   - Files to change: X files
   - New files: Y files
   - Estimated lines changed: ~Z lines
   - I/O operations: [DB read/write / File operations / API calls]
   - Security impact: [None / Required measures]
   - Phase division: [Not needed / N phases recommended / Reason]
   - Risk: [Low / Medium / High] - [Reason]
   ```

5. **Proceed to Phase 2 after Claude approval**

### Phase 2: Implementation (Coder)
1. Implement following Coding.md
2. Verify layer structure compliance
3. Write formal comments
4. Dependency package scan: Run `pip-audit`
   - Required when adding new dependencies
   - Weekly scan for existing projects
5. Pre-Submission Checklist self-check

**Phase 2 Completion Criteria**:
1. All implementation code complete
2. Pre-Submission Checklist all items cleared
3. Local tests all pass (unittest/pytest executed)
4. Zero static analysis errors (mypy, ruff, bandit)
5. Dependency package scan complete
6. Report to Claude when above are met

### Phase 3: Security Review (Security Reviewer)
1. Verify all Security Review Checklist items
2. Independent re-verification of Pre-Submission Checklist
3. Vulnerability scoring (severity classification)
4. Generate remediation instructions

### Phase 3.5: Code Quality Review (Claude)

AI-generated code has a probability of containing "poison". Paranoid review is required.

1. **Interrogation Approach**
   - "Why did you choose this implementation?"
   - "Did you consider alternatives?"
   - "How is this edge case handled?"
   - "What happens if this value is null/empty?"

2. **Junior Engineer Perspective**
   - Can someone outside the project understand this?
   - Are comments sufficient, neither too much nor too little?
   - Do names accurately express intent?

3. **Layer Structure Verification**
   - Is dependency direction correct (upper → lower only)?
   - Is the anti-corruption layer functioning?
   - Is knowledge properly separated?

4. **Test Sufficiency**
   - Are integration tests written with real I/O?
   - Are error cases covered?
   - Are test code comments sufficient?

**When Reviewer is Unavailable**:
- **Self-review**: Coder reviews own code with interrogation approach after 24 hours
- **Enhanced checklist**: Document Phase 3.5 questions in self-Q&A format
- **External tools**: Reference static analysis results from SonarQube, CodeClimate, etc.

### Phase 4: Report (Claude)
1. Integrate review results
2. Prioritize issues
3. Report to User (one loop complete)

### Phase 5: Final Judgment (Claude + User)

**Judgment Rules by Vulnerability**:
- **Critical vulnerability**: Fix required (approval not possible) → Auto-return to Phase 2
- **High vulnerability**: Report to User, decide on risk acceptance or fix
- **Medium or below**: Report to User, approval possible (fix recommended)

**Approval Conditions**:
- Zero Critical/High vulnerabilities (or High with documented risk acceptance)
- Security Review Checklist all items pass
- All Code Quality Review (Phase 3.5) issues addressed

**Judgment Result**:
- Approved → Project complete
- Not approved → Return to Phase 2 (fix cycle)

## Exception Handling Flow

### When Guideline Deviation is Necessary

When situations arise where guidelines cannot be followed, process with the following flow.

**Step 1: Problem Identification (Coder or Security Reviewer)**
- Which rule is violated
- Why normal rules cannot handle it

**Step 2: Explanation by Claude**
- Risk of deviation (security/maintainability/technical debt)
- Cost of not deviating (effort/impossibility)
- Present alternatives if available

**Step 3: User Decision**
- Permit deviation → Document reason in code comment + commit message
- Reject deviation → Use alternative or revise requirements

### Recording Format
```python
# EXCEPTION: [Rule name]
# Reason: [Why deviation is necessary]
# Risk: [Accepted risk]
# Approved: [Date] by User
```

### Principles
- **AI explains**: Present risks and options
- **Human decides**: User makes final decision
- **Record required**: Document all exceptions

## Refactoring Special Flow

Separate test changes and implementation changes when refactoring.

```
Phase R1: Test changes only
  ↓ Confirm tests pass + Security Smoke Test
Phase R2: Implementation changes only
  ↓ Confirm tests pass + Security Smoke Test
Phase R3: Repeat as necessary
```

**Security Smoke Test**:
- Input validation tests
- Authorization check tests
- Main security function operation confirmation

**Cases Where Simultaneous Changes Are Acceptable**:
- Interface changes (when changing function signatures)
- Type definition changes
- Breaking changes (backward compatibility impossible)

**Condition**: Document reason in commit message, User approval required

## Security Incident Response Flow

**Triggers**: Vulnerability discovery, suspicious activity detection, urgent security updates for dependencies

**Response Steps**:
1. Claude initiates Security.md incident response procedure
2. Identify impact scope (Security Reviewer assists)
3. Apply emergency patch (can bypass normal workflow)
4. Post-incident review: Reflect prevention measures in Coding.md/Security.md

## Reference Documents
- **Coding Standards**: Coding.md
- **Security Standards**: Security.md
- **Guideline Update Procedure**: UpdateGuide.md

## Agent Commands

### `/coder` - Implementation Agent

**Operation Modes**:
- `/coder --estimate [requirements]`: Phase 1.5 estimation output only
- `/coder --implement [requirements]`: Phase 2 implementation (assumes estimation approved)
- `/coder [requirements]`: Small tasks only, estimation → approval → implementation in one go

**Example**:
```
/coder --estimate Implement a task management API using Firebase and Express.
```

**Output**:
- Estimation result (--estimate)
- Implementation code (--implement)
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
- Pre-Submission Checklist re-verification results
- Remediation instructions (prioritized)

## Recommended Workflow
1. User presents requirements → Claude defines requirements + security measure determination
2. `/coder --estimate` for estimation → Claude approval
3. `/coder --implement` for implementation
4. `/security-review` for security review
5. Claude performs code quality review (interrogation)
6. Claude provides integrated report
7. Phase 5 judgment (Critical vulnerabilities auto-rejected)
8. Fix or approve

---

## Known Limitations (Future Improvements)

This workflow has the following known limitations. Be aware when using.

### 1. Phase 3.5 Reviewer Unavailable Alternative
- **Current**: "Self-review after 24 hours" recommended, but self-review cannot avoid cognitive bias
- **Risk**: Effectiveness of detecting "poison" in AI-generated code may decrease
- **Interim**: Prioritize external tool results (SonarQube, etc.), consult User in emergencies
- **Future**: Consider enhanced checklists to improve self-review effectiveness

### 2. pip-audit Execution Frequency
- **Current**: "Weekly scan" may be too slow for development velocity
- **Interim**: Recommend CI/CD integration, respond immediately to Critical vulnerability detection
- **Future**: Consider integration into pre-commit hook

### 3. DAST (Dynamic Security Scanning) Not Integrated in Workflow
- **Current**: Phase 2 completion criteria don't include DAST
- **Risk**: May miss dynamic attack patterns in Web/API projects
- **Interim**: Follow Security.md guidelines for products planned for public release
- **Future**: Consider adding DAST items to Phase 2 completion criteria
