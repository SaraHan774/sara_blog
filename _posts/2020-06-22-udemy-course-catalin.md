---
layout: post
title: "[Udemy 강좌 정리] MVVM + Dagger2 + Unit Testing"
author: "Sara Han"
categories : Android
comments : false
---


* [Udemy Course Link](https://www.udemy.com/share/102j2WBEoccVhTQHk=/)
* [Github Repository Link](https://github.com/SaraHan774/CountryFlags)



<img src="{{site.baseurl}}/assets/img/Screenshot_1592805995.png" alt="Screenshot_1592805995" style="zoom:25%;" />

위와 같이 간단한 국기 정보를 RecyclerView 에 띄우는 앱을 만든다. 

* Retrofit 을 이용해 네트워크 처리를 한다. 
  * RxJava 의 Single 타입으로 네트워크 응답을 변환한다. 
  * Single 을 Disposable 에 추가하여 네트워크 처리 도중 앱 프로세스가 종료되었을 때 메모리 누수를 방지하도록 한다. 
* MVVM 아키텍처를 이용한다. 
  * Dagger2 를 이용해 ViewModel 에서 CountriesService (Retrofit 에서 fetchCountries 하기 위해 필요한 객체) 를 주입받도록 하고, 
  * 마찬가지로 CountriesService 에서 interface CountriesApi 를 주입받도록 한다. 
  * 즉, CountriesApi -> CountriesService 의존성 주입, CountriesService -> ListViewModel 의존성 주입을 한다. 
* Unit Test 를 작성한다. 
  * Countries Service 를 ViewModel 에 inject 하도록 한다. 
  * 백그라운드 스레드를 기다리는 것으로 인해 테스트가 실패하지 않도록 synchronous 하게 네트워크 콜이 처리되도록 한다. 
  * Mockito 를 이용해서 mock 객체를 생성하고 결과를 테스트한다. 





## Retrofit 과 RxJava 사용하기 

```java
package com.gahee.countryflags.viewmodel;

import androidx.lifecycle.MutableLiveData;
import androidx.lifecycle.ViewModel;

import com.gahee.countryflags.di.DaggerApiComponent;
import com.gahee.countryflags.model.CountriesService;
import com.gahee.countryflags.model.CountryModel;

import java.util.List;

import javax.inject.Inject;

import io.reactivex.android.schedulers.AndroidSchedulers;
import io.reactivex.disposables.CompositeDisposable;
import io.reactivex.observers.DisposableSingleObserver;
import io.reactivex.schedulers.Schedulers;

public class ListViewModel extends ViewModel {
    //list of countries

    //1. define variables
    public MutableLiveData<List<CountryModel>> countries = new MutableLiveData<>();
    //live data is an object that generates value
    //값들은 비동기적으로 생성된다. Observable 이다.
    public MutableLiveData<Boolean> countryLoadError = new MutableLiveData<>();
    public MutableLiveData<Boolean> loading = new MutableLiveData<>();

    @Inject
    public CountriesService countriesService;

    public ListViewModel(){
        super(); //super 를 호출해야 한다.
        DaggerApiComponent.create().inject(this);
    }

    private CompositeDisposable disposable = new CompositeDisposable();

    public void refresh(){
        fetchCountries();
    }

    private void fetchCountries() {
        loading.setValue(true); //로딩을 일단 true 로 해둔다.
        //기다리는 동안 loading spinner 를 볼 수 있도록 해준다.

        //백그라운드 스레드에서 돌아가는 동안 app process 가 죽을 수 있다.
        //메모리 로스를 막기 위해서 RxJava 의 컴포넌트 사용.
        //disposable 을 이용한다.
        disposable.add(
                countriesService.getCountries() //retrofit 으로 백엔드와 통신
                .subscribeOn(Schedulers.newThread()) //Single 을 백그라운드 스레드에서 매니징 한다.
                .observeOn(AndroidSchedulers.mainThread()) //Main 에서 응답을 받을 수 있도록 한다.
                .subscribeWith(new DisposableSingleObserver<List<CountryModel>>(){
                //Single 로 반환을 받으므로 Single Observer,
                //Disposable 에 추가하므로 Disposable ~ Observer
                    @Override
                    public void onSuccess(List<CountryModel> countryModels) {
                        countries.setValue(countryModels);
                        countryLoadError.setValue(false);
                        loading.setValue(false);
                        //TEST : 세개의 LiveData 값들을 확인한다.
                    }
                    @Override
                    public void onError(Throwable e) {
                        //백엔드에서 에러가 나면 여기로 콜백이 온다.
                        countryLoadError.setValue(true);
                        loading.setValue(false);
                        e.printStackTrace();
                    }
                })
        );
    }

    @Override
    protected void onCleared() {
        super.onCleared();
        disposable.clear();
    }
}

```



### `CompositeDisposable` 이용해서 메모리 누수 방지하기 

* `CompositeDisposable` 을 이용하면 생성된 모든 Observable 을 안드로이드 라이프사이클에 맞춰 한 번에 모두 해제할 수 있음. 

* `.add()` : Disposable 객체를 container 에 넣거나, 만약 container 가 dispose 되었으면 Disposable 을 dispose 한다. 
  * 만약 add 가 성공적이라면 true, 아니라면 false 를 반환한다. 



### 다큐먼트 참고 



#### `CompositeDisposable.add(Disposable)` 

```
public boolean add(@NonNull
                   Disposable disposable)
```

Adds a disposable to this container or disposes it if the container has been disposed.

- **Specified by:**

  `add` in interface `io.reactivex.internal.disposables.DisposableContainer`

- **Parameters:**

  `disposable` - the disposable to add, not null

- **Returns:**

  true if successful, false if this container has been disposed

- **Throws:**

  `NullPointerException` - if `disposable` is null



#### `public interface Disposable` 

```
public interface Disposable
```

Represents a disposable resource. (일회성을 자원을 표현한다. )



#### `Single` 

* Observable 과 유사하다. 
* Single 은 Observable 의 한 형태이지만, Observable 처럼 존재하지 않는 곳에서부터 무한대까지의 임의의 연속된 값들을 배출하는 것과는 달리, **항상 한 가지 값 또는 오류 알림 둘 중 하나만 배출한다.** 
* 따라서 Single 을 구독할 때는 Observable 을 구독할 때 사용하는 세 개의 메서드 (onNext, onError, onCompleted) 다신 다음의 두 메서드만 사용할 수 있다. 
  * onSuccess : Single 은 자신이 배출하는 하나의 값을 이 메서드를 통해 전달한다. 
  * onError : Single 은 항목을 배출할 수 없을 때 이 메서드를 통해 Throwable 객체를 전달한다. 
* **Single 은 이 메서드 중 하나만, 그리고 한 번만 호출한다.** 메서드가 호출되면 Single 의 생명주기는 끝나고 구독도 종료된다. 



#### `Single` 연산자를 통한 구성

* `subscribeOn` : 특정 스케줄러 상에서 작동하는 Single 을 리턴한다. 



#### 스케줄러 

Observable 연산자 체인에 멀티스레딩을 적용하고 싶으면, 특정 스케줄러를 사용해서 연산자를 실행하면 된다. 

기본적으로 Observable 과 연산자 체인은 이처럼 스케줄러를 통해 동작한다. Subscribe 메서드가 호출되는 스레드를 사용해서 Observer 에게 알림을 보낸다. 

SubscribeOn 연산자는 다른 스케줄러를 지정해서 Observable 이 처리해야 할 연산자들을 실행시킨다. **(위의 예제에서는 새로운 백그라운드 스레드에서 연산을 처리한다는 의미.)**

ObserveOn 연산자는 Observable 이 Observer 에게 알림을 보낼 때 사용 할 스케줄러를 명시한다. **(위의 예제에서는 메인 스레드에 결과를 전달한다. )**



## Dagger2 사용하기 



### `CountriesApi` 에 대한 의존성 주입하기

```java
public class CountriesService {
...
     //creation 과 use of the object 를 분리하자
    @Inject
    public CountriesApi api;
...
}

```

> @Inject 어노테이션을 통해서 어떤 객체의 의존성을 주입할 것인지 표시한다. 



```java
@Module
public class ApiModule {

    private static final String BASE_URL = "https://raw.githubusercontent.com";

    //creates objects
    @Provides
    public CountriesApi provideCountriesApi(){
        return new Retrofit.Builder()
            .baseUrl(BASE_URL)
            .addConverterFactory(GsonConverterFactory.create())
            .addCallAdapterFactory(RxJava2CallAdapterFactory.create())
            .build()
            .create(CountriesApi.class);
    }

}
```

> @Module 을 통해서 어떻게 Inject 할 객체를 생성할지 작성한다. @Provides 어노테이션에서 CountriesApi 객체를 생성하게 된다. 



```java
@Component(modules = {ApiModule.class})
public interface ApiComponent {
    //Module 과 Injection location 사이의 다리를 바련한다.
    void inject(CountriesService countriesService);
    //어디다가 Module 을 inject 하는지 알려주는 것
}

```

> @Component 를 통해서 어디에 의존성을 주입하는 것인지 작성한다. 



=> 위의 과정을 거친 후 프로젝트를 Re-Build 해서 `DaggerApiComponent` 라는 클래스 파일이 생성되도록 한다. 



의존성을 주입하려면 `CountriesService` 의 생성자 에다가 다음과 같이 작성한다. 

```java
...
private CountriesService(){
    //interface ApiComponent 의 이름을 따서 만들어짐
    DaggerApiComponent.create().inject(this);
}
...
```





## Unit Test 작성하기 



```java
package com.gahee.countryflags;

import androidx.arch.core.executor.testing.InstantTaskExecutorRule;

import com.gahee.countryflags.model.CountriesService;
import com.gahee.countryflags.model.CountryModel;
import com.gahee.countryflags.viewmodel.ListViewModel;

import org.junit.Assert;
import org.junit.Before;
import org.junit.Rule;
import org.junit.Test;
import org.mockito.InjectMocks;
import org.mockito.Mock;
import org.mockito.Mockito;
import org.mockito.MockitoAnnotations;

import java.util.ArrayList;
import java.util.List;

import io.reactivex.Scheduler;
import io.reactivex.Single;
import io.reactivex.android.plugins.RxAndroidPlugins;
import io.reactivex.internal.schedulers.ExecutorScheduler;
import io.reactivex.plugins.RxJavaPlugins;

public class ListViewModelTest {

    @Rule
    public InstantTaskExecutorRule rule = new InstantTaskExecutorRule();
    //task will be instant.

    @Mock
    CountriesService countriesService;

    @InjectMocks
    ListViewModel listViewModel = new ListViewModel();

    @Before
    public void setUp(){
        MockitoAnnotations.initMocks(this);
    }

    @Before
    public void setUpRxSchedulers (){
        Scheduler immediate = new Scheduler() {
            @Override
            public Worker createWorker() {
                return new ExecutorScheduler.ExecutorWorker(
                        Runnable::run
                        , true);
            }
        };
        RxJavaPlugins.setInitNewThreadSchedulerHandler(schedulerCallable -> immediate);
        RxAndroidPlugins.setInitMainThreadSchedulerHandler(schedulerCallable -> immediate);
    }

    private Single<List<CountryModel>> testSingle;

    @Test
    public void getCountiesSuccess(){
        CountryModel countryModel = new CountryModel(
                "countryName", "capital", "flag"
        );

        ArrayList<CountryModel> countryModels = new ArrayList<>();
        countryModels.add(countryModel);

        testSingle = Single.just(countryModels);

        Mockito.when(countriesService.getCountries()).thenReturn(testSingle);

        listViewModel.refresh();

        Assert.assertEquals(1, listViewModel.countries.getValue().size());
        Assert.assertEquals(false, listViewModel.countryLoadError.getValue());
        Assert.assertEquals(false, listViewModel.loading.getValue());
    }

    @Test
    public void getCountriesError(){
        //test single that returns throwable
        testSingle = Single.error(new Throwable());
        Mockito.when(countriesService.getCountries()).thenReturn(testSingle);
        listViewModel.refresh();
        Assert.assertEquals(true, listViewModel.countryLoadError.getValue());
        Assert.assertEquals(false, listViewModel.loading.getValue());
    }

}
```

1. CountriesService 를 ViewModel 에 Inject 한다. 
2. listViewModel 의 LiveData 들의 값들을 테스트에 따라 알맞게 나오는지 확인한다. 



#### Class RxJavaPlugins 

* 특정한 RxJava 연산들에 핸들러를 주입하기 위한 Utility 클래스이다. 

#### setInitNewThreadSchedulerHandler

```java
public static void setInitNewThreadSchedulerHandler(@Nullable Function<? super Callable<Scheduler>,? extends Scheduler> handler)
```

Sets the specific hook function.

- **Parameters:**

  `handler` - the hook function to set, null allowed, but the function may not return null



* [Mockito Tutorial Link](https://javacodehouse.com/blog/mockito-tutorial/)
  * 추후에 정리해 볼 예정.


  ---

  <img src="{{site.baseurl}}/assets/img/UC-2b1d4862-aa06-4d40-b2e6-bb3f1c056353.jpg" alt="Screenshot_1592805995" style="zoom:33%;" />