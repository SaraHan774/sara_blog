---
layout: post
title: "Kotlin Coroutines"
author: "Sara Han"
categories : Android
comments : false
---

# Use Kotlin Coroutines in your Android App

* [Codelab : Kotlin Coroutines](https://codelabs.developers.google.com/codelabs/kotlin-coroutines/#2)


코루틴 사용을 위해서 디펜던시 추가하기

```groovy
dependencies {
  ...
  implementation "org.jetbrains.kotlinx:kotlinx-coroutines-core:x.x.x"
  implementation "org.jetbrains.kotlinx:kotlinx-coroutines-android:x.x.x"
}
```



만약 RxJava 를 사용한다면 Kotlin Coroutines Rx 라이브러리를 사용하면 된다. 

* [Kotlinx.coroutines](https://github.com/Kotlin/kotlinx.coroutines/tree/master/reactive)



안드로이드에서는 메인 스레드를 blocking 하지 않는 것이 필수적이다. 메인 스레드는 하나의 스레드로, UI의 업데이트를 담당한다. 또한 Click Handler 과 다른 UI 콜백을 호출하는 역할을 하기도 한다. 

앱을 멈춤 없이 유저에게 보여줄 수 있으려면 메인 스레드는 매 16ms 마다 UI를 업데이트 할 수 있어야 하고, 이는 대락 1초에 60 frame 을 보여주는 것이다. 

많은 task 들이 이 보다 더 많은 시간을 필요로 하는데, 대표적으로 큰 JSON 데이타셋을 파싱 하거나, 데이터베이스에 write data 를 하거나, 네트워크로부터 데이터를 가져오는 task 들이 있다. 

따라서 위와 같은 작업들을 메인 스레드에서 한다면 앱이 멈추거나, 버벅거리거나 혹은 아예 freeze 되어 버릴 수 도 있다. 최악의 경우에는 ANR 다일로그가 뜨게 된다. 



### The Callback Pattern

long running task 들을 메인 스레드를 블럭하지 않고 수행하는 패턴 중 하나는 callback 을 사용하는 것이다. 콜백을 통해서 백그라운드 스레드에서 task 를 처리하고, 완료되면 결과를 메인 스레드에서 받을 수 있다. 

콜백 패턴

```kotlin
// Slow request with callbacks
@UiThread
fun makeNetworkRequest() {
    // The slow network request runs on another thread
    slowFetch { result ->
        // When the result is ready, this callback will get the result
        show(result)
    }
    // makeNetworkRequest() exits after calling slowFetch without waiting for the result
}
```

* @UiThread 가 있으므로 이 코드는 반드시 메인 스레드에서 수행되어야 한다. 다시말해, next screen update 가 delay 되지 않도록 매우 빨리 돌아가야 한다는 것이다. 
* 한편, slowFetch 는 몇 초, 혹은 몇 분이 걸려야 끝난다. 따라서 메인스레드는 해당 작업의 결과를 마냥 기다릴 수 없다. 
* show(result) 콜백은 slowFetch 가 백그라운드에서 돌아가도록 하고, 준비 되었을 때 slowFetch 가 결과를 반환하도록 한다. 



### 콜백의 단점 

* 가독성이 떨어진다. 
* exception 을 사용할 수 없다. 



### Using Coroutines to remove callbacks 

코루틴은 콜백 코드를 sequential code 로 변환하도록 한다. 

sequential 코드는 읽기 쉽고, exception 과 같은 language feature 도 사용할 수 있다. 

```kotlin
// Slow request with coroutines
@UiThread
suspend fun makeNetworkRequest() {
    // slowFetch is another suspend function so instead of 
    // blocking the main thread  makeNetworkRequest will `suspend` until the result is 
    // ready
    val result = slowFetch()
    // continue to execute after the result is ready
    show(result)
}

// slowFetch is main-safe using coroutines
suspend fun slowFetch(): SlowResult { ... }
```

* `suspend` 키워드는 어떠한 function, function type 을 coroutine 이 사용할 수 있도록 마킹하는 용도이다. 
* 코루틴이 suspend 가 붙은 함수를 호출하면, function return 이 올 때 까지 기다리는 것이 아니라, execution 을 suspend 하고, 결과가 준비되면 resume 한다. 
* 결과를 기다리는 동안 해당 코루틴이 돌아가던 스레드를 unblock 하고, 이 스레드의 다른 함수들 혹은 코루틴들이 돌아갈 수 있다. 



콜백 버전과 마찬가지로 makeNetworkRequest() 함수는 Main Thread 에서 돌아간다. (어노테이션 때문에.) 보통은 Main thread 에서 slowFetch 와 같은 blocking method 를 실행시킬 수 없지만, suspend 키워드는 Main 혹은 Background 둘 중 어떤 스레드에서도 돌아갈 수 있기 때문에 위와같은 마법이 가능하다. 

> Compared to callback-based code, coroutine code accomplishes the same result of unblocking the current thread with less code. Due to its sequential style, it's easy to chain several long running tasks without creating multiple callbacks.



```kotlin
// Request data from network and save it to database with coroutines

// Because of the @WorkerThread, this function cannot be called on the
// main thread without causing an error.
@WorkerThread
suspend fun makeNetworkRequest() {
    // slowFetch and anotherFetch are suspend functions
    val slow = slowFetch()
    val another = anotherFetch()
    // save is a regular function and will block this thread
    database.save(slow, another)
}

// slowFetch is main-safe using coroutines
suspend fun slowFetch(): SlowResult { ... }
// anotherFetch is main-safe using coroutines
suspend fun anotherFetch(): AnotherResult { ... }
```

여러 결과를 동시에 기다렸다가 DB operation 을 수행할 수 있다. 



### Understanding Coroutine Scope 

**모든 코루틴은 CoroutineScope 안에서 실행된다. Scope 은 코루틴의 lifetime 을 제어하고, 이는 Job 을 통해서 이루어진다.** 

만약 Scope 의 Job 의 캔슬하면, 해당 스코프 안에 있는 모든 코루틴들은 중단된다. 

예를 들면, 유저가 Activity 혹은 Fragment 에서 나갔을 때 모든 코루틴을 중단시키는 방법으로 Scope 를 사용할 수 있다. 

Scope 는 또한 default dispatcher 를 설정할 수 있도록 해준다. Dispatcher 는 어떤 스레드에서 코루틴이 돌아가는지 제어한다. 

UI 로부터 시작된 코루틴의 경우 보통 `Dispatchers.Main` 에서 시작하도록 하는게 올바르다. 이는 안드로이드의 메인스레드에 해당한다. `Dispatchers.Main` 에서 시작된 코루틴은 suspend 된 동안 Main thread 를 block 하지 않는다. 

`ViewModel` 의 코루틴은 거의 항상 Main thread 에 있는 UI 를 업데이트 하므로, **Main thread 에서 코루틴을 시작하면 thread switch 의 오버헤드를 줄일 수 있다.** 

Main thread 에서 시작한 코루틴은 시작한 후에 언제나 dispatcher 를 교체할 수 있다. 예를 들어 메인 스레드에서 벗어나는 다른 dispatcher 를 시작해서 large JSON 을 파싱 할 수 있다. 



### Coroutines Offer Main Safety 

코루틴은 언제나 thread 를 교체할 수 있고, 원래 코루틴이 시작된 스레드에 결과를 전달할 수 있기 떄문에, UI 관련된 코루틴을 Main thread 에서 시작하는 것이 바람직하다. 

Room 혹은 Retrofit 과 같은 라이브러리는 coroutine 을 사용할 때 main-safety 를 제공하므로 이와 관련된 래스크는 thread 를 관리해줄 필요가 없다. 

하지만 file 을 읽어들여서 list 를 정렬한다거나 하는 blocking code 의 경우 코루틴을 사용하더라도 명시적으로 main-safe 한 코드를 작성하는 것이 요구된다. 코루틴을 지원하지 않는 네트워킹 혹은 데이터베이스 라이브러리를 사용하는 경우도 마찬가지이다. 



### Using `viewModelScope` 

The AndroidX `lifecycle-viewmodel-ktx` library adds a CoroutineScope to ViewModels that's configured to start UI-related coroutines. 

```groovy
dependencies {
  ...
  implementation "androidx.lifecycle:lifecycle-viewmodel-ktx:x.x.x"
}
```

이 라이브러리는 `viewModelScope` 를 ViewModel 의 extension Function 으로 추가하였다. 

**This scope is bound to `Dispatchers.Main` and will automatically be cancelled when the `ViewModel` is cleared.**



```kotlin
/**
* Wait one second then update the tap count.
*/
private fun updateTaps() {
   // TODO: Convert updateTaps to use coroutines
   tapCount++
   BACKGROUND.submit {
       Thread.sleep(1_000)
       _taps.postValue("$tapCount taps")
   }
}
```

> This code uses the `BACKGROUND ExecutorService` (defined in `util/Executor.kt`) to run in a background thread. Since `sleep` blocks the current thread it would freeze the UI if it were called on the main thread. One second after the user clicks the main view, it requests a snackbar.



```kotlin
/**
* Wait one second then display a snackbar.
*/
fun updateTaps() {
   // launch a coroutine in viewModelScope
   viewModelScope.launch {
       tapCount++
       // suspend this coroutine for one second
       delay(1_000)
       // resume in the main dispatcher
       // _snackbar.value can be called directly from main thread
       _taps.postValue("$tapCount taps")
   }
}
```

> This code does the same thing, waiting one second before showing a snackbar. However, there are some important differences:

1. `viewModelScope.launch` 는 뷰모델 스코프 안에서 코루틴을 시작할 것이다. 우리가 viewModelScope 에다가 넘겨준 job 이 취소되는 경우, 해당 Job 혹은 Scope 안에 있는 모든 코루틴들이 취소된다. 만약 delay 가 return 하기 전에 유저가 Activity 를 떠났다면, 이 코루틴은 onCleared 에서 자동적으로 취소된다. 
2. viewModelScope 은 초기값으로 Dispatchers.Main 을 사용하기 때문에 메인 스레드에서 코루틴이 launch 될 것이다. 
3. delay 함수는 suspend function 이다. **이 코루틴은 main thread 에서 돌아가지만 delay 는 thread 를 1초동안 블러킹 하지 않을 것이다.** Dispatcher 는 코루틴이 1초 후 resume 하도록 스케줄 한다. 


---

*  참고자료
    * [Medium Post : Structured Concurrency](https://medium.com/@elizarov/structured-concurrency-722d765aa952)
    * [Kotlin Doc : Structured Concurrency](https://kotlinlang.org/docs/reference/coroutines/basics.html#structured-concurrency)

> Instead of launching coroutines in the GlobalScope, just like we usually do with threads (threads are always global), we can launch coroutines in the specific scope of the operation we are performing.

> Every coroutine builder, including runBlocking, adds an instance of CoroutineScope to the scope of its code block. We can launch coroutines in this scope without having to join them explicitly, because an outer coroutine (runBlocking in our example) does not complete until all the coroutines launched in its scope complete. 