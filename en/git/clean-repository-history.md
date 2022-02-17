---
title: Clean history of a GIT repository
description: Clean all history in a repository
published: true
date: 2022-02-17T08:30:11.640Z
tags: git
editor: markdown
dateCreated: 2022-02-17T08:18:06.733Z
---

# Before Starting

The goal here is to delete the history of a GIT repository.
This is useful, for example, if you merged sensitive information by error.

# Deleting the history of a repository

> Please make sure you have a good internet connection, you are gonna download the whole repository, delete it and upload it back.
{.is-warning}


First of all, connect to your git repository in command line.

> If the history is on another branch than "main", puth the name of the actual branch or you won't delete anything.
{.is-danger}

Checkout the actual repository
```
git checkout --orphan latest_branch
```

Add all the files
```
git add -A
```

Commit the changes
```
git commit -am "Deleting history"
```

Delete the branch

```
git branch -D main
```



Rename the current branch to main
```
git branch -m main
```

Finally, force update your repository
```
git push -f origin main
```


## Source
Thanks to: https://stackoverflow.com/a/26000395