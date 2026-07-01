# Lab: Configure Privileged Identity Management (PIM)

**Series:** AZ-500 / SC-500 Hands-on Labs  
**Platform:** Microsoft Entra ID — Identity Governance  
**Duration:** ~45 minutes  
**Level:** 300 (Intermediate / Advanced)

---

## Overview

Privileged Identity Management (PIM) is a Microsoft Entra ID service that enforces just-in-time (JIT) privileged access to Azure and Microsoft Entra roles. Instead of granting permanent admin access — which creates a persistent attack surface — PIM requires users to request and activate elevated access for a limited time window, with optional approval and justification requirements.

The core security principle: standing permissions that never expire are a least-privilege failure. A compromised account with permanent Global Admin access gives an attacker immediate, persistent, broad access. PIM eliminates standing permissions by making elevation time-bound, auditable, and approval-gated.

---

## What this lab covers

- Assigning a PIM-eligible role (Conditional Access Administrator) to a user
- Configuring activation settings: time limit, justification requirement, and named approver
- Requesting role activation as a non-privileged user
- Approving the activation request as an approver account
- Verifying that the activated role grants the expected permissions
- Manually deactivating the role to close the JIT access window
- Enabling a system-assigned managed identity on an Azure App Service

---

## Key concepts before starting

### Eligible vs Active assignment

| Assignment type | What it means | Access granted? |
|---|---|---|
| Eligible | User can request activation | No — must activate first |
| Active | Role is currently activated | Yes — until expiry or deactivation |

An eligible assignment is not access. It is permission to request access. This distinction matters in audits and incident response — "eligible assignments" in a tenant are not the same as "active privileged accounts."

### Why approval workflows matter

Without an approval gate, a user with an eligible assignment can self-activate at any time with no oversight. Adding a named approver means every elevation is reviewed by a human before access is granted. Combined with justification requirements, this creates an audit trail: who requested access, why, who approved it, and when it expired.

### Managed identity vs service principal

A managed identity is an automatically managed identity in Entra ID assigned to an Azure resource (VM, App Service, Function App). The platform handles credential rotation — no passwords or secrets to store or rotate manually. System-assigned managed identities are tied to the lifecycle of the resource: if the resource is deleted, the identity is deleted. This is the recommended pattern for workload identity in Azure.

---

## Step-by-step reference

### Part 1 — Assign a PIM-eligible role

**Path:** entra.microsoft.com > Identity governance > Privileged Identity Management > Microsoft Entra roles > Assignments > Add assignments

1. Sign in to `https://entra.microsoft.com` as Administrator.
2. Navigate to **Identity governance > Privileged Identity Management**.
3. Select **Microsoft Entra roles** under Manage.
4. Select **Assignments**, then **Add assignments**.
5. Configure the assignment:
   - Role: `Conditional Access Administrator`
   - Members: `Adele Vance`
6. Select **Next**.
7. Set Assignment type to **Eligible**.
8. Select **Assign**.

**Verification:** On the Assignments page, Adele Vance appears under the **Eligible assignments** tab with the Conditional Access Administrator role. No access is active at this point.

---

### Part 2 — Configure activation settings

**Path:** PIM > Microsoft Entra roles > Settings > Conditional Access Administrator > Edit

1. In PIM, select **Settings** under Manage.
2. Find and select **Conditional Access Administrator**.
3. Select **Edit**.
4. On the **Activation** tab, set:
   - Activation maximum duration: `1 hour`
   - On activation, require: `Justification`
   - Require approval to activate: `Enabled`
5. Under Select approvers, select **+ Select members**.
6. Search for and select **MOD Administrator**.
7. Select **Update**.

**Verification:** Role settings page shows:
- Maximum activation duration: 1 hour
- Approval required: Yes
- Approver: MOD Administrator

**Why 1 hour:** The activation window should be the minimum time needed to complete the task. A Security Engineer reviewing CA policies does not need 8-hour access — 1 hour enforces the just-enough-time principle alongside just-enough-access.

---

### Part 3 — Request role activation (as Adele Vance)

**Path:** PIM > My roles > Microsoft Entra roles > Eligible assignments > Activate

1. Open a new InPrivate browser window.
2. Navigate to `https://entra.microsoft.com` and sign in as **Adele Vance**.
3. Navigate to **Identity governance > Privileged Identity Management**.
4. Under Tasks, select **My roles**.
5. Select the **Microsoft Entra roles** tab.
6. Under Eligible assignments, find **Conditional Access Administrator** and select **Activate**.
7. Configure the activation:
   - Duration: `1 hour`
   - Justification: `Reviewing and updating Conditional Access policies as part of a scheduled security review.`
