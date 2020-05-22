---
layout: post
title: "ViewModel & ViewModelProviders"
---

### ViewModel?

[Google I/O 2017](https://www.youtube.com/watch?v=FrteWKKVyzI) 에서 발표된 Android Architucture Components에 포함된 라이브러리 중 하나이며, UI 와 관련된 데이터 처리에 관한 다양한 기능들을 제공한다.

![]({{ "/assets/images/viewmodel.png" | absolute_url }})

> The `ViewModel` class is designed to store and manage UI-related data in a lifecycle conscious way. The `ViewModel` class allows data to survive configuration changes such as screen rotations.

### ViewModelProviders?

ViewModelProvider를 사용하기위한 일종의 util class.

ViewModel을 제공(create 또는 get)해주는 기능을 제공. 
{% highlight kotlin %}
ViewModelProviders.of(this).get(MyViewModel.class);
{% endhighlight %}
위와 같은 형태로 사용하며 `ViewModelProviders`의 `ViewModelStore`에 적합한 `ViewModel`이 존재하는 경우 해당 `ViewModel`을 리턴하며 존재하지 않는 경우 `ProvideFactory`를 통해 새롭게 생성한 `ViewModel`을 리턴하는 방식으로 구현되어 있다.
{% highlight kotlin %}
private final HashMap<String, ViewModelStore> mViewModelStores = new HashMap<>();
//View Model Store in ViewModelProviders
{% endhighlight %}
### ViewModelProvider.Factory? (이하 Factory)

`ViewModelProviers`의 `ViewModelStore`에 원하는 `ViewModel`이 존재하지 않는 경우 새로운 `ViewModel`을 생성한다. 이 때, `**Factory**`를 통해서 `ViewModel`을 생성하며 기본적으로 인자 없는 `ViewModel`의 경우 `DefaultViewModelFactory`(`AndroidViewModelFactory`)를 ViewModel 생성을 진행한다.

만약, `ViewModel`의 생성 인자가 존재하는 경우 해당 `ViewModel`을 만들어낼 `Factory`를 따로 지정해 주어야 한다.
{% highlight kotlin %}
MyViewModelProviderFactory factory = new MyViewModelProviderFactory();
ViewModelProviders.of(this, factory).get(MyViewModel.class);
{% endhighlight %}
### 결론

- ViewModelProviders는 호출되는 view를 토대로 ViewModel을 생성하거나 찾아준다.
- Factory를 통해 VIewModel을 생성하며 따로 생성 인자가 없는 경우 따로 Factory class를 지정하지 않아도 된다.
    - 생성 인자가 있는 경우 ViewModelFactory를 지정해 주어야한다.

---

### Reference

[ViewModel](https://developer.android.com/topic/libraries/architecture/viewmodel)
[ViewModelProviders](https://developer.android.com/reference/android/arch/lifecycle/ViewModelProviders)