# Entra Conditional Access Framework

A production-grade Conditional Access (CA) policy framework for Microsoft Entra ID, built and validated in a hybrid identity lab environment. This repository documents the design decisions, policy configurations, validation testing, and rollout planning behind a layered Zero Trust access model.

## Project Goals

This project demonstrates:

- A tiered Conditional Access architecture (baseline → persona → risk-based → session controls)
- Documentation discipline appropriate for change management and audit
- Validation methodology using the Entra What If tool before production rollout
- Awareness of policy interaction, evaluation order, and "deny wins" semantics
- Operational considerations: break-glass design, exclusion patterns, rollout sequencing

## Lab Environment

- **Tenant:** Microsoft Entra ID (P2 trial)
- **Identity model:** Hybrid — on-premises Active Directory synced via Entra Connect to a cloud tenant
- **Infrastructure:** Domain controller, Entra Connect server, and client VM hosted on a local Hyper-V environment
- **User population:** 11 cloud and synced users grouped into operational personas (admins, end users, contractors, executives, helpdesk, guests)
- **Licensing:** Entra ID P2 assigned to all test users to enable risk-based policies, PIM, and Identity Protection

## Policy Framework

Policies follow a structured naming convention:

```
CA[number]-[scope]-[category]-[action]-v[version]
```

Example: `CA001-Global-BaseProtection-BlockLegacyAuth-v1.0`

This convention enables sortable, searchable, auditable policy management as the framework scales.

| Policy | Scope | Category | Action | Status |
|--------|-------|----------|--------|--------|
| CA001 | Global | Base Protection | Block legacy authentication | ✅ Built, validated |
| CA002 | Global | Base Protection | Require MFA for all users | ✅ Built, validated |
| CA003 | Global | Location-Based | Block high-risk countries | ✅ Built, validated |
| CA010 | Admins | Auth Strength | Require phishing-resistant MFA | 🔨 In progress |
| CA011 | Guests | Resource Restriction | Block admin portals | ⏳ Planned |
| CA012 | Contractors | Device Trust | Require compliant device | ⏳ Planned |
| CA020 | Global | Risk-Based | Sign-in risk MFA | ⏳ Planned |
| CA021 | Global | Risk-Based | User risk password change | ⏳ Planned |
| CA030 | Admins | Session Control | Sign-in frequency | ⏳ Planned |

Each policy has its own documentation file in `policies/` covering purpose, configuration, user impact, exclusions, validation, and rollout plan.

## Repository Structure

```
.
├── README.md                  # This file
├── policies/                  # Per-policy documentation
├── personas/                  # User persona model and group design
├── diagrams/                  # Architecture and flow diagrams
├── screenshots/               # Supporting validation evidence
├── exports/                   # JSON exports of CA policies (policy-as-code)
└── LICENSE
```

## Design Principles

**Break-glass first.** Every policy excludes a designated emergency access account, secured by a dedicated security group (`CA-Exclude-BreakGlass`). The break-glass account has no MFA dependency, allowing recovery if the rest of the framework fails.

**Report-only by default.** All policies are deployed in Report-only mode, monitored via sign-in logs for unintended impact, and only enforced after validation. This is Microsoft's recommended rollout pattern and is reflected in every policy's rollout plan.

**Persona-based scoping.** Rather than targeting "All users" directly, policies scope through assigned security groups (`CA-Persona-Admins`, `CA-Persona-Contractors`, etc.). This creates explicit, auditable membership and supports temporary exception workflows in operations.

**Defense in depth.** Policies layer rather than overlap. CA001 (block legacy auth) and CA002 (require MFA) work together: legacy auth is blocked outright by CA001, while CA002 requires MFA for all modern auth sign-ins. Where policies conflict, Block always wins over Grant — a core CA evaluation principle.

## Gotchas Encountered

Real-world detail — these are issues hit during lab construction that aren't obvious from documentation alone:

- **Five overlapping MFA enforcement mechanisms exist in Entra:** Security Defaults, per-user MFA (legacy), Conditional Access, the Authentication Methods Registration Campaign, and the Identity Protection MFA Registration Policy. Disabling one doesn't disable the others. Lab break-glass setup required disabling or scoping all five.
- **The Registration Campaign is "Microsoft managed" by default in new tenants.** It will prompt break-glass accounts to register Authenticator unless explicitly excluded. This is a critical gotcha for break-glass design.
- **What If tool's "Reasons why this policy will not apply" output** is a powerful validation signal — confirms not just that a policy didn't fire, but *why* it didn't fire (e.g., "Client app" mismatch). Used as standard validation evidence in this framework.

## Status

Work in progress. Currently implementing the baseline tier. Expected completion: May 22, 2026.

## Author

Calvin Wong — pursuing SC-300 (Microsoft Identity and Access Administrator) certification, with a background in Azure administration, cloud security (CompTIA Security+), and IT service management (ITIL 4). Building toward an IAM engineering role.

[[LinkedIn URL](https://www.linkedin.com/in/calvinkmwong/)] · [Other relevant links]