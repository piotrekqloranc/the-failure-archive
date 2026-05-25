
---
title: "M365 Account Hijacking via Tycoon2FA Device Code Phishing"
category: "Cybersecurity"
date_of_incident: 2024
system_affected: "Microsoft 365 / OAuth 2.0 Device Authorization Flow"
failure_type: "OAuth Exploitation / Device Code Phishing / Adversary-in-the-Middle (AiTM)"
severity: "High"
---

### 1. System Architecture and Context

* Microsoft 365 relies on OAuth 2.0 and Azure Active Directory (Microsoft Entra ID) to handle secure user authorization and token issuance.
* The architecture includes an OAuth 2.0 Device Authorization Grant flow, designed to authorize client applications on input-constrained or secondary devices.
* This protocol allows a secondary device to poll Microsoft's token endpoint after generating a short, temporary 9-character code.
* Once a user manually inputs this code on a primary trusted browser via the official login interface, Entra ID completes the handshake and resolves MFA and Conditional Access rules entirely inside official identity infrastructures.

### 2. What went wrong (Trigger)

* Threat actors deployed the Tycoon2FA Adversary-in-the-Middle (AiTM) phishing kit to exploit user trust and bypass standard MFA mechanisms.
* Instead of proxying raw usernames and passwords, the malicious infrastructure initiated an automated backend OAuth device code request to Microsoft's APIs on behalf of a native first-party application.
* The phishing server received a legitimate 9-character login code and dynamically presented it to the victim via a deceptive, spoofed enterprise interface, instructing them to activate it at Microsoft's official endpoint.

### 3. Cascading Effects and Impact

* The victim logged into their official Microsoft account using a trusted workstation, successfully fulfilling all active MFA challenges (Fido2, Authenticator app, or SMS OTP).
* Concurrently, the Tycoon2FA backend server continuously polled Microsoft's OAuth token API (`/oauth2/v2.0/token`) using the generated device code.
* Upon completion of the user's login, Entra ID released wide-scope authenticated access and refresh tokens directly to the attacker's polling endpoint.
* The threat actors hijacked the active session context, completely bypassing Conditional Access boundaries because the initial authentication process occurred on legitimate Microsoft assets.
* Attackers automated post-compromise routines: establishing persistent inbox routing rules, exfiltrating corporate directories, and launching secondary Business Email Compromise (BEC) campaigns.

### 4. Resolution and Prevention

* Enterprise administrators implemented tenant hardening by systematically disabling human-driven device code authorization support inside Entra ID.
* Deployed strict Conditional Access Policies (CAP) that restrict token generation based on managed, compliant device states, isolating unverified assets.
* Built SIEM detection rules to identify high-frequency polling requests targeting token endpoints and alert on anomalous device authorization events across distinct geographic IP blocks.
* Updated internal OAuth architecture guidelines to restrict token scopes, require unique client secrets or certificates for application authentication, and enforce short-lived, single-use token lifecycles tied to verified client endpoints.
