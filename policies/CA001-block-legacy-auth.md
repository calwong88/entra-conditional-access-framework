# CA001 — Block Legacy Authentication

## Purpose

Block sign-ins using legacy authentication protocols that do not support modern authentication or MFA.

## Policy Configuration

| Field | Value |
|-------|-------|
| Policy name | `CA001-Global-BaseProtection-BlockLegacyAuth-v1.0` |
| State | Report-only |
| Users included | All users |
| Users excluded | `CA-Exclude-BreakGlass` |
| Target resources | All cloud apps |
| Conditions | Client apps: Exchange ActiveSync clients + Other clients |
| Grant control | Block access |
| Session controls | None |

## Why This Policy Matters

Legacy protocols — IMAP, POP, SMTP AUTH, older MAPI clients, basic authentication for Exchange ActiveSync — do not support multifactor authentication. Attackers use these endpoints for password spray attacks because they can bypass MFA entirely. Microsoft has published data showing that blocking legacy authentication alone defeats the overwhelming majority of password spray attempts in real-world tenants.

This policy is numbered first in the framework because it acts as a fail-safe block: even if other policies (CA002, CA010) require MFA for modern sign-ins, legacy protocols would still represent a bypass without CA001 in place.

## User Impact

**No impact for users on modern clients:**
- Outlook 2016 or newer with modern authentication enabled
- Outlook on the web, Outlook mobile
- Microsoft Teams, SharePoint Online, OneDrive
- Microsoft Authenticator app

**Blocked clients:**
- Outlook 2010, Outlook 2013 with basic authentication
- Third-party mail clients using IMAP or POP without OAuth 2.0
- Scripts or applications using SMTP AUTH for sending mail
- Older mobile mail apps using Exchange ActiveSync basic auth

## Exclusions and Rationale

| Excluded entity | Rationale |
|-----------------|-----------|
| `CA-Exclude-BreakGlass` group | Emergency access account must not depend on any policy that could lock out recovery. Standard break-glass design pattern. |

## Validation via What If Tool

**Test 1 — Legacy auth should be blocked:**
- User: `bob.user`
- Cloud app: Office 365
- Client app: Mobile apps and desktop clients - Other clients
- **Result:** ✅ CA001 applied, Report-only: Block access
- See `screenshots/CA001-whatif-legacy.png`

**Test 2 — Modern auth should NOT be affected:**
- User: `bob.user`
- Cloud app: Office 365
- Client app: Browsers
- **Result:** ✅ 0 policies applied. CA001 listed under "will not apply" with reason: "Client app"
- See `screenshots/CA001-whatif-browser.png`

## Rollout Plan

1. Deploy in Report-only mode (current state)
2. Monitor sign-in logs for `Conditional Access: Report-only: Failure` entries over 7+ days
3. Identify any users or service accounts hitting the policy via log analysis
4. Communicate with affected users and provide migration paths to modern auth
5. After 7+ days with no critical impacts, change state from Report-only to On
6. Continue monitoring for 30 days post-enforcement

## Lab Limitations

This lab does not have Exchange Online licenses or real legacy mail clients connected, so live legacy auth traffic cannot be generated. Policy correctness was validated via the What If simulation tool. In a production rollout, the Report-only phase would be backed by real sign-in log evidence.

## References

- [Microsoft Learn: Block legacy authentication](https://learn.microsoft.com/en-us/entra/identity/conditional-access/policy-block-legacy-authentication)
- Related policies: CA002 (Require MFA for All Users), CA010 (Admin phishing-resistant MFA)