8. Select **Activate**.

**Result:** A confirmation appears that the request is pending approval. The role is not yet active. Leave this window open.

**Note on justification text:** In a real environment, the justification should reference a specific ticket, incident, or change request number. This creates traceability between access elevation and the business reason for it — critical for compliance audits and incident investigations.

---

### Part 4 — Approve the activation request (as MOD Administrator)

**Path:** PIM > Approve requests > Microsoft Entra roles

1. Return to the primary browser window (MOD Administrator).
2. Navigate to `https://entra.microsoft.com`.
3. Navigate to **Identity governance > Privileged Identity Management**.
4. Under Tasks, select **Approve requests**.
5. Select **Microsoft Entra roles**.
6. Find the pending request from **Adele Vance** for **Conditional Access Administrator**.
7. Check the box next to the request and select **Approve**.
8. Enter justification: `Approved for scheduled security review task.`
9. Select **Submit**.

**Result:** An approval confirmation appears. The role is now active for Adele Vance.

---

### Part 5 — Verify the activated role

**Path:** PIM > My roles > Microsoft Entra roles > Active assignments

1. Return to the Adele Vance browser window and refresh.
2. In PIM, select **My roles > Microsoft Entra roles**.
3. Select the **Active assignments** tab.
4. Confirm **Conditional Access Administrator** appears with:
   - Status: Active
   - Expiration: approximately 1 hour from activation time

**Test in Conditional Access:**

1. In the left navigation, find **Conditional Access** under the Entra ID section.
2. Select **+ Create new policy**.
3. Confirm the policy creation pane opens without errors.
4. Select **X** to close without saving — this step verifies access only.

If the pane opens, the role is active and granting the correct permissions. Without this role active, the option would be unavailable or throw an access error.

---

### Part 6 — Deactivate the role

**Path:** PIM > My roles > Microsoft Entra roles > Active assignments > Deactivate

1. Navigate to **Privileged Identity Management > My roles > Microsoft Entra roles**.
2. Select the **Active assignments** tab.
3. Find **Conditional Access Administrator** and select **Deactivate**.
4. In the confirmation dialog, select **Deactivate** again.

**Verification:** The role no longer appears under Active assignments. It has returned to Eligible assignments only.

This is the correct operational pattern: release access as soon as the task is complete, not when the time window expires. A 1-hour window that gets manually closed after 15 minutes is better than waiting for automatic expiry — it reduces the window of exposure if the session is compromised.

---

### Part 7 — Enable system-assigned managed identity on App Service

**Path:** Azure portal > App Service > Settings > Identity > System assigned

1. Navigate to `https://portal.azure.com`.
2. Find the pre-provisioned App Service in the resource group.
3. In the left menu, under Settings, select **Identity**.
4. On the **System assigned** tab, set Status to **On**.
5. Select **Save** and confirm the dialog.

**Result:** Azure creates a service principal in Entra ID tied to this App Service. The identity can now be assigned RBAC roles (for example, Key Vault Secrets User) without storing any credentials in application code or configuration.

**Security implication:** Applications that authenticate using managed identity cannot have their credentials stolen because there are no credentials to steal. This eliminates an entire credential-based attack class relevant to cloud workloads.

---

## Cleanup

The lab environment resets automatically. To clean up PIM assignments manually before session end:

1. Sign in as Global Administrator.
2. Navigate to PIM > Microsoft Entra roles > Assignments.
3. Find the Conditional Access Administrator eligible assignment for Adele Vance.
4. Select **Remove** and confirm.

---

## Security notes

**MITRE ATT&CK relevance:**  
T1078 — Valid Accounts: PIM directly mitigates the impact of compromised privileged accounts by ensuring those accounts hold no standing permissions.  
T1098 — Account Manipulation: PIM audit logs detect unexpected role activation patterns.

**Licensing requirement:**  
PIM requires Microsoft Entra ID P2 or the Microsoft Entra ID Governance add-on. It is not available in P1 or Free tiers. This is a common gap in client environments — organisations with M365 E3 (which includes P1 only) do not have PIM available without an upgrade.

**Break-glass accounts:**  
Emergency access accounts should be excluded from PIM-managed roles and held as permanently active assignments. A misconfigured PIM workflow that locks out all administrators with no break-glass account is a tenant lockout scenario. Always maintain at least two emergency access accounts outside of PIM scope.

**Approval workflow failure mode:**  
If the named approver is unavailable and no backup approver is configured, activation requests queue indefinitely. In production, always configure at least two approvers per role to prevent access delays during incidents.

---

