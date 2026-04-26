# CA010 — Require Phishing-Resistant MFA for Admins

## Purpose

Require administrators to authenticate using cryptographically phishing-resistant methods (FIDO2 security keys, Windows Hello for Business, or certificate-based authentication). Mitigates adversary-in-the-middle (AiTM) attacks targeting privileged accounts.

## Policy Configuration

| Field | Value |
|-------|-------|
| Policy name | `CA010-Admins-AuthStrength-PhishingResistantMFA-v1.0` |
| State | Report-only |
| Users included | `CA-Persona-Admins` group + 13 privileged directory roles |
| Users excluded | `CA-Exclude-BreakGlass` |
| Target resources | All cloud apps |
| Conditions | None |
| Grant control | Require authentication strength: Phishing-resistant MFA |
| Session controls | None |

## Why Phishing-Resistant MFA, Not Just MFA

Standard MFA methods (SMS, voice call, Microsoft Authenticator push notifications) are vulnerable to adversary-in-the-middle (AiTM) phishing attacks. In an AiTM attack, the attacker hosts a reverse proxy that mimics the legitimate sign-in page. When the user enters credentials and approves the MFA prompt on their device, the attacker captures the resulting session token and gains authenticated access.

Phishing-resistant methods — FIDO2 security keys, Windows Hello for Business, and certificate-based authentication — defeat AiTM attacks because the cryptographic signature is bound to the actual destination origin. A proxied sign-in page produces an invalid signature.

Microsoft recommends phishing-resistant MFA for all privileged roles. This policy implements that recommendation.

## Authentication Strengths Concept

This policy uses the built-in **Phishing-resistant MFA** authentication strength, which accepts only:

- FIDO2 security keys (e.g., YubiKey, Feitian)
- Windows Hello for Business
- Certificate-based authentication (X.509)

It rejects:

- Password
- SMS / voice call
- Microsoft Authenticator notification
- Microsoft Authenticator OTP / Software OATH tokens
- Email OTP (for guests)

The two other built-in authentication strengths (Multi-factor authentication, Passwordless MFA) accept progressively broader sets of methods.

## Dual Targeting: Group + Directory Roles

This policy targets both the `CA-Persona-Admins` security group AND Entra directory roles directly. This is intentional defense-in-depth:

- Group targeting catches users explicitly designated as administrators
- Role targeting catches users who hold privileged roles through any assignment path — including PIM eligible/active assignments, direct role grants, or assignments made outside the persona group structure

Targeting only the group would create a gap: a user granted a privileged role outside the group would not be caught. Targeting only roles would miss users who are administrators by job function but don't currently hold an active role assignment. Both are required for complete coverage.

Privileged directory roles included:
- Global Administrator
- Privileged Role Administrator
- User Administrator
- Security Administrator
- Helpdesk Administrator
- Authentication Administrator
- Conditional Access Administrator
- Application Administrator
- Cloud Application Administrator
- Privileged Authentication Administrator
- Exchange Administrator
- SharePoint Administrator
- Billing Administrator

## User Impact

Administrators must register a phishing-resistant authentication method before this policy is enforced. Acceptable methods:

- A FIDO2 security key (e.g., YubiKey 5 series). Cost: ~$50-60 per key. Best practice is to register two per admin (primary + backup).
- Windows Hello for Business on a corporate-managed Windows device.
- A certificate issued by an enterprise PKI bound to the user's smart card or device.

Until phishing-resistant methods are registered tenant-wide, this policy must remain in Report-only mode. Premature enforcement would lock out admins.

## Exclusions and Rationale

| Excluded entity | Rationale |
|-----------------|-----------|
| `CA-Exclude-BreakGlass` group | Emergency access account remains available with password-only authentication for crisis recovery. |

## Validation via What If Tool

**Test 1 — Admin user (alice.admin):**
- Result: ✅ CA002 (Require MFA) and CA010 (Require Phishing-resistant MFA) both apply. CA010's stricter requirement effectively supersedes CA002.
- See `screenshots/CA010-whatif-admin.png`

**Test 2 — Non-admin user (bob.user):**
- Result: ✅ Only CA002 applies. CA010 correctly does NOT apply because bob is not in the admin persona group and holds no privileged directory role.
- See `screenshots/CA010-whatif-nonadmin.png`

## Lab Limitation

This lab does not have FIDO2 security keys provisioned. The policy is configured correctly and validated via What If, but live admin sign-ins cannot be tested end-to-end. In a production rollout, the steps would be:

1. Procure FIDO2 keys for all admins (primary + backup per admin)
2. Conduct registration sessions with admin team
3. Verify all admins have at least one phishing-resistant method registered
4. Move CA010 from Report-only to On

A future iteration of this lab will incorporate a FIDO2 security key to demonstrate the registration flow and end-to-end authentication.

## Rollout Plan

1. Deploy in Report-only mode (current state)
2. Communicate to admin team: phishing-resistant method registration required within 30 days
3. Procure and distribute FIDO2 keys (or enable Windows Hello for Business via Intune)
4. Run admin registration sessions
5. Monitor sign-in logs for `Conditional Access: Report-only: Failure` for CA010 — these indicate admins still using non-phishing-resistant methods
6. After 100% of admins have registered phishing-resistant methods, change state to On
7. Maintain the break-glass exclusion permanently

## References

- [Microsoft Learn: Conditional Access authentication strength](https://learn.microsoft.com/en-us/entra/identity/authentication/concept-authentication-strengths)
- [Microsoft Learn: Securing privileged access](https://learn.microsoft.com/en-us/entra/identity/role-based-access-control/security-planning)
- Related policies: CA002 (Require MFA — baseline), future CA040 (passwordless for all users)