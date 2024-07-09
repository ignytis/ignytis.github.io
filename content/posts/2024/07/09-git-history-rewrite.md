---
title: "Rewriting the Git history: user name and e-mail"
date: 2024-07-09T21:38:03+02:00
description: This article describes how to rewrite the user name and email in Git history
tags: ["git", "bash", "software-development"]
draft: false
---
## About

This script will help you to delete your name and/or e-mail address from git history. It might be useful, for instance, if you had discovered a typo in your name or e-mail address, or just want to delete your real personal data.

## The scope

- User's name and email to be changed
- A list of emails to patch will be provided. Only those comments which author's email is listed will be rewritten. Other e-mails and names will stay unchanged
- Git history will be updated:
  - All branches
  - All tags

## Warning

### Note 1

This method rewrites the Git history, so your modified local repository will be incompatible with
original one (i.e. you will be unable to merge / push / pull from that repo).
After rewriting the history you will need to force push into your original repo which
is potentially dangerous. I suggest to have a backup of your repository to proceed.

### Note 2

Obviously, this script  will NOT help you in situation when somebody had forked your repo or any online tool had scraped your info from your public repo. In these cases you will need to contact with whoever is maintainer of those services or repository to erase your data

## Preparation

You need to fetch all branches and tags from your remote repo. An example script:

```bash
# Clone your project
git clone somegituser@gitexample.com:your_user/your_project.git
# Go into Git directory
cd your_project

# Maybe some extra here. Just to be 100% sure
git fetch --all
git pull --all
git fetch --tags
git pull --tags
```

## The script

You can save this code to some file like `rewrite_history.sh`

__NB:__ please review this code and define `CORRECT_NAME`, `CORRECT_EMAIL`, and list of emails to look up in `for` loop. It can be a single email too

```bash
#!/bin/sh
set -e # exit on error

git filter-branch -f --env-filter '
CORRECT_NAME="Your desired new name"
CORRECT_EMAIL="your.desired.new.email@example.com"

for OLD_EMAIL in old.email1@example.com old.email2@example.com
do
  if [ "$GIT_COMMITTER_EMAIL" = "$OLD_EMAIL" ]
  then
      export GIT_COMMITTER_NAME="$CORRECT_NAME"
      export GIT_COMMITTER_EMAIL="$CORRECT_EMAIL"
  fi
  if [ "$GIT_AUTHOR_EMAIL" = "$OLD_EMAIL" ]
  then
      export GIT_AUTHOR_NAME="$CORRECT_NAME"
      export GIT_AUTHOR_EMAIL="$CORRECT_EMAIL"
  fi
done
' --tag-name-filter cat -- --branches --tags
```

## Patching the repository

_Step 1._ Run the script above e.g. `bash rewrite_history.sh` from Git directory

_Step 2. Verification._ Check `git log` if the result looks good to you. You might want to switch to other branches or tags to check them as well

_Step 3. Force push_
```bash
git push --force origin your_branch:your_branch
# ^ you need to do it with each branch. Check 'git branch' to see a full list of your branches
git push --force --tags
```