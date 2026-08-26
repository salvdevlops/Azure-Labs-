# Azure RBAC and Subscription Management

This lab covers role-based access control (RBAC) in Azure — controlling what people can actually do, scoped to specific resources — plus using management groups to make subscriptions easier to manage as a group instead of one at a time.

## Overview

In this lab I built a management group, assigned a built-in role to a group, created a custom RBAC role by cloning and trimming permissions off a built-in one, and checked the Activity Log to see role assignment events. The point is to get hands-on with least-privilege access control instead of just handing out Owner/Contributor to everyone.

## Lab Scenario

The goal was to make subscription management easier for a company with multiple Azure subscriptions by:

- Creating one management group that all the subscriptions roll up into
- Giving a Help Desk group permission to submit support requests across every subscription in that group — but scoped down to just VM management and creating support tickets, nothing more (specifically not the ability to register Azure resource providers)

## Environment

- Azure subscription
- Microsoft Entra ID (Azure AD)

## Diagram

`[add diagram later]`

---

## Task 1: Implement Management Groups

Management groups let you organize subscriptions logically so RBAC and Azure Policy can be assigned once and inherited down, instead of having to set things up on every subscription individually. In this scenario, the whole Help Desk needs to submit support requests across all subscriptions, so it makes more sense to manage that at the management group level than subscription-by-subscription.

First, I searched for and opened **Microsoft Entra ID**, then went to the **Manage** blade and selected **Properties** to look at the **Access management for Azure resources** section — this is where you can manage access to every Azure subscription and management group in the tenant.

From there I searched for and selected **Management groups**, then on the **Management groups** blade hit **+ Create** and set it up like this:

| Setting | Value |
|---|---|
| Management group ID | salnoct-mg1 *(must be unique in the directory)* |
| Management group display name | salnoct-mg1 |

<img width="786" height="173" alt="image" src="https://github.com/user-attachments/assets/09870664-ccbd-46e2-989f-527e7fc53684" />

After hitting Submit and refreshing the page, **salnoct-mg1** showed up (took about a minute):

<img width="1409" height="168" alt="image" src="https://github.com/user-attachments/assets/7199ad4f-8161-4569-815d-8e8c1378ffd5" />

There's also a root management group built into the hierarchy by default — every management group and subscription eventually rolls up into it, which is how global policies and role assignments can be applied at the directory level. From here, the next step would normally be moving actual subscriptions into the group or mgmnt dependings on your needs.

---

## Task 2: Review and Assign a Built-In Azure Role

Azure comes with a ton of built-in roles already defined, so next I assigned the **Virtual Machine Contributor** role to a Help Desk group at the management group level. (If you don't already have a Help Desk group set up in Entra ID, you'll need to create one first.)

I selected the **salnoct-mg1** management group, went to **Access control (IAM)**, then the **Roles** tab to scroll through what's available. **Owner**, **Contributor**, and **Reader** are the ones you'll see referenced the most, but clicking into any role shows its permissions, the underlying JSON, and current assignments.

<img width="1893" height="816" alt="image" src="https://github.com/user-attachments/assets/101d767f-3f14-47cb-af8d-e020c4716344" />

From **+ Add**, I selected **Add role assignment**, then searched for and picked **Virtual Machine Contributor**. This role lets you manage VMs but doesn't give access to the OS inside them or to the virtual network/storage account they're connected to — a good fit for what Help Desk actually needs to do.

<img width="1893" height="613" alt="image" src="https://github.com/user-attachments/assets/2e239c6f-0b01-44c5-9311-32e92bb560ed" />

On the **Members** tab, I searched for and selected the **helpdesk** group, clicked through **Conditions** without changing anything, and left **Assignment type** on the default (**Eligible**, default time-bound settings). Then **Review + assign**:

<img width="1911" height="364" alt="image" src="https://github.com/user-attachments/assets/e5eef118-697e-46c2-83a8-600039f9f085" />

Back on **Access control (IAM)** → **Role assignments**, the helpdesk group now shows the **Virtual Machine Contributor** role:

<img width="1422" height="91" alt="image" src="https://github.com/user-attachments/assets/1b9ad985-5590-48ef-a8a4-be1899303280" />

Worth noting — best practice is to assign roles to groups instead of individuals, since it's a lot easier to manage as people join or leave. Also, if you've already got Owner somewhere up the chain, this assignment won't actually change anything for you personally — Owner already covers everything VM Contributor does.

---

## Task 3: Create a Custom RBAC Role

Built-in roles are sometimes broader than what you actually need, so here I cloned an existing role and stripped out a permission to tighten it up — basic least-privilege stuff.

Still on the management group's **Access control (IAM)** blade, I went **+ Add** → **Add custom role** and filled in:

| Setting | Value |
|---|---|
| Custom role name | Custom Support Request |
| Description | A custom contributor role for support requests. |

For **Baseline permissions**, I chose **Clone a role** and picked **Support Request Contributor**:

<img width="609" height="500" alt="image" src="https://github.com/user-attachments/assets/fc8169cc-7b7c-4061-adfa-cc95ae49ef60" />

On the **Permissions** tab, I hit **+ Exclude permissions**, searched `.Support` in the resource provider field, and selected **Microsoft.Support**. Then I checked the box for **Other: Registers Support Resource Provider** and added it — that permission now shows up as a `NotAction` on the role. Help Desk doesn't need the ability to register resource providers, so it gets pulled out of the cloned role.

I confirmed the management group was listed under **Assignable scopes**, then checked over the JSON to make sure the `Actions`, `NotActions`, and `AssignableScopes` actually reflected the change:

<img width="1346" height="709" alt="image" src="https://github.com/user-attachments/assets/4e6ef3a9-f40b-4190-a206-6bb346be4c9a" />

**Review + Create**, then **Create**:

<img width="584" height="517" alt="image" src="https://github.com/user-attachments/assets/c34bc10d-b819-4eda-80a1-3c95d5a8dcb2" />

---

## Task 4: Monitor Role Assignments with the Activity Log

Last step — the Activity Log tracks subscription-level events, including role assignment changes, so it's worth knowing how to check it.

I found the **salnoct-mg1** resource and opened **Activity log**, then looked through the recent activity for the role assignment events from earlier in the lab. The log can be filtered down if you're hunting for something specific:

<img width="1622" height="242" alt="image" src="https://github.com/user-attachments/assets/c3c26cb4-2359-4d7b-8d54-42bd1515e819" />

---

## Cleanup

If you're running this on your own subscription, clean up the lab resources afterward so you're not paying for anything you don't need. Any of these work:

- Azure portal: select the management group, select **Delete**, confirm with **Yes**
- PowerShell: `Remove-AzManagementGroup -GroupName salnoct-mg1`
- CLI: `az account management-group delete --name salnoct-mg1`

---

## Key Takeaways

- **Management groups** organize subscriptions logically so RBAC/Policy can be applied once and inherited down.
- There's a built-in **root management group** that every other management group and subscription eventually rolls up into.
- Azure has a big set of **built-in roles** you can assign to control access.
- You can **clone and customize** roles when a built-in one isn't scoped tightly enough.
- Roles are defined as JSON with `Actions`, `NotActions`, and `AssignableScopes`.
- The **Activity Log** is how you monitor and audit role assignment changes.
