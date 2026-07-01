# Lab: MFA & Phishing-Resistant Authentication — Microsoft Entra ID

**Series:** AZ-500 / SC-500 Hands-on Labs  
**Platform:** Microsoft Entra ID  
**Duration:** ~20 minutes  
**Level:** Intermediate

---

## Overview

This lab covers the full configuration of multifactor authentication in Microsoft Entra ID — from initial MFA registration all the way to phishing-resistant authentication via Passkey (FIDO2). The goal is to understand the practical difference between "having MFA enabled" and "having MFA that actually protects against modern attacks."

---

## What this lab covers

- Configuring an MFA Registration Policy via Identity Protection
- Creating a Conditional Access Policy requiring MFA for cloud admin portals
- Building a custom Authentication Strength enforcing phishing-resistant MFA
- Enabling and registering a Passkey (FIDO2) via Microsoft Authenticator
- Verifying end-to-end phishing-resistant login

---

## Key concepts before starting

### MFA Registration Policy vs Conditional Access

These are two distinct controls that are frequently confused:

| | MFA Registration Policy | Conditional Access |
|---|---|---|
| **Location** | Identity Protection | Entra ID > Security |
| **What it does** | Forces users to register an MFA method | Requires MFA to access a resource |
| **Immediate impact** | None — user still accesses without MFA | Blocks access if MFA is not completed |
| **Use case** | Gradual rollout with zero disruption | Active enforcement of security policies |

The Registration Policy is not enforcement. It is onboarding. A user who has registered MFA through the Registration Policy can still sign in with only a password until a Conditional Access policy requires MFA at the resource level. This separation allows organisations to roll out MFA to all users before turning on enforcement — reducing helpdesk load and avoiding lockouts.

### Why build a custom Authentication Strength

Entra ID ships with built-in strengths such as "Phishing-resistant MFA," but building a custom one allows you to:

- Define exactly which methods are accepted — for example, FIDO2 Passkey and Microsoft Authenticator, explicitly excluding SMS and TOTP
- Align with specific regulatory requirements such as GDPR, DORA, and NIS2
- Apply different controls per application or user risk tier

A custom Authentication Strength is the correct pattern when built-in options are either too broad or too restrictive for your environment.

### Why FIDO2 and not just MFA

Traditional MFA methods — SMS, TOTP, and push notifications — are vulnerable to Adversary-in-the-Middle (AiTM) attacks. Tools like Evilginx can proxy a real login session and intercept the MFA token in real time. The user sees nothing unusual.

```
[User] -> [Phishing proxy (Evilginx)] -> [Legitimate site]
                  |
                  intercepts MFA token in real time
```

A FIDO2 Passkey cryptographically binds authentication to the legitimate domain using the WebAuthn protocol. A phishing site cannot complete the handshake even with the user's password, because the passkey challenge is domain-scoped and the attacker's domain does not match.

| MFA type | Phishing-resistant | AiTM-resistant |
|---|---|---|
| SMS OTP | No | No |
| TOTP (Authenticator app code) | No | No |
| Push notification | No | No |
| Passkey (FIDO2) | Yes | Yes |

This distinction matters in client assessments and security architecture reviews. Checking "MFA enabled" on a compliance checklist is not the same as being protected against modern credential attacks.

---

## Step-by-step reference

### Part 1 — MFA Registration Policy

**Path:** entra.microsoft.com > Protection > Identity Protection > MFA Registration Policy

1. Sign in to `https://entra.microsoft.com` as Administrator.
2. Navigate to **Identity > Protection > Identity Protection**.
3. Select **Multifactor authentication registration policy**.
4. Under Assignments, select **All users**, then configure:
   - Include: Select individuals and groups > `Delia Dennis`
5. Select the **Exclude** tab and add the Administrator account used for the lab (break-glass protection).
6. Set the policy to **Enabled**.
7. Select **Save**.

**Result:** Delia Dennis is prompted to register Microsoft Authenticator on next login. She is not blocked from accessing resources — the registration is a prerequisite step, not an enforcement gate.

**Why exclude the administrator account:** Break-glass and emergency access accounts should always be excluded from MFA registration policies and restrictive Conditional Access policies. If a misconfiguration locks out all standard admin accounts, the break-glass account must remain accessible. This is a standard security architecture requirement, not optional.

---

### Part 2 — Test registration (as Delia Dennis)

1. Open a new InPrivate browser window.
2. Navigate to `https://entra.microsoft.com` and sign in as Delia Dennis.
3. When prompted with "More information required," select **Next**.
4. Follow the Microsoft Authenticator setup flow:
   - Install the Authenticator app on your phone if not already installed
   - Select **+ > Work or school account > Scan QR code**
   - Scan the QR code displayed on screen
5. Approve the test notification on the phone.
6. Select **Done**.

**Verification:** Sign out and sign back in as Delia Dennis. Confirm that no MFA prompt appears at this stage — registration is complete but enforcement is not yet active.

---

### Part 3 — Conditional Access policy requiring MFA for admin portals

**Path:** entra.microsoft.com > Protection > Conditional Access > + New policy

1. Sign back in as Administrator.
2. Navigate to **Identity > Protection > Conditional Access**.
3. Select **+ New policy**.
4. Configure the policy:

```
Name:          Require MFA for portals
Users:         Include: Delia Dennis
               Exclude: Administrator account
Target:        Select resources > Microsoft Admin Portals
Grant:         Require multifactor authentication
               Require all selected controls
Status:        On
```

5. Select **Create**.

**Verification:**

- Open InPrivate window, sign in as Delia Dennis, navigate to `https://www.office.com` — no MFA prompt (not in scope)
- Navigate to `https://entra.microsoft.com` — MFA is required (Microsoft Admin Portals matched)

