---
layout: post
title: "Github action 종속 job의 결과에 따라 slack 전송"
categories: [Dev, DevOps]
tags: [github, github-action]

---

# Needs

workflow 작성시 job의 종속job을 설정 할 수 있는 키워드이다.

```yaml
jobA:
  ~~
jobB:
  ~~
send-notify:
  needs: [ jobA, jobB ] # job A,B 가 모두 완료되야 실행된다
```

# 원하는 조건에 따른 if 키워드 작성예시

```yaml
{% raw %}
send-notify:
  needs: [ jobA, jobB ]
  runs-on: ubuntu-latest
  if: ${{ always() && needs.jobA.result == 'success' }} # jobA 성공
  if: ${{ always() && contains(join(needs.*.result, ','), 'success') }} # 하나라도 success인 경우
  if: ${{ always() && !contains(join(needs.*.result, ','), 'failure') }} # failure job이 하나도 없으면, skipped or canceled result 도 있으니 주의
  steps:
    - name: Notify slack
      env:
        SLACK_BOT_TOKEN: ${{ secrets.SLACK_BOT_TOKEN }}
        VERSION: ${{ needs.teardown.outputs.version }}
      uses: pullreminders/slack-action@master
      with:
        args: '{\"channel\":\"C1234567890\",\"text\":\"SUCCESS!!\"}'
{% endraw %}
```

# 참조

[Workflow syntax](https://docs.github.com/actions/using-workflows/workflow-syntax-for-github-actions#jobsjob_idneeds)
