---
layout: post
title: "Github action set-output is deprecated"
categories: [Dev, DevOps]
tags: [github, github-action]

---

# AS-IS

```bash
- name: Set output
  run: echo "::set-output name={name}::{value}"
```

기본 Github action workflow에서 step의 output값을 추가 할 때 위 처럼 활용했는데 보안 이슈로 deprecated

# TO-BE

```bash
- name: Set output
  run: echo "{name}={value}" >> $GITHUB_OUTPUT
```
