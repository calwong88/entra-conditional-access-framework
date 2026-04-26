# CA003 — Block Access from High-Risk Countries

## Purpose

Block authentication attempts originating from countries designated as high-risk in the `Blocked Countries` named location. Reduces automated attack surface from regions associated with high volumes of credential-based attacks.

## Policy Configuration

| Field | Value |
|-------|-------|
| Policy name | `CA003-Global-LocationBased-BlockHighRiskCountries-v1.0` |
| State | Report-only |
| Users included | All users |
| Users excluded | `CA-Exclude-BreakGlass` |
| Target resources | All cloud apps |
| Conditions | Locations: Include `Blocked Countries` named location |
| Grant control | Block access |
| Session controls | None |

## Named Location: Blocked Countries

The `Blocked Countries` named location currently includes a representative set of countries selected for lab demonstration purposes. **In production, the country list would be informed by:**

- Threat intelligence feeds (Microsoft Threat Intelligence, organizational SIEM data)
- Sanctions and export-control restrictions (OFAC, EU sanctions lists)
- Business presence and travel patterns (no point blocking a country where employees regularly work)
- Risk appetite and regulatory posture

This list should be reviewed quarterly and updated based on observed attack patterns and business changes.

## Why This Policy Matters — and Its Limits

Location-based blocking is a **defense-in-depth** control, not a primary identity control. Its value:

- Eliminates a meaningful percentage of automated, untargeted attack traffic
- Reduces noise in security logs, making targeted attacks easier to spot
- Provides a baseline guardrail for unattended service accounts

Its limits:

- Sophisticated attackers route traffic through VPNs, proxies, or compromised hosts in trusted countries
- Legitimate users who travel may be incorrectly blocked
- IP-to-country geolocation is imperfect; edge cases occur

Location blocking should always be layered with identity-based controls (MFA, risk-based policies). It is never a primary defense.

## User Impact

Users attempting to sign in from countries in the `Blocked Countries` list will be denied access regardless of credentials or MFA status. This includes:

- Travelers visiting blocked countries (intentional impact — requires exception process)
- VPN users connecting through endpoints in blocked countries (often a false positive, requires investigation)
- Service accounts running from infrastructure hosted in blocked regions (should be exception-handled via dedicated exclusion group)

## Exclusions and Rationale

| Excluded entity | Rationale |
|-----------------|-----------|
| `CA-Exclude-BreakGlass` group | Emergency access account must remain reachable in all scenarios. |

In a production deployment, additional exclusion patterns would include a `CA-Exception-Travelers` group with time-bound membership for users with legitimate travel needs.

## Validation via What If Tool

**Test 1 — Sign-in from blocked country:**
- User: `bob.user`
- Country: [country from Blocked Countries list]
- Client app: Browsers
- **Result:** ✅ CA003 applied with Block access. (CA002 also applied with Require MFA, but Block wins.)
- See `screenshots/CA003-whatif-blocked-country.png`

**Test 2 — Sign-in from non-blocked country:**
- User: `bob.user`
- Country: Canada
- Client app: Browsers
- **Result:** ✅ CA003 did NOT apply (reason: Locations). CA002 applied as expected.
- See `screenshots/CA003-whatif-allowed-country.png`

## Rollout Plan

1. Deploy in Report-only mode (current state)
2. Monitor sign-in logs for 14+ days. Location-based policies require longer monitoring than auth-based ones because traveling users may not sign in every day.
3. Build an exception workflow before enforcement — a process for users to request temporary exclusion when traveling.
4. After 14 days, change state to On.
5. Monitor service desk tickets related to "blocked from country" issues for the first 60 days.

## References

- [Microsoft Learn: Conditional Access location condition](https://learn.microsoft.com/en-us/entra/identity/conditional-access/concept-conditional-access-conditions)
- Related policies: CA002 (Require MFA), CA020 (Sign-in risk MFA — covers VPN-bypass scenarios)