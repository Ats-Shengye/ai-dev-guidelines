# AI Development Guidelines

A framework for AI-assisted development with Claude and similar coding assistants.

## Overview

This repository provides a **skeleton framework** for security-conscious AI-assisted development. It includes coding standards, workflow definitions, and quality gates that can be adapted to your projects.

**Important**: This is a reference framework, not a complete plug-and-play solution. You should customize these guidelines for your specific needs.

## File Structure

| File | Description |
|------|-------------|
| **Coding.md** | Coding standards, style guide, architecture principles, pre-submission checklist |
| **Workflow.md** | AI-assisted development workflow, agent team structure, phase definitions |
| **UpdateGuide.md** | Guideline update procedures, review process |

## What's NOT Included

**Security.md** is intentionally not included. Security configurations vary significantly by:
- Project type (Web, API, CLI, library)
- Deployment environment (local, cloud, on-premise)
- Compliance requirements (GDPR, HIPAA, PCI-DSS)
- Risk tolerance

You should create your own Security.md tailored to your project's needs. The Coding.md and Workflow.md files reference Security.md as a placeholder for your custom security standards.

## Core Principles

### Security First
- No compromise on security regardless of project scale
- Apply all applicable security measures
- When in doubt, implement the security measure

### AI Collaboration
- Never trust AI-generated code without review
- Humans decide architecture, AI assists implementation
- Document generation prompts for reproducibility

### Quality Gates
- Static analysis required before commit (mypy, ruff, bandit)
- Pre-submission checklist verification for every change
- Security review for critical code paths

## Workflow Overview

```
User Requirements
      |
[Phase 1] Claude: Requirements Definition
      |
[Phase 1.5] Coder: Work Estimation
      |
[Phase 2] Coder: Implementation (per Coding.md)
      |
[Phase 3] Security Reviewer: Verification (per your Security.md)
      |
[Phase 3.5] Claude: Code Quality Review
      |
[Phase 4] Claude: Integrated Report
      |
[Phase 5] User: Approve or Request Fixes
      |
   Complete
```

## How to Use

### For AI Assistants
Reference these documents in your system prompt or CLAUDE.md. The agent commands (`/coder`, `/security-review`) are conceptual prompts - adapt them to your tooling.

### For Developers
- Use the Pre-Submission Checklist before code submission
- Follow the Universal Coding Principles (10 items in Coding.md)
- Create your own Security.md with project-specific requirements

### For Teams
- Adapt the workflow phases to your development process
- Map agent roles (Coder, Security Reviewer) to team members or AI assistants
- Customize the estimation and review criteria for your context

## Customization Guide

1. **Fork this repository**
2. **Create Security.md** for your project (include: security principles, review checklist, incident response)
3. **Adjust Coding.md** for your language/framework preferences
4. **Modify Workflow.md** phases to match your team structure

## Known Limitations

See the "Known Limitations" sections in Coding.md and Workflow.md for acknowledged gaps and future improvement areas. These are documented transparently.

## License

MIT License - Free to customize and use.
