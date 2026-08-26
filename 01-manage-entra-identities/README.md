# Microsoft Entra ID: Users and Groups

This lab is about the basics of identity management in Microsoft Entra ID (formerly Azure AD) — creating user accounts, inviting an external/guest user, and putting both into a group.

## Overview

Users and groups are the building blocks of any identity setup. In this lab I created a regular Entra ID user, invited a guest user, and added both to a security group — basically simulating how a company would onboard a couple new engineers into a lab environment.

## Lab Scenario

A company is setting up a new lab environment for testing apps/services before they go to production. A couple engineers are getting hired to manage it (including the VMs), and they need to authenticate through Microsoft Entra ID. My job: set up the users and groups so it's easy to manage as more people get added later.

## Environment

- Azure subscription
- Microsoft Entra ID (Azure AD)

## Diagram

<img width="530" height="378" alt="image" src="https://github.com/user-attachments/assets/d6447533-a316-4b35-b146-8a1e3a8fc1d6" />







## Skills Demonstrated

- Creating and configuring Microsoft Entra ID user accounts
- Inviting and configuring external (guest) users
- Creating security groups and managing membership
- Understanding tenants, static vs. dynamic group membership, and Entra ID licensing implications

---

## Task 1: Create and Configure User Accounts

User accounts hold info like name, department, location, and contact details. Here I created a regular Entra ID user and invited a guest user, then set them both up with the same job/department info.

### Step 1: Open Microsoft Entra ID

In the Azure portal, search for **Microsoft Entra ID** and select it. Entra ID is Azure's cloud-based identity and access management service.

### Step 2: Review tenant information

From the **Overview** blade, select the **Manage tenants** tab.

> A tenant is a dedicated instance of Microsoft Entra ID that holds an organization's accounts and groups. Depending on your setup, you can create additional tenants and switch between them.

![Manage tenants view](https://github.com/user-attachments/assets/3977c45f-bbf4-4b1a-b92d-7f35fdfdcb51)

### Step 3: Create a new user

In the **Manage** blade, select **Users**, then from the **New user** dropdown select **Create new user**.

> This opens a new browser tab — select **New user** again once it loads.

![Create new user menu](https://github.com/user-attachments/assets/425a22f2-a63a-4caf-abf4-ece29968644a)

### Step 4: Set the user's basic info

Name the user anything you'd like.

![New user basic info](https://github.com/user-attachments/assets/41aeb488-54a7-478b-a7cb-f335e0075e81)

### Step 5: Fill in the Properties tab

| Setting | Value |
|---|---|
| Job title | IT Lab Administrator |
| Department | IT |
| Usage location | United States |

### Step 6: Review and create

Select **Review + create**, confirm the details, and create the user. Your summary should resemble the screenshot below (your specific values will differ based on what you entered).

![Review and create user](https://github.com/user-attachments/assets/e013f04f-4c06-45ee-80c3-56f5987e42ee)

### Step 7: Confirm the user was created

Refresh the **Users** page and confirm the new account appears in the list.

![New user appears in users list](https://github.com/user-attachments/assets/9e947d93-fb32-4f3d-9f63-80a925ac3dc0)

### Step 8: Invite an external user

From the **New user** dropdown, select **Invite an external user**.

**Settings:**

| Setting | Value |
|---|---|
| Email | Your email address |
| Display name | Your name |
| Send invite message | Checked |
| Message | Hey, you're invited to check out the IT lab environment I'm setting up in Azure. Welcome aboard! |

### Step 9: Configure the guest user's properties

Move to the **Properties** tab and fill in the same fields used for the internal user:

| Setting | Value |
|---|---|
| Job title | IT Lab Administrator |
| Department | IT |
| Usage location | United States |

### Step 10: Send the invite

Select **Review + invite**, then **Invite**.

### Step 11: Confirm the guest user was created

Refresh the page and confirm the invited (guest) user now appears in the list. You should also receive the invitation email shortly.

<img width="617" height="715" alt="image" src="https://github.com/user-attachments/assets/43ea5d79-bb2d-4ca8-be7d-4c4192d6e0b2" />


> **Note:** Creating accounts one-by-one like this obviously doesn't scale. In a real org you'd bulk-create users — CSV import, PowerShell/Graph API scripts, or syncing from an HR system through Entra Connect.

---

## Task 2: Create Groups and Add Members

Groups can hold users or devices, and there's two ways to manage membership:

- **Static** — you manually add/remove members
- **Dynamic** — membership updates on its own based on a rule, like job title (needs Entra ID Premium P1 or P2)

### Step 1: Open Groups

In the Azure portal, search for and select **Microsoft Entra ID**, then in the **Manage** blade select **Groups**.

### Step 2: Review group settings

Take a quick look at the group settings in the left pane:

- **Expiration** — sets how long a group lasts before the owner has to renew it
- **Naming policy** — blocks certain words and can force a prefix/suffix on group names

`[ADD SCREENSHOT HERE — group settings pane]`

### Step 3: Create a new group

From **All groups**, select **+ New group**.

| Setting | Value |
|---|---|
| Group type | Security |
| Group name | IT Lab Administrators |
| Group description | Administrators that manage the IT lab |
| Membership type | Assigned |

> Dynamic membership requires an Entra ID Premium P1 or P2 license. If your tenant has one, additional membership type options will appear in the dropdown.

<img width="799" height="514" alt="image" src="https://github.com/user-attachments/assets/37cd8b67-1805-4d62-8daf-3b25a08e88ee" />


### Step 4: Add an owner

Select **No owners selected**, then search for and select yourself as the owner.

> A group can have more than one owner.

<img width="739" height="504" alt="image" src="https://github.com/user-attachments/assets/6675c6b9-7350-4592-a98c-966491afd65f" />


### Step 5: Add members

Select **No members selected**. In the **Add members** pane, search for and select the user account and guest user created in Task 1, then add both to the group.

<img width="1045" height="403" alt="image" src="https://github.com/user-attachments/assets/539da38d-14fc-4ebc-9bdf-47dbc2be4de2" />


### Step 6: Create the group

Select **Create** to deploy the group.

### Step 7: Confirm the group was created

Refresh the page and confirm the new group appears in the group list.

<img width="414" height="270" alt="image" src="https://github.com/user-attachments/assets/2cc73f92-bf02-4dae-90a2-0a91c9a23146" />


### Step 8: Review group membership

Open the new group and review its **Members** and **Owners** tabs to confirm everything was added correctly.

<img width="418" height="301" alt="image" src="https://github.com/user-attachments/assets/6a38dbdf-5e5e-48f7-a7e5-c24fbbe88be9" />      <img width="421" height="305" alt="image" src="https://github.com/user-attachments/assets/aebbb80e-e15c-49ae-b43c-eba8938d7cce" />


> **Note:** Doing this manually is fine for one group, but it doesn't hold up at scale. A real setup would need clear naming conventions, defined ownership, and either a process or automation for adding/removing members as people join or change roles.

---

## Key Takeaways

- A **tenant** is basically an org's own dedicated instance of Microsoft cloud services — covers both internal and guest users.
- Entra ID has **user** and **guest** accounts, each with access scoped to what they actually need.
- **Groups** combine related users or devices, and come in two types: **Security** and **Microsoft 365**.
- Group membership can be **static** (manual) or **dynamic** (rule-based, needs Premium licensing).







