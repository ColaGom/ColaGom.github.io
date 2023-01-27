---
layout: post
title: "Github action set-output is deprecated 해결"
categories: [Dev, DevOps]
tags: [github, github-action]

---

# set-up is deprecated

```bash
- name: Set output
  run: echo "::set-output name={name}::{value}"
```

기본 Github action workflow에서 step의 output값을 추가 할 때 위 처럼 활용했는데 보안 이슈로 deprecated

# 해결방안

```bash
- name: Set output
  run: echo "{name}={value}" >> $GITHUB_OUTPUT
```

위 처럼 수정해주면 끝
