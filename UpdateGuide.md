# Coding & Security Guidelines Update Procedure

## Purpose
This document defines the standardized process for updating Coding.md and Security.md to maintain their relevance, accuracy, and robustness over time.

## When to Update

### Scheduled Reviews
- **Quarterly**: Every 3 months for routine review
- **Semi-annually**: Every 6 months for comprehensive update
- **Annually**: Full audit and major revision

### Trigger Events
- New major attack vectors or exploit techniques emerge
- Significant changes in Python ecosystem (new versions, deprecated features)
- Security incidents reveal guideline gaps
- Discovery of outdated or incorrect recommendations
- New static analysis tools become industry standard
- Breaking changes in mypy/ruff/bandit requiring configuration updates

## Update Process

### Phase 1: Information Gathering

#### 1.1 Search Strategy
Execute web searches covering multiple dimensions:

**AI-Generated Code Issues**
```
- "AI generated code common bugs mistakes [CURRENT_YEAR]"
- "LLM coding assistant hallucination errors security vulnerabilities"
- "AI code generation best practices security [CURRENT_YEAR]"
```

**General Coding Security**
```
- "common programming mistakes Python security [CURRENT_YEAR]"
- "Python security best practices [CURRENT_YEAR]"
- "static analysis Python mypy ruff bandit best practices"
```

**Security Incidents and Trends**
```
- "web service security incidents [CURRENT_YEAR] breaches vulnerabilities"
- "cyber attack techniques [CURRENT_YEAR] OWASP trends"
- "supply chain attacks [CURRENT_YEAR] Python npm"
- "OWASP Top 10 [CURRENT_YEAR]"
```

#### 1.2 Information Sources Priority
1. **Academic papers** (arXiv, IEEE, ACM) - for foundational understanding
2. **Security advisories** (CVE databases, OWASP, NIST) - for current threats
3. **Industry reports** (Verizon DBIR, Mandiant, Snyk) - for trends and statistics
4. **Technical blogs** (authoritative sources only) - for practical guidance
5. **Tool documentation** (mypy, ruff, bandit official docs) - for tool-specific updates

#### 1.3 Critical Evaluation Criteria
When reviewing collected information, filter for:

**Universal vs. Specific**
- Keep: Principles that apply regardless of tool/version
- Discard: Statistics tied to specific LLM versions or tools
- Discard: Temporary bugs in specific software versions

**Mechanism vs. Symptom**
- Keep: Why vulnerabilities occur (root causes)
- Keep: How to prevent entire classes of issues
- Discard: Individual incident anecdotes without generalizable lessons

**Actionable vs. Theoretical**
- Keep: Concrete practices with clear implementation
- Keep: Verifiable checklist items
- Discard: Vague recommendations without clear action

**Time-Tested vs. Trendy**
- Keep: Established security principles (least privilege, defense in depth)
- Evaluate: New practices - verify with multiple authoritative sources
- Discard: Hype-driven recommendations without peer review

### Phase 2: Content Organization

#### 2.1 Summary Document
Before drafting updates, create a summary document:

**Structure:**
```
## New Principles to Add
- [Principle Name]: [Rationale and evidence]

## Existing Items to Modify
- [Item Name]: [Problem] → [Proposed fix]

## Items to Remove
- [Item Name]: [Reason for removal]

## New Tools/Commands
- [Tool Name]: [Purpose and rationale for adoption]
```

This summary serves as:
- Sanity check before drafting
- Reference for discussion
- Archive of decision rationale

#### 2.2 Cross-Reference Check
Before finalizing updates:
- Ensure Coding.md and Security.md remain consistent
- Verify checklist items match principle sections
- Confirm command examples reference correct config files
- Check that all mentioned tools have corresponding setup instructions

### Phase 3: Review Process

#### 3.1 Technical Review
After drafting updates, request review with specific focus areas:

**Review Request Template:**
```
Please review the updated Coding.md and Security.md for:

1. Technical Accuracy
   - Are commands syntactically correct?
   - Will configurations work as described?
   - Are file paths and permissions appropriate?

2. Operational Clarity
   - Are there ambiguous instructions that could be misinterpreted?
   - Do we specify concrete values (paths, modes, frequencies)?
   - Is the "who does what when" clear?

3. Scalability Concerns
   - Will instructions break when project grows?
   - Are there hardcoded assumptions that limit future expansion?

4. Missing Details
   - Are there gaps in configuration setup?
   - Do we specify what to do when things go wrong?
   - Is the failure mode for each recommendation clear?
```

#### 3.2 Iterative Refinement
- Address ALL review feedback - do not skip technical corrections
- If security holes are identified, fix immediately
- Re-submit for review if major changes made
- Continue until review gives approval or has no further technical objections

### Phase 4: Documentation Standards

#### 4.1 Format Requirements

**Good Example Style (Preferred)**
```markdown
### Principle Name
- **Action**: specific requirement or practice
- **Rationale**: why this matters
- **Implementation**: how to actually do it
```

**No Bad/Good Code Comparisons**
- Reason: AI models may learn from bad examples
- Exception: Only if good example is incomprehensible without context
- Alternative: Describe the anti-pattern in text, show only the correct approach

**Command Examples**
- Always include comments explaining what each command does
- Specify configuration file locations
- Show expected output or success indicators when helpful
- Include error handling or fallback procedures

#### 4.2 Checklist Maintenance
Every principle section must have corresponding checklist items:

- **Pre-Submission Checklist** (Coding.md): verification before code commit
- **Security Review Checklist** (Security.md): security-focused validation

When adding a new principle:
1. Write the principle explanation
2. Add corresponding checklist item(s)
3. Ensure checklist is actionable (yes/no answer possible)

### Phase 5: Update Execution

#### 5.1 File Modification
- Use Edit tool for existing sections
- Maintain existing structure and flow
- Keep technical content in English for broader applicability

#### 5.2 Version Documentation
After completing update, record:

```markdown
## Update History

### [YYYY-MM-DD] - [Update Type]
**Trigger**: [Why this update was performed]

**Major Changes**:
- Coding.md: [summary of changes]
- Security.md: [summary of changes]

**New Principles Added**: [list]
**Deprecated Items Removed**: [list]
**Tools/Commands Updated**: [list]

**Review Notes**: [Key feedback from review]
```

## Quality Assurance

### Before Finalizing Update

- [ ] All web search results critically evaluated for universality
- [ ] Model-specific or version-specific statistics excluded
- [ ] Summary document created and reviewed
- [ ] Technical review completed with all issues addressed
- [ ] Cross-references between Coding.md and Security.md verified
- [ ] All checklists updated to match new principles
- [ ] Command examples tested for syntax correctness
- [ ] File paths and permissions specified with concrete values
- [ ] No "Bad example" code blocks added (text description only)
- [ ] Update history recorded

### Post-Update Validation

- [ ] Read through both files end-to-end for flow and consistency
- [ ] Verify all tools mentioned are actually available/installable
- [ ] Check that pyproject.toml references are consistent
- [ ] Ensure context is maintained (no enterprise-only recommendations for personal dev)

## Continuous Improvement

### Feedback Loop
When actual coding work reveals issues:
1. Note the gap or error in guidelines
2. Determine if it's an edge case or fundamental flaw
3. If fundamental: schedule immediate update
4. If edge case: collect and address in next scheduled review

### Tracking Emerging Topics
Maintain awareness of:
- New OWASP Top 10 releases
- Python version EOL and new releases
- Major CVEs affecting Python ecosystem
- Evolution of AI coding assistants and their common pitfalls
