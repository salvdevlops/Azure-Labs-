# Azure Resource Manager Templates and Bicep

## Overview

In this lab I created a managed disk in the portal, exported it as an ARM template, edited and redeployed that template to create a second disk, then repeated the deployment two more times — once through PowerShell in Cloud Shell, once through the Azure CLI — and finished by deploying a fifth disk using a Bicep file instead of raw JSON. Five disks, five different ways of getting there, same end result.

## Lab Scenario

The team wanted a way to automate and simplify resource deployments — cutting down on admin overhead, reducing human error, and making deployments more consistent instead of everyone doing things slightly differently by hand in the portal.

## Environment

- Azure subscription

## Diagram

`[add diagram later]`

---

## Task 1: Create an Azure Resource Manager Template

First step was creating a basic managed disk in the portal, just so I'd have something real to export as a template afterward. Managed disks are Azure's block-level storage volumes used with VMs — nothing fancy needed here since the point was practicing with templates, not the disk itself.

I searched for **Disks**, hit **Create**, and configured it:

| Setting | Value |
|---|---|
| Subscription | my subscription |
| Resource Group | whitesnake *(created new)* |
| Disk name | whitesnake_disk1 |
| Region | East US |
| Availability zone | No infrastructure redundancy required |
| Source type | None |
| Performance | Standard HDD |
| Size | 32 GiB |

**Review + Create** → **Create**, then once it deployed I selected **Go to resource**.

<img width="528" height="779" alt="image" src="https://github.com/user-attachments/assets/6ca81554-58a7-41db-abf6-392eb22432a1" />

From the **Automation** blade, I selected **Export template** and looked over the generated Template and Parameters files. Downloaded both as JSON — one is the actual resource definition, the other holds the parameter values separately.

<img width="1566" height="755" alt="image" src="https://github.com/user-attachments/assets/6bd34eee-bc09-4a66-8f2e-cfd9cb855e2d" />

> You can export a whole resource group this way too, not just a single resource.

---

## Task 2: Edit and Redeploy the Template

Now that I had a working template, the goal was to tweak it slightly and reuse it to spin up a second disk — basically proving the whole point of templates: deploy once, repeat easily.

In the portal I searched for **Deploy a custom template**. Instead of using one of the built-in Quickstart templates, I picked **Build your own template in the editor** and loaded the `template.json` file I downloaded.

In the editor, I made two changes:

- Renamed `disks_whitesnake_disk1_name` to `disk_name` (appears in two places)
- Changed `whitesnake_disk1` to `whitesnake_disk2` (one place)

Everything else stayed the same — still a Standard disk, still `eastus`, still 32GB.

I saved that, then did the matching edit on the parameters file (**Edit parameters** → **Load file** → `parameters.json`), renaming `disks_whitesnake_disk1_name` to `disk_name` there too so it lined up with the template.

For the deployment itself:

| Setting | Value |
|---|---|
| Subscription | my subscription |
| Resource Group | whitesnake |
| Region | (US) East US |
| Disk_name | whitesnake_disk2 |

**Review + Create** → **Create** → **Go to resource**, and confirmed `whitesnake_disk2` was actually there.

<img width="1911" height="909" alt="image" src="https://github.com/user-attachments/assets/287ccfc4-71b2-4806-8dc4-acfb95ddbc6a" />

Back on the resource group's Overview, I now had two disks. Under **Settings** → **Deployments**, every deployment gets logged with its own Input and Template details — good practice to check the first few template-based deployments before trusting them for anything bigger.

<img width="1582" height="591" alt="image" src="https://github.com/user-attachments/assets/247691b1-5da9-46e1-a946-f210afbf1d43" />

---

## Task 3: Deploy a Template with Azure PowerShell

This task moves into Cloud Shell to deploy the same kind of template, but through PowerShell instead of the portal UI.

I opened Cloud Shell from the top bar (you can also go straight to shell.azure.com) and picked **PowerShell** when prompted.

> Bash tends to feel more natural if you're coming from Linux; PowerShell feels more natural coming from Windows.

Cloud Shell dropped me into an ephemeral PowerShell session — no storage account setup needed.

<img width="1905" height="265" alt="image" src="https://github.com/user-attachments/assets/0fb6ff67-a38e-49cd-9889-f62ff934abed" />

