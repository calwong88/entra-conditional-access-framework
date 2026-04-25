# CA001 - Block Legacy Authentication

## Purpose
Block sign-ins using legacy authentication protocols that do not 
support modern authentication and MFA.

## Policy Details
- **Policy name:** CA001-Global-BaseProtection-BlockLegacyAuth-v1.0
- **State:** Report-only
- **Target:** All users except break-glass
- **Resources:** All cloud apps
- **Condition:** Client apps = Exchange ActiveSync clients + Other clients
- **Action:** Block access

## Why This Policy Matters
Legacy protocols (IMAP, POP, SMTP AUTH, older MAPI clients, basic 
authentication for Exchange ActiveSync) do not support MFA. Attackers 
use password spray attacks against these endpoints to bypass MFA. 
Microsoft published data showing this single control blocks >99% of 
password spray attempts.

## User Impact
Minimal for modern environments. Users on:
- Outlook 2016+ with modern auth enabled: no impact
- Outlook mobile, Outlook on the web: no impact  
- Teams, SharePoint Online, OneDrive: no impact

Impact to:
- Outlook 2010, 2013 (basic auth): blocked
- Third-party mail clients using IMAP/POP without OAuth: blocked
- Scripts using SMTP AUTH to send mail: blocked
- Older phone mail apps using EAS basic auth: blocked

## Exclusions and Rationale
- **CA-Exclude-BreakGlass:** Emergency access account must not be 
  dependent on any policy that could lock out recovery.

## Rollout Plan
1. Deploy in Report-only mode (current state)
2. Monitor sign-in logs for Report-only: Failure entries over 7 days
3. Identify affected users/services via the log analysis
4. Communicate to affected users, provide migration path
5. After 7 days with no critical impacts, enable policy (State: On)

## Validation via What If Tool

**Test 1: Legacy auth should be blocked**
- User: bob.user
- Cloud app: Office 365
- Client app: Mobile apps and desktop clients - Other clients
- Expected: CA001 applies with Block access
- Actual: ✅ CA001 applied, Report-only: Block access
- [CAA001-screenshot-test1.png]

**Test 2: Modern auth should NOT be affected**
- User: bob.user  
- Cloud app: Office 365
- Client app: Browser
- Expected: CA001 does NOT apply
- Actual: ✅ 0 policies applied
- [CAA001-screenshot-test2.png]

## Testing Notes
In this lab, legacy auth testing is limited — no Exchange Online 
licenses to generate real legacy auth traffic. Verified policy 
configuration correctness via the "What If" tool.

## References
- Microsoft docs: [link]
- Related: CA002 (Require MFA), CA010 (Admin phishing-resistant MFA)