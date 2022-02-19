---
title: Réparer la synchronisation DFS du dossier SYSVOL
description: Réparer la synchronisation DFS du dossier SYSVOL après un crash de la synchronisation AD
published: true
date: 2022-02-19T07:09:29.522Z
tags: active directory, dfs
editor: markdown
dateCreated: 2021-08-24T16:03:04.514Z
---

# Introduction

Après une migration, nous avons découvert que le sysvol ne se répliquait plus sur 7 des 7 contrôleurs de domaines des 3 sites.

Nous allons voir comment réparer la synchronisation DFSR du dossier SYSVOL après 5 ans de perte de synchronisation.

Voici le message d'erreur que j'avais :

 ![dfrs-01.webp](/activedirectory/dfrs-01.webp)
Voici ce que j'ai fait pour réparer.

 
# Isolation du contrôleur de domaine primaire

En premier lieu, il vous faudra isoler le contrôleur de domaine primaire.

Lancez la commande suivante depuis tous vos contrôleurs de domaines en powershell (ou cmd) :
```powershell
netdom query pdc
```

Cela permets de s'assurer que tous les contrôlleurs de domaines dispose du même PDC, ce qui peut ne pas être le cas si le problème est présent depuis longtemps.


Ensuite, installez directement les outils DFS qui nous serviront pour la suite en powershell (à faire sur chaque contrôleur de domaine) :
```powershell
Install-WindowsFeature RSAT-DFS-Mgmt-Con
```
 

 
# Initialisation de la synchronisation d’autorité

Votre PDC va devoir servir de base à la restauration, car, c’est normalement lui qui dispose du dossier sysvol le plus à jour.

Dans un premier temps, faites une copie de votre dossier sysvol sur le bureau.
Ensuite, stoppez le service DFSR sur tous les contrôleurs de domaine.

Ouvrez maintenant « **ADSIEDIT.MSC** » sur votre PDC. Ouvrez le « **Default naming context** ».

Maintenant, ouvrez la configuration suivante :
```powershell
CN=SYSVOL Subscription,CN=Domain System Volume,CN=DFSR-LocalSettings,CN=<SERVEUR PDC>,OU=Domain Controllers,DC=<domain>,DC=<domain>
```

Changez les options suivantes :
```powershell
msDFSR-Enabled=FALSE
msDFSR-options=1
```

Maintenant, modifiez tous les autres AD du domaine :
```powershell
CN=SYSVOL Subscription,CN=Domain System Volume,CN=DFSR-LocalSettings,CN=<Autres AD>,OU=Domain Controllers,DC=<domain>,DC=<domain>
```

Changez l’option suivante :
```powershell
msDFSR-Enabled=FALSE
```

Forcez une synchronisation AD depuis le PDC :
```powershell
repadmin /syncall NOMDUPDC /APed
```

Redémarrez DFSR sur le PDC.


Ouvrez l’Event Viewer et allez dans « **Applications and Services Logs -> DFS Replication** ».
Vérifiez que vous trouvez l’événement **4114** après **5 minutes**.


Si vous n’avez pas de chance, comme moi, vous tomberez sur l’événement **2212** indiquant une corruption de la base DFSR.
Dans le log de cet évent, vous trouverez une commande à lancer sous la forme suivante pour reconstruire la base :
```powershell
wmic /namespace:\\root\microsoftdfs path dfsrVolumeConfig where volumeGuid=<GUID> call ResumeReplication
```
 

Lancez-la dans un CMD administrateur et redémarrez DFSR.
Si vous avez finalement l’évent **4114**, vérifiez que les fichiers du SYSVOL sont à jour et continuez.

 

# Synchronisation des autres contrôleurs de domaines

Nettoyer d’abord les dossiers sysvol des autres contrôleurs de domaines :
```powershell
%WINDIR%\SYSVOL\domain\Policies
%WINDIR%\SYSVOL\domain\Scripts
```
 
Réactivez DFSR sur le PDC depuis l’ADSI :
```powershell
CN=SYSVOL Subscription,CN=Domain System Volume,CN=DFSR-LocalSettings,CN=<SERVEUR PDC>,OU=Domain Controllers,DC=<domain>,DC=<domain>
```

Changez l’option suivante :
```powershell
msDFSR-Enabled=TRUE
```
 
Ensuite, resynchronisez l’AD et le DFSR depuis la ligne de commande :
```powershell
DFSRDIAG POLLAD
repadmin /syncall NOMDUPDC /APed
```
 

Vous devriez voir les événements **2002 et 4602** sur le PDC.

Si c’est bon, réactivez la synchronisation sur tous les autres contrôleurs de domaine :
```powershell
CN=SYSVOL Subscription,CN=Domain System Volume,CN=DFSR-LocalSettings,CN=<Autres AD>,OU=Domain Controllers,DC=<domain>,DC=<domain>
```
 

Changez l’option suivante :
```powershell
msDFSR-Enabled=TRUE
```
 

Relancez la commande suivante (vous pouvez le faire sur chaque AD séparément):
```powershell
DFSRDIAG POLLAD
```
 

Vous devriez (enfin) voir les événements  **2002 et 4602** sur les autres contrôleurs de domaines.

 

Si vous faites comme moi, et que vous enchainez les coups de bol par contre, vous pourriez découvrir l’événement **4012**.

> The DFS Replication service stopped replication on the folder with the following local path: C:\Windows\SYSVOL\domain. This server has been disconnected from other partners for 1821 days, which is longer than the time allowed by the MaxOfflineTimeInDays parameter (60). DFS Replication considers the data in this folder to be stale, and this server will not replicate the folder until this error is corrected.



Vous pouvez truander et changer le paramètre bloquant sur chaque AD posant le souci :
```powershell
wmic.exe /namespace:\\root\microsoftdfs path DfsrMachineConfig set MaxOfflineTimeInDays=2000
```
 

Redémarrez le service DFSR et relancez les synchros.

Dès que ça passe, remettez la valeur par défaut :
```powershell
wmic.exe /namespace:\\root\microsoftdfs path DfsrMachineConfig set MaxOfflineTimeInDays=60
```

 
## Sources

- https://support.microsoft.com/en-us/help/2218556/how-to-force-an-authoritative-and-non-authoritative-synchronization-fo
- https://social.technet.microsoft.com/Forums/ie/en-US/c57791e6-c4f4-4e8f-9a74-eab985ecf614/event-id-4012-the-dfs-replication-service-stopped-replication?forum=winserverDS