I switched to the classic Cloud Shell view (**Settings** → **Go to classic version**), then used the upload icon to upload both the template and parameters files from my Downloads folder.

Opened the editor (curly-brace icon), found the template JSON in the file pane, and changed the disk name to `whitesnake_disk3`. Saved with `Ctrl+S`, closed the editor with `Ctrl+Q`.

Since Cloud Shell sessions are ephemeral, the active subscription context can drift from whatever's selected in the portal — so I double-checked it first:

```powershell
Get-AzContext
Set-AzContext -Subscription <your-subscription-id>
```

Then deployed to the resource group:

```powershell
New-AzResourceGroupDeployment -ResourceGroupName whitesnake -TemplateFile template.json -TemplateParameterFile parameters.json
```

Confirmed `ProvisioningState` came back as `Succeeded`, then checked the disk actually existed:

```powershell
Get-AzDisk | ft Name,ResourceGroupName,Location,DiskSizeGb,ProvisioningState
```

<img width="578" height="110" alt="image" src="https://github.com/user-attachments/assets/8e853850-8e50-4678-b224-e5003d7e47c6" />

---

## Task 4: Deploy a Template with the CLI

Same idea, now using Bash and the Azure CLI instead of PowerShell — still inside Cloud Shell.

Switched the Cloud Shell session over to **Bash**. Since switching shells can also reset the active subscription context, I checked that first too:

```bash
az account show
az account set --subscription <your-subscription-id>
```

Ran `ls` to confirm the template files from the previous task were still sitting in Cloud Shell storage.

Opened the editor again, changed the disk name in the template to `whitesnake_disk4`, saved (`Ctrl+S`), closed the editor (`Ctrl+Q`).

Deployed with the CLI equivalent of the PowerShell command from before:

```bash
az deployment group create --resource-group whitesnake --template-file template.json --parameters parameters.json
```

Confirmed `ProvisioningState: Succeeded`, then checked the disk list:

```bash
az disk list --resource-group whitesnake --output table
```

<img width="794" height="163" alt="image" src="https://github.com/user-attachments/assets/4b954155-78a5-448d-8aa5-6c61e1af6db0" />

---

## Task 5: Deploy a Resource Using Azure Bicep

Last one — same disk-deployment idea, but using a Bicep file instead of raw ARM JSON. Bicep is a cleaner, declarative syntax that compiles down to ARM under the hood.

Working from the provided `azuredeploydisk.bicep` file, still in a Bash Cloud Shell session, I uploaded it and opened it in the editor.

Reading through the file, it was pretty easy to follow how the disk resource was defined compared to the verbosity of the JSON template. I made three changes:

- `managedDiskName` (line 2) → `whitesnake_disk5`
- `diskSizeinGiB` (line 7) → `32`
- `sku` name (line 26) → `StandardSSD_LRS`

Saved (`Ctrl+S`), closed the editor (`Ctrl+Q`), then deployed directly from the Bicep file — no separate parameters file needed:

```bash
az deployment group create --resource-group whitesnake --template-file azuredeploydisk.bicep
```

Checked the disk list one more time to confirm:

```bash
az disk list --resource-group whitesnake --output table
```

<img width="773" height="148" alt="image" src="https://github.com/user-attachments/assets/9f2f366a-0b21-4aae-a881-044093ac26be" />

By the end of this lab I had five managed disks in the same resource group, each one deployed a different way — portal export/redeploy, PowerShell, CLI, and Bicep.

---

## Cleanup

If running this on a real subscription, delete the resource group afterward to avoid leftover cost:

- Azure portal: select the resource group → **Delete the resource group** → enter the name to confirm → **Delete** (confirm a second time)
- PowerShell: `Remove-AzResourceGroup -Name whitesnake`
- CLI: `az group delete --name whitesnake`

---

## Key Takeaways

- **ARM templates** let you deploy, manage, and monitor a whole group of resources together instead of one at a time.
- An ARM template is a **JSON** file that defines infrastructure declaratively instead of through a sequence of manual steps or scripts.
- Parameters can live in a **separate JSON file** instead of being hardcoded inline in the template.
- ARM templates can be deployed through the **portal, PowerShell, or CLI** — same template, different entry points.
- **Bicep** is an alternative to raw ARM JSON, also declarative, but with cleaner syntax, type safety, and support for reusable code.
