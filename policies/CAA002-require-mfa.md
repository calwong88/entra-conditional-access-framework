## Policy Interaction with CA001

When a user attempts legacy authentication, both CA001 and CA002 
evaluate and apply. CA001 issues a Block, CA002 requires MFA. 
Microsoft's evaluation logic always gives precedence to Block access 
over any Grant control, so the effective outcome is: access denied.

This demonstrates the "deny wins" principle of Conditional Access 
evaluation and justifies why CA001 is numbered first — it acts as a 
fail-safe block for legacy auth regardless of whether other policies 
would otherwise allow the sign-in.