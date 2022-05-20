---
layout: post
title: "CI&CD: GithubAction, AWS S3 및 Slack을 활용한 안드로이드 자동 배포기능 구현"
categories: [Dev, CI&CD]
tags: [github-action, aws, kmm]

---

# 목표

> github에 *새로운 `release`가 생겼을 때 `signed apk(release)`를 생성하여 `AWS S3`에 업로드하고 관련 메세지를 `slack`에 전송한다.*
>

# 해결하고자하는 것

- 기존에도 s3에 debug apk를 업로드하게 구현해뒀으나, 실제 배포환경과 동일한 테스트를 위해 signing apk 배포 필요
  - post signing
- 고정 URL의 S3에 업로드하는방식은 테스트 버전 파악이 쉽지않다.
  - version name with release 적용.
- 새로운 버전이 release 되었을때 notify
  - Slack
- APK 설치가 쉽지않다.
  - QR code & Action button

# Workflow 작성

### 1. 환경 변수 설정

```yaml
- name: Set vars
  id: vars
  run: |
    IFS='/'
    TAG="${GITHUB_REF#refs/*/}"
    read -a strarr <<< "$TAG"
    echo ::set-output name=tag::$TAG
    echo ::set-output name=platform::${strarr[0]}
    echo ::set-output name=version::${strarr[1]}
    echo ::set-output name=filename::**[업로드할 파일이름]**-${strarr[1]}.apk
```

github action에서 사용할 변수들을 미리 설정해주는 방식으로 작업.

`TAG`에 release tag값이 지정되며 이를 활용하여 필요한 변수들 저장

### 2. Signing APK

```yaml
- name: Signing APK
  id: sign_app
  uses: r0adkll/sign-android-release@v1
  with:
    releaseDirectory: androidApp/build/outputs/apk/release
    signingKeyBase64: ${{ secrets.SIGN_TOKEN }}
    alias: ${{ secrets.SIGN_ALIAS }}
    keyStorePassword: ${{ secrets.SIGN_STORE_PASSWORD }}
    keyPassword: ${{ secrets.SIGN_KEY_PASSWORD }}
```

keystore 자체를 repository에  넣어두고 signing을 할 수 있으나 보안상 apk build 후 따로 signing을 진행하는 식으로 작업 assembleRelease시 signing을 진행하는 경우 생략

# 전체 Workflow

```yaml
name: Android Deploy

on:
  workflow_dispatch:
  push:
    tags: 'aos/*'

env:
  GITHUB_URL: **[~~~]**
  AWS_REGION: ap-northeast-2

jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - name: Set vars
        id: vars
        run: |
          IFS='/'
          TAG="${GITHUB_REF#refs/*/}"
          read -a strarr <<< "$TAG"
          SIZE=${#strarr[*]}
          echo $TAG
          echo ::set-output name=tag::$TAG
          echo ::set-output name=platform::${strarr[0]}
          echo ::set-output name=version::${strarr[1]}
          echo ::set-output name=filename::**[~~~]**-${strarr[1]}.apk
      - name: set up JDK 11
        uses: actions/setup-java@v3
        with:
          java-version: '11'
          distribution: 'temurin'
          cache: gradle

      - name: Grant execute permission for gradlew
        run: chmod +x ./gradlew

      - name: Build Release APK with Gradle
        run: ./gradlew :androidApp:assembleRelease

      - name: Signing APK
        id: sign_app
        uses: r0adkll/sign-android-release@v1
        with:
          releaseDirectory: androidApp/build/outputs/apk/release
          signingKeyBase64: ${{ secrets.SIGN_TOKEN }}
          alias: ${{ secrets.SIGN_ALIAS }}
          keyStorePassword: ${{ secrets.SIGN_STORE_PASSWORD }}
          keyPassword: ${{ secrets.SIGN_KEY_PASSWORD }}

      - name: Rename apk
        run: mv ${{ steps.sign_app.outputs.signedReleaseFile }} ${{ steps.vars.outputs.filename }}

      - name: Upload apk to s3
        id: upload
        uses: hkusu/s3-upload-action@v2
        with:
          aws-access-key-id: ${{ secrets.AWS_ACCESS_KEY_ID }}
          aws-secret-access-key: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
          aws-region: ${{ env.AWS_REGION }}
          aws-bucket: ${{ secrets.AWS_BUCKET }}
          file-path: ${{ steps.vars.outputs.filename }}
          bucket-root: '/'
          destination-dir: 'android'
          output-file-url: 'true'
          output-qr-url: 'true'
          qr-width: 500
          public: 'true'

      - name: Notify slack
        env:
          SLACK_BOT_TOKEN: ${{ secrets.SLACK_BOT_TOKEN }}
          S3_URL: ${{ steps.upload.outputs.file-url }}
          QR_URL: ${{ steps.upload.outputs.qr-url }}
          CHANGE_URL: ${{ env.GITHUB_URL }}releases/tag/${{ steps.vars.outputs.tag }}
        uses: pullreminders/slack-action@master
        with:
          args: '{\"channel\": ~~~'
```

# Slack

![slack message](/assets/img/220520-1-1.png)

Block kit을 사용하여 원하는 message payload를 구현하여 전달

# References

[https://github.com/r0adkll/sign-android-release](https://github.com/r0adkll/sign-android-release)

[https://app.slack.com/block-kit-builder](https://app.slack.com/block-kit-builder)
