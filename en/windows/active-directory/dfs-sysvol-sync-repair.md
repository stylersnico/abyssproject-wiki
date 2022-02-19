---
title: Repair DFS synchronisation of SYSVOL folder
description: Repair the DFS synchronisation of SYSVOL folder after crash of AD synchronisation
published: true
date: 2022-02-19T07:26:29.411Z
tags: active directory, dfs
editor: markdown
dateCreated: 2022-02-19T07:26:29.411Z
---

# Introduction

After a migration, we discovered that the sysvol folder was not replicating anymore on 7 of 7 domain controllers of 3 sites.

We are going to see how you can repair the DFSR sync of the SYSVOL folder after 5 years out of sync.

Here is the error that I had:

 ![dfrs-01.webp](/activedirectory/dfrs-01.webp)
 
And now, this is how I repaired it.

 
# Isolating the primary domain controller

In the first place, you need to isolate the Primary Domain Controller (PDC).

Launch this command on every domain controller that you have:
```powershell
netdom query pdc
```

It allows you to check if you have the same PDC on each domain controller.

Then, install the DFS tools on every DC:

```powershell
Install-WindowsFeature RSAT-DFS-Mgmt-Con
```
 

 
# Initializing Authoritative Sync

Your PDC will be the base for the restoration because it has normally the most up-to-date SYSVOL folder.

First, make a copy of your sysvol folder.
Then, stop DFSR service on all DC.

Now open « **ADSIEDIT.MSC** » on your PDC. Open the « **Default naming context** ».

Open the following configuration:
```powershell
CN=SYSVOL Subscription,CN=Domain System Volume,CN=DFSR-LocalSettings,CN=<PDCNAME>,OU=Domain Controllers,DC=<domain>,DC=<domain>
```
Change the following options:
```powershell
msDFSR-Enabled=FALSE
msDFSR-options=1
```

Now, edit the other DCs:
```powershell
CN=SYSVOL Subscription,CN=Domain System Volume,CN=DFSR-LocalSettings,CN=<Other AD AD>,OU=Domain Controllers,DC=<domain>,DC=<domain>
```

Change the following option:
```powershell
msDFSR-Enabled=FALSE
```

Force ad replication from the PDC:
```powershell
repadmin /syncall PDCNAME /APed
```

Restart DFSR service on the PDC.


In the event viewer, go in « **Applications and Services Logs -> DFS Replication** ».
Check for event **4114** after **5 minutes**.


If you are unlucky, event **2212** will drop, indicating a DFSR database corruption.
In the log you will find the command to rebuilt it:
```powershell
wmic /namespace:\\root\microsoftdfs path dfsrVolumeConfig where volumeGuid=<GUID> call ResumeReplication
```
 

Launch this as admin in CMD and restart DFSR service.
If you have event **4114** after that, check that the files in SYSVOL are up-to-date and continue.

 

# Synchronization with other Domain Controller

First, empty the SYSVOL folder of other domain controller:
```powershell
%WINDIR%\SYSVOL\domain\Policies
%WINDIR%\SYSVOL\domain\Scripts
```
 
Enable DFSR on PDC from ADSI:
```powershell
CN=SYSVOL Subscription,CN=Domain System Volume,CN=DFSR-LocalSettings,CN=<SERVEUR PDC>,OU=Domain Controllers,DC=<domain>,DC=<domain>
```

Change this option:
```powershell
msDFSR-Enabled=TRUE
```
 
Then, sync AD and DSFR from command line:
```powershell
DFSRDIAG POLLAD
repadmin /syncall NOMDUPDC /APed
```
 
You should have both events **2002 et 4602** on the PDC.

If this is good, enable DFSR sync on all other DC:
```powershell
CN=SYSVOL Subscription,CN=Domain System Volume,CN=DFSR-LocalSettings,CN=<Autres AD>,OU=Domain Controllers,DC=<domain>,DC=<domain>
```
 

Change this option:
```powershell
msDFSR-Enabled=TRUE
```
 

Launch this command on each AD:
```powershell
DFSRDIAG POLLAD
```
 

You should finally have events **2002 et 4602** on other DCs.

 

If you have unlucky again, you will have event **4012**.

> The DFS Replication service stopped replication on the folder with the following local path: C:\Windows\SYSVOL\domain. This server has been disconnected from other partners for 1821 days, which is longer than the time allowed by the MaxOfflineTimeInDays parameter (60). DFS Replication considers the data in this folder to be stale, and this server will not replicate the folder until this error is corrected.



You can cheat this, and edit the max offline delay like this:
```powershell
wmic.exe /namespace:\\root\microsoftdfs path DfsrMachineConfig set MaxOfflineTimeInDays=2000
```
 

Restart DFSR, restart sync command.
When it's ok, put back the default value:
```powershell
wmic.exe /namespace:\\root\microsoftdfs path DfsrMachineConfig set MaxOfflineTimeInDays=60
```

 
## Sources

- https://support.microsoft.com/en-us/help/2218556/how-to-force-an-authoritative-and-non-authoritative-synchronization-fo
- https://social.technet.microsoft.com/Forums/ie/en-US/c57791e6-c4f4-4e8f-9a74-eab985ecf614/event-id-4012-the-dfs-replication-service-stopped-replication?forum=winserverDS
