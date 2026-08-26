# Azure Policy and Governance

This lab is about implementing governance using Azure Policy — enforcing standards across resources instead of relying on people remembering to follow the rules — plus using resource tags and resource locks to keep things organized and protected.

## Overview

In this lab I tagged a resource group, built an Azure Policy to require that tag on any new resources, tested what happens when the policy gets violated, then switched to a policy that auto-inherits the tag onto child resources so I didn't have to tag everything by hand. Last step was setting up a resource lock to stop accidental deletion.

## Lab Scenario

The company's Azure footprint had grown a lot over the past year, and an audit turned up a bunch of resources with no defined owner, project, or cost center attached. To clean that up, the plan was to:

- Tag resources with key metadata (starting with a Cost Center tag)
- Use Azure Policy to require that tag on any new resources going forward
- Use Azure Policy to automatically apply the tag to existing/child resources instead of doing it manually
- Use resource locks to protect resources from getting deleted by accident

## Environment

- Azure subscription


## Diagram

`[add diagram later]`

---

## Task 1: Assign Tags via the Azure Portal

Tags matter more than they sound like they should — they're how you actually identify who owns what, when something's supposed to be decommissioned, which team to contact, etc. Without them you end up with exactly the mess this org ran into: a pile of resources nobody can account for.

I signed into the Azure portal, searched for **Resource groups**, and created a new one:

| Setting | Value |
|---|---|
| Subscription | my subscription |
| Resource group name | rg-lab |
| Region | East US |

On the **Tags** tab during creation, I added:

| Setting | Value |
|---|---|
| Name | Cost Center |
| Value | 000 |

Then **Review + Create** → **Create**.

<img width="429" height="403" alt="image" src="https://github.com/user-attachments/assets/92318b6c-f22f-4b9b-b477-895527bf33a3" />

---

## Task 2: Enforce Tagging via an Azure Policy

Tagging things manually only works until someone forgets. This task was about using a built-in policy to actually enforce it — if a new resource doesn't have the right tag, Azure blocks it from being created at all.

