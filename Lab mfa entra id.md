🔐 Lab: MFA & Phishing-Resistant Authentication — Microsoft Entra ID


Series: AZ-500 / SC-500 Hands-on Labs
Platform: Microsoft Entra ID
Duration: ~20 minutes
Level: Intermediate




📋 Overview

This lab covers the full configuration of multifactor authentication in Microsoft Entra ID — from initial MFA registration all the way to phishing-resistant authentication via Passkey (FIDO2). The goal is to understand the practical difference between "having MFA enabled" and "having MFA that actually protects against modern attacks."


🎯 Objectives


 Configure an MFA Registration Policy via Identity Protection
 Create a Conditional Access Policy requiring MFA for cloud admin portals
 Build a custom Authentication Strength enforcing phishing-resistant MFA
 Enable and register a Passkey (FIDO2) via Microsoft Authenticator
 Verify end-to-end phishing-resistant login



🧠 Key Concepts

MFA Registration Policy vs Conditional Access

These are two distinct controls that are frequently confused:

MFA Registration PolicyConditional AccessLocationIdentity ProtectionEntra ID → SecurityWhat it doesForces users to register an MFA methodRequires MFA to access a resourceImmediate impactNone — user still accesses without MFABlocks access if MFA is not completedUse caseGradual rollout with zero disruptionActive enforcement of security policies

Why Create a Custom Authentication Strength?

Entra ID ships with built-in strengths (e.g. "Phishing-resistant MFA"), but building a custom one allows you to:


Define exactly which methods are accepted (e.g. FIDO2 Passkey + Microsoft Authenticator, explicitly excluding SMS)
Align with specific regulatory requirements (GDPR, DORA, NIS2)
Apply different controls per application or user risk tier


Passkey (FIDO2) — The Why

Traditional MFA (SMS, TOTP) is still vulnerable to Adversary-in-the-Middle (AiTM) attacks:

[User] → [Phishing proxy (e.g. Evilginx)] → [Legitimate site]
                  ↑ intercepts MFA token in real time

A FIDO2 Passkey cryptographically binds authentication to the legitimate domain. A fake site can never complete the WebAuthn handshake — even with the user's password in hand.

MFA TypePhishing-resistant?AiTM-resistant?SMS OTP❌❌TOTP (Google/Microsoft Auth)❌❌Push notification❌❌Passkey (FIDO2)✅✅


🔧 Tasks Completed

Task 1 — MFA Registration Policy

Path: Entra ID → Protection → Identity Protection → MFA Registration Policy


Scoped to test user (Delia Dennis) via Include: Select individuals and groups
Administrator account excluded via Exclude tab (break-glass account protection)
Policy set to Enabled
Result: user is prompted to register Microsoft Authenticator on next login without being blocked from accessing resources


Task 2 — Conditional Access: Require MFA for Admin Portals

Path: Entra ID → Security → Conditional Access → + New policy

Name:           Require MFA for portals
Users:          Delia Dennis (included) | Administrator (excluded)
Target:         Microsoft Admin Portals
Grant:          Require multifactor authentication
Status:         On

Verification:


Login to office.com → no MFA prompt (not in scope)
Login to entra.microsoft.com → MFA required (target resource matched)


This confirms Conditional Access is enforcing per resource, not per URL pattern.

Task 3 — Phishing-Resistant MFA with Passkey (FIDO2)

3.1 — Enable the authentication method:
Entra ID → Authentication methods → Policies → Passkey (FIDO2) → Enable → Save

3.2 — Create a custom Authentication Strength:

Name:         SC500 phishing resistant MFA
Description:  Forces login with phishing-resistant MFA
Methods:      Passkeys (FIDO2) + Microsoft Authenticator

3.3 — Update the Conditional Access policy:


Grant: replace Require MFA → Require authentication strength
Select: SC500 phishing resistant MFA


3.4 — Register Passkey via Bluetooth:


Login as Delia Dennis → prompted to set up Passkey
Select Other options → Having trouble? → Create your passkey a different way → iPhone/Android
QR code generated on screen → scan with phone camera
Bluetooth handshake between PC and phone → Passkey saved to Microsoft Authenticator


3.5 — Login with Passkey:

Username → QR code → phone camera → biometric/PIN → authenticated


⚠️ Technical Notes


VMs do not support Bluetooth — Passkey registration must be done on a physical PC browser, not inside a virtualized lab environment
New Authentication Strengths may take a few minutes to appear in Conditional Access dropdowns — signing out and back in resolves this
Always exclude administrator and break-glass accounts from restrictive MFA policies to prevent accidental tenant lockout
The "Name your passkey" screen can hang in some lab environments — this is a lab artifact, not a production issue



🔗 References


Microsoft Docs — MFA Registration Policy
Microsoft Docs — Conditional Access Overview
Microsoft Docs — Authentication Strengths
Microsoft Docs — FIDO2 Passkeys
MITRE ATT&CK — T1111: MFA Interception
AiTM Phishing with Evilginx — Research Overview
