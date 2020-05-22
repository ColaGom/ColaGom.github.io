---
layout: post
title: "[Error]Kotlin bundle at onCreate needs nullable"
category: Dev
tags: [android]

---


### Problem

Java to kotlin 마이그레이션 진행 중 `android.app.Dialog`의 `onCreate` signature가 일치하지 않는 경우 `Exception` 발생.

마이그레이션 진행 시 parent의 parameter가 nullable인 경우를 항상 주의 해야 된다.

`androidx`에 포함된 view들은 전부(maybe) `nonnull annotation`의 형태로 구현되어 있는것으로 파악.

### Solution

**before**
{% highlight kotlin %}
override fun onCreate(savedInstanceState: Bundle)
{% endhighlight %}
**after**
{% highlight kotlin %}
override fun onCreate(savedInstanceState: Bundle?) //nullable
{% endhighlight %}
