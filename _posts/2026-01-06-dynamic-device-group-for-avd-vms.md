<!-- ---
title: Creating a Dynamic Device Group for AVD Virtual Machines
date: 2026-01-06 10:00:00 +0100
categories: [Azure Virtual Desktop, Entra ID]
tags: [avd, entra-id, dynamic-groups, powershell]
description: How to create a dynamic device group in Microsoft Entra ID for automatic management of Azure Virtual Desktop session hosts.
---

## Introduction

When managing an Azure Virtual Desktop (AVD) environment, it's useful to have an automatically updated group containing all session hosts. Dynamic groups in Microsoft Entra ID allow us to do this without manually adding devices.

## Why Use Dynamic Groups for AVD?

- 🔄 **Automatic management** - New session hosts are automatically added to the group
- 🎯 **Targeted policies** - Easy deployment of Intune policies or Conditional Access rules
- 🧹 **Clean environment** - Deleted VMs are automatically removed from the group

## Prerequisites

- Microsoft Entra ID P1 or P2 license
- AVD session hosts joined to Entra ID (Hybrid Join or Azure AD Join)
- Permissions to create groups in Entra ID

## Creating a Dynamic Group

### Option 1: Via Azure Portal

1. Navigate to **Microsoft Entra admin center** → **Groups** → **All groups**
2. Click **New group**
3. Configure:
   - **Group type**: Security
   - **Group name**: `AVD-SessionHosts`
   - **Membership type**: Dynamic Device
4. Click **Add dynamic query**

### Dynamic Group Rule

Use the following rule to capture AVD session hosts by name:

```text
(device.displayName -startsWith "AVD-")
```

Or if you use a specific naming pattern:

```text
(device.displayName -match "^AVD-HP-[0-9]+$")
```

### Option 2: PowerShell

```powershell
# Connect to Microsoft Graph
Connect-MgGraph -Scopes "Group.ReadWrite.All"

# Define group parameters
$groupParams = @{
    DisplayName = "AVD-SessionHosts"
    Description = "Dynamic group for Azure Virtual Desktop session hosts"
    GroupTypes = @("DynamicMembership")
    SecurityEnabled = $true
    MailEnabled = $false
    MailNickname = "avd-sessionhosts"
    MembershipRule = '(device.displayName -startsWith "AVD-")'
    MembershipRuleProcessingState = "On"
}

# Create the group
New-MgGroup @groupParams
```

## Advanced Rules

### By Device Extension Attribute

If you set extension attributes on AVD VMs:

```text
(device.extensionAttribute1 -eq "AVD")
```

### Combining Multiple Conditions

```text
(device.displayName -startsWith "AVD-") and (device.deviceOSType -eq "Windows") and (device.accountEnabled -eq true)
```

### By Device Model (for Azure VMs)

```text
(device.deviceModel -eq "Virtual Machine")
```

## Verifying Membership

After creating the group, it may take a few minutes for membership to update. Check the status:

```powershell
# Get the group
$group = Get-MgGroup -Filter "displayName eq 'AVD-SessionHosts'"

# Display members
Get-MgGroupMember -GroupId $group.Id | ForEach-Object {
    Get-MgDevice -DeviceId $_.Id | Select-Object DisplayName, DeviceId, OperatingSystem
}
```

## Tips and Recommendations

> When creating a naming convention for AVD session hosts, think ahead about dynamic groups. Consistent naming makes management easier.
{: .prompt-tip }

> Dynamic groups can have a delay of up to 24 hours during initial processing. For urgent cases, consider manual addition.
{: .prompt-warning }

## Using the Dynamic Group

Once you have the group created, you can use it for:

- **Intune policies** - Automatic deployment of configurations to session hosts
- **Conditional Access** - Specific rules for access from AVD
- **Azure RBAC** - Assigning permissions for VM management
- **Update rings** - Controlled deployment of Windows updates

## Conclusion

Dynamic groups are a powerful tool for automating AVD environment management. With a properly configured rule, you always have an up-to-date overview of all session hosts without manual maintenance.

---

*Have questions or your own tips for working with dynamic groups? Share them in the comments!*
 -->
