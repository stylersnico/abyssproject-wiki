---
title: Batch printing PDF files with a script
description: Batch printing PDF files with a simple script and Adobe Reader or Acrobat
published: true
date: 2026-10-09T08:00:00.000Z
tags: pdf printing, pdf
editor: markdown
dateCreated: 2026-10-09T08:00:00.000Z
---

# Introduction

These two scripts print batches of PDF files placed in the same folder as the scripts.
You may need to disable Adobe's protected mode for the scripts to work properly.



# Explanations

Create the following two files (or download the archive from GitHub):

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

Then put your PDF files in the folder that contains the scripts and run `print.bat`.

## Source
- https://github.com/stylersnico/batch-pdf-printer
