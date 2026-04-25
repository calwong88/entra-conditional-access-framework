# CA002 — Require MFA for All Users

## Purpose

Require multifactor authentication for all user sign-ins to all cloud applications. Establishes the baseline authentication floor for modern auth traffic.

## Policy Configuration

| Field | Value |
|-------|-------|
| Policy name | `CA002-Global-BaseProtection-RequireMFA-v1.0` |
| State | Report-only |
| Users included | All users |
| Users excluded | `CA-Exclude-BreakGlass` |
| Target resources | All cloud apps |
| Conditions | None (applies to all client app types) |
| Grant control | Require multifactor authentication |
| Session controls | None |

## Why This Policy Matters

CA001 closes the legacy authentication bypass; CA002 raises the floor for everything that remains. Together they establish the baseline: no legacy auth allowed, and modern auth must be MFA-protected.

This policy applies broadly (no Client app filter) because modern authentication clients support MFA. Restricting it to specific client types would create gaps.

## User Impact

All users (except break-glass) will be prompted for MFA on:
- New device sign-ins
- Sign-ins after their session expires
- Sign-ins from new networks (depending on session policies layered later)

Frequency depends on default sign-in frequency settings and any session controls layered in later policies (e.g., CA030 for admins).

## Exclusions and Rationale

| Excluded entity | Rationale |
|-----------------|-----------|
| `CA-Exclude-BreakGlass` group | Emergency access account must not be MFA-dependent. |

## Policy Interaction with CA001

When a user attempts legacy authentication, both CA001 and CA002 evaluate and apply. CA001 issues a Block, CA002 requires MFA. Microsoft's evaluation logic always gives precedence to **Block access** over any Grant control, so the effective outcome is access denial.

This demonstrates the "deny wins" principle of Conditional Access evaluation and validates the policy numbering: CA001 acts as a fail-safe block for legacy auth regardless of other policy outcomes.

## Validation via What If Tool

**Test 1 — Browser sign-in:**
- Client app: Browsers
- **Result:** ✅ CA002 applied with Require MFA
- See `screenshots/CA002-whatif-browser.png`

**Test 2 — Modern mobile/desktop sign-in:**
- Client app: Mobile apps and desktop clients - Modern auth
- **Result:** ✅ CA002 applied with Require MFA
- See `screenshots/CA002-whatif-modern.png`

**Test 3 — Legacy auth (policy stacking validation):**
- Client app: Other clients
- **Result:** ✅ Both CA001 (Block) and CA002 (Require MFA) applied. Block wins per evaluation logic.
- See `screenshots/CA002-whatif-stacking.png`

## Rollout Plan

1. Deploy in Report-only mode (current state)
2. Monitor sign-in logs over 7+ days, focusing on users without registered MFA methods
3. Run a parallel MFA registration campaign for users not yet registered (Authentication Methods → Registration campaign)
4. After 90% of active users have registered an MFA method, change state to On
5. Maintain Report-only on a subset (e.g., service accounts via separate exclusions) until those edge cases are addressed
6. Monitor for 30 days post-enforcement

## References

- [Microsoft Learn: Require MFA for all users](https://learn.microsoft.com/en-us/entra/identity/conditional-access/policy-all-users-mfa-strength)
- Related policies: CA001 (Block legacy auth), CA010 (phishing-resistant MFA for admins), CA020 (sign-in risk MFA)