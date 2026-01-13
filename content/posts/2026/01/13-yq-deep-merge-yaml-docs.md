---
title: "Deep merging the YAML files using yq"
date: 2026-01-13T19:03:56
description: A short guide how to merge the YAML documents using yq tool
tags: ["yaml", "yq", "configuration"]
draft: false
---

## Example inputs

```yaml
# 1.yaml    
a:
  b: 1
  c:
    - apple
    - banana
top: foo
```

```yaml
# 2.yaml
a:
  d: 2
  c:
    - cherry
top: bar
```

## Deep merging, lists appended


```bash
$ yq eval-all '. as $item ireduce ({}; . *+ $item)' 1.yaml 2.yaml

a:
  b: 1
  c:
    - apple
    - banana
    - cherry
  d: 2
top: bar
```

## Deep merging, lists overwritten

```bash
$ yq eval-all 'select(fileIndex == 0) * select(fileIndex == 1)' 1.yaml 2.yaml

a:
  b: 1
  c:
    - cherry
  d: 2
top: bar
```