I searched for and opened **Policy**, went to **Definitions** under the Authoring section, and looked through the built-in policy definitions available (there's a lot — you can also just search for what you need). I found **Require a tag and its value on resources** and read through its definition before assigning it.

<img width="1896" height="822" alt="image" src="https://github.com/user-attachments/assets/169b74c6-cc19-4e20-91a7-4862a65940f9" />

From **Assign policy**, I set the scope to my subscription and the **rg-lab** resource group specifically — policies can be assigned at the management group, subscription, or resource group level, and you can carve out exclusions too, but here I wanted the tag enforced on everything in that one resource group.

For the assignment basics:

| Setting | Value |
|---|---|
| Assignment name | Require Cost Center tag and its value on resources |
| Description | Require Cost Center tag and its value on all resources in the resource group |
| Policy enforcement | Enabled |

Parameters:

| Setting | Value |
|---|---|
| Tag Name | Cost Center |
| Tag Value | 000 |

I left the Remediation and Managed Identity tabs as default (no managed identity needed for this one), then **Review + Create** → **Create**.

To actually test it, I waited a few minutes for the policy to take effect, then tried creating a Storage Account in **rg-lab** without adding the tag:

| Setting | Value |
|---|---|
| Resource group | rg-lab|
| Storage account name | any unique lowercase name, 3–24 characters |

Hit **Review**, then **Create** — and got a **Validation failed** message, which is exactly what should happen. The error confirmed the deployment got blocked by the policy because the storage account was missing the required Cost Center tag.

<img width="581" height="536" alt="image" src="https://github.com/user-attachments/assets/0bba1913-de3d-42e7-b5e9-40e37ccbb4da" />

> You can dig into the **Raw Error** tab for more detail, including the actual policy name that triggered the block.

---

## Task 3: Apply Tagging via an Azure Policy

Blocking untagged resources is one thing, but I also wanted existing/child resources to automatically pick up the Cost Center tag from their parent resource group instead of tagging each one by hand. So this task swaps the previous policy out for one that does inheritance instead of just enforcement.

First I deleted the old policy assignment — back in **Policy** → **Assignments**, found the **Require a tag and its value on resources** assignment, hit the 3 dots on the right, and selected **Delete assignment** (confirmed with Yes).

Then I assigned a new policy with the same scope (subscription + **rg-lab**), this time picking **Inherit a tag from the resource group if missing** as the policy definition.

Basics:

| Setting | Value |
|---|---|
| Assignment name | Inherit the Cost Center tag and its value 000 from the resource group if missing |
| Description | Inherit the Cost Center tag and its value 000 from the resource group if missing |
| Policy enforcement | Enabled |

Parameters:

| Setting | Value |
|---|---|
| Tag Name | Cost Center |

On the **Remediation** tab:

| Setting | Value |
|---|---|
| Create a remediation task | Enabled |
| Policy to remediate | Inherit a tag from the resource group if missing |

This policy uses the **Modify** effect, which is why it needs a managed identity — Azure created one automatically for the remediation task.

**Review + Create** → **Create**, then waited again for the policy to kick in.

<img width="1823" height="864" alt="image" src="https://github.com/user-attachments/assets/b47a54d3-7507-4b1b-ad6e-73949cb39a06" />

To test it, I created another storage account in the same resource group — same as before, but this time *without* manually adding the tag:

| Setting | Value |
|---|---|
| Storage account name | any unique lowercase name, 3–24 characters |

This time validation actually passed and the storage account got created. Once I went to the resource and checked the **Tags** blade, the **Cost Center** tag with value **000** was already sitting there — inherited automatically from the resource group, no manual tagging needed.

<img width="1890" height="656" alt="image" src="https://github.com/user-attachments/assets/3b2d80e2-3a52-4f37-96cd-c67e79d277a2" />

> Side note: searching **Tags** directly in the portal lets you pull up every resource carrying a specific tag, which is handy for reporting.

---

## Task 4: Configure and Test Resource Locks

Last piece — resource locks. These stop a resource (or resource group) from being deleted or modified, even by someone who technically has permission to do it. Good safety net for stuff you really don't want gone by accident.

I went to my resource group, opened **Locks** under Settings, and added one:

| Setting | Value |
|---|---|
| Lock name | rg-lock |
| Lock type | Delete *(there's also a Read-only option)* |

Then I went back to the resource group's Overview page and tried to actually delete it — typed `rg-lab` into the confirmation box, got the "this is permanent" warning, and clicked through both delete confirmations.

It failed — got a notification saying the deletion was denied, exactly because of the lock.

<img width="511" height="109" alt="image" src="https://github.com/user-attachments/assets/688a8056-09ec-4646-9dc2-dfb2044ecb70" />


> If I actually wanted to delete the resource group, I'd need to remove the lock first.

---

## Cleanup

If running this on a real subscription, clean up the resource group afterward to avoid ongoing cost (remember to remove the lock first if one's still in place):

- Azure portal: select the resource group → **Delete the resource group** → enter the name to confirm → **Delete**
- PowerShell: `Remove-AzResourceGroup -Name resourceGroupName`
- CLI: `az group delete --name resourceGroupName`

---

## Key Takeaways

- **Tags** are key-value metadata that describe a resource — owner, cost center, environment, whatever matters to the org.
- **Azure Policy** enforces conventions on resources. A policy definition lays out a compliance condition and what to do if it's not met.
- **Remediation tasks** bring existing non-compliant resources into line with a Modify or deployIfNotExist policy, instead of requiring manual fixes.
- **Resource locks** (Delete or Read-only) protect resources from accidental changes or deletion — and they override normal user permissions.
- Azure Policy is a **pre-deployment** governance tool. RBAC and resource locks are **post-deployment** protections.
