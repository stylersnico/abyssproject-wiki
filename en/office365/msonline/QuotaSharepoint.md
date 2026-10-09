---
title: Script to apply a storage limit to SharePoint sites
description: Script to apply a storage limit to SharePoint sites
published: true
date: 2026-10-09T08:00:00.000Z
tags: sharepoint, quota
editor: markdown
dateCreated: 2026-10-09T08:00:00.000Z
---

# Apply a storage limit to SharePoint sites

In this example, we will see how to set a 40 GB quota on every SharePoint site smaller than 40 GB, with a warning set at 30 GB (75% of the quota).

## Enable the storage limit

First, you need to enable quotas for all SharePoint sites.
In the SharePoint admin center, go to **Settings**, find the storage limit setting and set it to **Manual**.

Once done, the quota applies to all SharePoint sites.
The default quota is **25 TB**.

## Change all limits with a script

Since some SharePoint sites already exceed this limit, we will run this script, which applies the quota only to the SharePoint sites smaller than 40 GB.

````Bash
Connect-SPOService -Url ....

# Get the list of all sites
$sites = Get-SPOSite -Limit All

# Filter sites smaller than 40 GB
$filteredSites = $sites | Where-Object { $_.StorageUsageCurrent -lt 40000 }

# Loop through each filtered site to set the limit and the warning
foreach ($site in $filteredSites) {
    # Set the storage limit to 40 GB and the warning to 30 GB
    Set-SPOSite -Identity $site.Url -StorageQuota 40000 -StorageQuotaWarningLevel 30000
    
    Write-Host "Storage limit configured for site: $($site.Url)"
}

Write-Host "Quota configuration completed for all sites smaller than 40 GB."
````

You will get an overview of all SharePoint sites where the quota has been applied.
