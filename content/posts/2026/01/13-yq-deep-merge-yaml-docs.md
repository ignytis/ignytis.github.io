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

## Bonus tracks: key-based merging

### Insert items by key

```yaml
# 1.yaml

- name: alpha
  value1: aaa1
  value2: bbb1
- name: bravo
  value1: aaa2
  value2: bbb2
```

```yaml
# 2.yaml

name: myconfig
value: bravo
---
name: myconfig2
value: notexist
```

Result:

```bash
$ yq eval-all '(select(fileIndex == 0)) as $list |
    (select(fileIndex == 1)) as $cfg |
    (($list[] | select(.name == $cfg.value) | del(.name)) // {}) as $match |
    $cfg |
    .value = $match' 1.yaml 2.yaml

name: myconfig
value:
  value1: aaa2
  value2: bbb2
---
name: myconfig2
value: {}
```

### Merge items by key

```yaml
# 1.yaml

- name: alpha
  value1: aaa1
  value2: bbb1
- name: bravo
  value1: aaa2
  value2: bbb2
```

```yaml
# 2.yaml

name: myconfig0
value:
  rel_id: alpha
  value1: override
---
name: myconfig
value:
  rel_id: bravo
  value3: ccc3
```

Result:

```bash
$ yq eval-all '
    (select(fileIndex == 0)) as $list |
    (select(fileIndex == 1)) as $cfg |
    ($list[] | select(.name == $cfg.value.rel_id) | del(.name)) as $match |
    $cfg |
    .value = ( ($match // {}) * ( (.value // {}) | del(.rel_id) ) )
    ' 1.yaml 2.yaml 

name: myconfig0
value:
  value1: override
  value2: bbb1
---
name: myconfig
value:
  value1: aaa2
  value2: bbb2
  value3: ccc3
```
