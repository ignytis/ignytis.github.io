---
title: "Bash: search and replace operation in directory"
date: 2024-11-25T11:12:23+02:00
description: An example command to search and replace some text in a directory
tags: ["bash", "text", "howto"]
draft: false
---

To perform a Search and Replace operation in current directory, you can use:

```bash
grep -rl "OLD_STRING" . | xargs sed -i 's/OLD_STRING/NEW_STRING/g'
```

__NB!!!__ It's potentially harmful operation as no confirmation will be asked. It's highly recommended to have a backup copy of current directory in case if something goes wrong.

