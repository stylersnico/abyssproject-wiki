---
title: Impression de PDF en lots via script
description: Impression de PDF en lots via script simple et Adobe Reader ou Acrobat
published: true
date: 2023-08-10T12:09:32.066Z
tags: impression pdf, pdf
editor: markdown
dateCreated: 2023-08-10T12:09:32.066Z
---

# Introduction

Ces deux scripts sont faits pour imprimer des lots de PDF qui seront placés dans le dossier ou le script est présent.
Il est possible que vous deviez désactiver le mode sécurisé d'Adobe pour le bon fonctionnement du script.



# Explications

Créez les deux fichiers suivants (ou téléchargez l'archive depuis le github):

`print.bat`
```batch
REM Kill all existings Reader instance
taskkill /F /IM AcroRd32.exe

REM Launch the background script to kill the Acrobat Reader after each print because he don't do that itself
start cmd.exe /c kill.bat

REM Launch the loop to print all the files in the folder and launch back the program killer after each print
for %%i in (*.pdf) do (
    "C:\Program Files (x86)\Adobe\Acrobat Reader DC\Reader\AcroRd32.exe" /t "%%i"
	start cmd.exe /c kill.bat
)
```

`kill.bat`
```batch
REM Default timer to give some time for Adobe Reader to print the file
timeout /t 15 /nobreak

REM Kill Adobe Reader after printing
taskkill /F /IM AcroRd32.exe
```

Ensuite, mettez des PDF dans le dossier qui contient les scripts et lancez le script `print.bat`.

## Source
- https://github.com/stylersnico/batch-pdf-printer