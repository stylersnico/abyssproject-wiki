---
title: Nettoyage des logs Exchange
description: Nettoyage des logs Exchange en powershell
published: true
date: 2021-09-17T08:23:17.866Z
tags: exchange, powershell
editor: markdown
dateCreated: 2021-09-17T08:23:15.170Z
---

# Introduction

Le but de ce script est de nettoyer tous les logs Exchange.


# Script

Dans un IDE powershell ajoutez ceci pour tester le script (adaptez les dossiers si Exchange est ailleurs que sur C:) :

```powershell
Set-Executionpolicy RemoteSigned
$days=0
$IISLogPath="C:\inetpub\logs\LogFiles\"
$ExchangeLoggingPath="C:\Program Files\Microsoft\Exchange Server\V15\Logging\"
$ETLLoggingPath="C:\Program Files\Microsoft\Exchange Server\V15\Bin\Search\Ceres\Diagnostics\ETLTraces\"
$ETLLoggingPath2="C:\Program Files\Microsoft\Exchange Server\V15\Bin\Search\Ceres\Diagnostics\Logs"
Function CleanLogfiles($TargetFolder)
{
  write-host -debug -ForegroundColor Yellow -BackgroundColor Cyan $TargetFolder

    if (Test-Path $TargetFolder) {
        $Now = Get-Date
        $LastWrite = $Now.AddDays(-$days)
        $Files = Get-ChildItem $TargetFolder -Include *.log,*.blg, *.etl -Recurse | Where {$_.LastWriteTime -le "$LastWrite"} 
        foreach ($File in $Files)
            {
               $FullFileName = $File.FullName  
               Write-Host "Deleting file $FullFileName" -ForegroundColor "yellow"; 
                Remove-Item $FullFileName -ErrorAction SilentlyContinue | out-null
            }
       }
Else {
    Write-Host "The folder $TargetFolder doesn't exist! Check the folder path!" -ForegroundColor "red"
    }
}
CleanLogfiles($IISLogPath)
CleanLogfiles($ExchangeLoggingPath)
CleanLogfiles($ETLLoggingPath)
CleanLogfiles($ETLLoggingPath2)
```


    
## Sources


Cleanup logs Exchange 2013/2016/2019 par ALI TAJRAN : https://www.alitajran.com/cleanup-logs-exchange-2013-2016-2019/