This confirms Conditional Access enforces per target resource, not per URL pattern. The same user accessing a different application is not affected by this policy.

---

### Part 4 — Enable Passkey (FIDO2) authentication method

**Path:** entra.microsoft.com > Protection > Authentication methods > Policies > Passkey (FIDO2)

1. Navigate to **Identity > Protection > Authentication methods**.
2. Select **Policies**.
3. Select **Passkey (FIDO2)**.
4. Set the slider to **Enable**.
5. Select **Save**.

**Result:** Passkey (FIDO2) is now available as an authentication method in the tenant. It is not yet required — that is enforced through the Authentication Strength in the next step.

---

### Part 5 — Create a custom Authentication Strength

**Path:** entra.microsoft.com > Protection > Authentication methods > Authentication strengths > + New authentication strength

1. Navigate to **Identity > Protection > Authentication methods**.
2. Select **Authentication strengths**.
3. Select **+ New authentication strength**.
4. Configure:

```
Name:         SC500 phishing resistant MFA
Description:  Forces login with phishing-resistant MFA — lab created
Methods:      Passkeys (FIDO2)
              Microsoft Authenticator (under Advanced options)
```

5. Select **Next**, review the configuration, then select **Create**.

**Note:** If the new Authentication Strength does not appear in the Conditional Access dropdown immediately, sign out and back in. Propagation can take a few minutes.

---

### Part 6 — Update Conditional Access policy to require phishing-resistant MFA

**Path:** entra.microsoft.com > Protection > Conditional Access > Require MFA for portals > Edit

1. Open the **Require MFA for portals** policy created in Part 3.
2. Under **Access controls > Grant**, select the current grant control.
3. Uncheck **Require multifactor authentication**.
4. Check **Require authentication strength**.
5. From the dropdown, select **SC500 phishing resistant MFA**.
6. Confirm **Require all selected controls** is selected.
7. Select **Select**, then **Save**.

**Result:** The policy now requires phishing-resistant MFA specifically, not just any MFA method. SMS and TOTP codes will no longer satisfy this policy — only FIDO2 Passkey or the specific Authenticator methods defined in the strength.

---

### Part 7 — Register a Passkey (FIDO2) for Delia Dennis

**Important:** Passkey registration via Microsoft Authenticator requires a Bluetooth connection between the phone and the computer. This step must be performed on a physical PC browser, not inside a virtual machine.

1. Open an InPrivate browser window on the physical PC.
2. Navigate to `https://portal.azure.com` and sign in as Delia Dennis.
3. When prompted to set up a Passkey, select **Other options**.
4. Select **Having trouble?**
5. Select **Create your passkey a different way**.
6. Choose **iPhone** or **Android** from the list.
7. Follow the on-screen instructions to enable Microsoft Authenticator as a passkey provider on the phone.
8. Select **Continue**, then **I'm ready to proceed**.
9. On the phone, open the camera and scan the QR code displayed on the PC screen.
10. This establishes the Bluetooth-secured link between the devices.
11. On the phone, select **Save a Passkey**, then **Continue**.
12. Enter the Microsoft Authenticator passcode when prompted.
13. On the PC, select **OK**.
14. On the Name your passkey screen, select **Next**.

**Result:** The Passkey is saved to Microsoft Authenticator on the phone and linked to Delia Dennis's account.

**Known lab issue:** The "Name your passkey" screen can hang in some lab environments. If this happens, close the browser — the passkey creation and save process has already completed.

---

### Part 8 — Verify login with Passkey (FIDO2)

1. Open a new InPrivate browser window on the physical PC (Bluetooth must remain active).
2. Navigate to `https://portal.azure.com`.
3. Enter Delia Dennis's username and password.
4. On the **Sign in with your passkey** screen, select **iPhone, iPad, or Android**.
5. Select **Next**.
6. On the phone, open the camera and scan the QR code on the screen.
7. Select **Sign in with a passkey**.
8. Select **Continue**.
9. Authenticate using the Authenticator app biometric or PIN.
10. On the PC, select **Yes** on the Stay signed in dialog.

**Result:** Delia Dennis is signed in using phishing-resistant FIDO2 authentication. No password was transmitted over the network in a form that could be intercepted or replayed.

---

## Technical notes

- Bluetooth is required for Passkey registration and login on this flow. Virtual machines do not support Bluetooth — perform these steps on the physical PC browser.
- New Authentication Strengths can take several minutes to propagate to the Conditional Access policy dropdown. Sign out and back in if it does not appear.
- Always exclude administrator and break-glass accounts from restrictive MFA and Conditional Access policies to prevent accidental tenant lockout.
- The Conditional Access policy targets Microsoft Admin Portals as the resource, not individual URLs. This is the correct scope for admin portal protection — it covers Entra admin center, Azure portal, and Microsoft 365 admin center under a single policy target.

---

## Security notes

**MITRE ATT&CK relevance:**  
T1111 — MFA Interception: AiTM attacks intercepting SMS and TOTP tokens. FIDO2 Passkeys eliminate this attack vector by design.  
T1078 — Valid Accounts: Phishing-resistant MFA prevents attackers from using stolen credentials with intercepted MFA tokens to establish valid sessions.

**Regulatory alignment:**  
DORA (Digital Operational Resilience Act) and NIS2 both require strong authentication controls for privileged access to critical systems. Phishing-resistant MFA satisfies the "strong authentication" requirement in both frameworks more completely than SMS or TOTP.

**Authentication Strength vs Security Defaults:**  
Security Defaults enable basic MFA for all users but accept any MFA method including SMS. Custom Authentication Strengths allow you to reject weak second factors. For environments with regulatory requirements or elevated risk profiles, Security Defaults alone are insufficient.

---

