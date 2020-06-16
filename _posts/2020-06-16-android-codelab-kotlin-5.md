---
layout: post
title: "Android Codelab 정리 : ViewModel, LiveData, DataBinding"
author: "Sara Han"
categories : Android
comments : false
---

> 개인적으로 헷갈릴 때마다 참조하기 위한 용도로 매우 불친절한 설명들임. 

# Android Kotlin Fundamentals 05 

### ViewModelFactory 

예시 앱에서는 게임 화면에서 최종 점수 화면으로 넘어가는 상황을 보여준다. 게임을 하면서 얻은 점수를 ScoreFragment 에 디스플레이 하는게 목적이다. 이 때 GameFragment 에서 얻은 점수를 navigation 의 safe args 를 통해서 넘겨주고, 받은 점수를 ScoreViewModelFactory 에 넘겨주어서 ViewModelFactory 객체를 만들어 이를 이용해 ScoreViewModel 을 초기화한다. 



#### 1. ViewModel 을 만든다 

ScoreViewModel 은 인자값으로 finalScore 를 받는다. 

```kotlin
class ScoreViewModel(finalScore: Int) : ViewModel() {
   // The final score
   var score = finalScore
   init {
       Log.i("ScoreViewModel", "Final score is $finalScore")
   }
}
```



#### 2. ViewModelFactory 를 만든다 

```kotlin
class ScoreViewModelFactory(private val finalScore: Int) : ViewModelProvider.Factory {
    override fun <T : ViewModel?> create(modelClass: Class<T>): T {
       if (modelClass.isAssignableFrom(ScoreViewModel::class.java)) {
           return ScoreViewModel(finalScore) as T
       }
       throw IllegalArgumentException("Unknown ViewModel class")
    }
}
```

* create 함수를 오버라이딩 한다. 



#### Factory method pattern 이란? 

Creational design pattern 의 하나로, 객체를 생성할 때 팩토리 메서드를 사용하는 패턴이다. 팩토리 메서드는 같은 클래스의 인스턴스를 반환하는 메서드이다. 



#### 3. ScoreFragment 에서 ViewModel 을 팩토리를 이용해 초기화한다. 

```kotlin
private lateinit var viewModel: ScoreViewModel
private lateinit var viewModelFactory: ScoreViewModelFactory

...

viewModelFactory = ScoreViewModelFactory(ScoreFragmentArgs.fromBundle(arguments!!).score)

viewModel = ViewModelProviders.of(this, viewModelFactory)
       .get(ScoreViewModel::class.java)
```



---

### Data Binding with ViewModel and LiveData 

* 현재 앱의 아키텍처 

현재 샘플 앱에서는 XML 파일들에 뷰 들이 정의되어 있고, 뷰가 띄워야 하는 데이터들은 뷰모델 객체 안에 있다. view 와 이에 해당하는 ViewModel 사이에는 UI Controller 가 개입한다. 

예를 들어서, Button 이 XML 파일에 정의되어 있고, 이를 클릭하면 Fragment 가 해당하는 ViewModel 을 호출한다. 

다시말해서 ViewModel 과 Button (View) 는 직접적으로 소통하지 않는다. 클릭 리스너를 도입해야만 작동하는 것이다. 

![img](https://codelabs.developers.google.com/codelabs/kotlin-android-training-live-data-data-binding/img/3f68038d95411119.png)

* 데이터 바인딩을 이용한 방식 

만약 뷰들이 ViewModel 객체들과 직접적으로 소통할 수 있다면 더 좋을 것이다. (without relying on UI Controllers as intermediaries)

![img](https://codelabs.developers.google.com/codelabs/kotlin-android-training-live-data-data-binding/img/7f26738df2266dd6.png)



GameViewModel 의 game_fragment.xml 파일에다가 데이터 바인딩을 추가하자 

````kotlin
<layout ...>

   <data>

       <variable
           name="gameViewModel"
           type="com.example.android.guesstheword.screens.game.GameViewModel" />
   </data>
  
   <androidx.constraintlayout...
````



**Event 핸들링을 위해서 listener binding 을 사용하자** 

Listener binding 들은 바인딩 익스프레션으로, onClick(), onZoomIn() 과 같은 함수들이 트리거 되었을 때 수행되는 것이다. 리스너 바인딩들은 람다 표현식으로 작성되어 있다. 

데이터 바인딩은 리스너를 만들고, 뷰에다가 리스너를 셋팅해준다. listen 하고 있는 이벤트가 일어나면, 리스너는 람다 표현식을 evaluate 한다. 리스너 바인딩은 그래이들 플러그인 2.0 혹은 그 이상부터 작동한다. 

* [ListenerBindings](https://developer.android.com/topic/libraries/data-binding/expressions#listener_bindings)

아래와 같이 버튼에 리스너 바인딩을 할 수 있다. 

```kotlin
<Button
android:id="@+id/skip_button"
...
android:onClick="@{() -> gameViewModel.onSkip()}"
... />

<Button
android:id="@+id/correct_button"
...
android:onClick="@{() -> gameViewModel.onCorrect()}"
... />

<Button
android:id="@+id/end_game_button"
...
android:onClick="@{() -> gameViewModel.onGameFinish()}"
... />
```

위와 같이 해주면 Fragment 클래스 안에서 직접적으로 onClickListener 를 셋팅 해줄 필요가 없어진다. 



ScoreViewModel 에다가 데이터 바인딩을 추가해보자. 

score_fragment.xml 에다가 아래와 같이 데이터 바인딩을 넣어준다. 

```kotlin
<layout ...>
   <data>
       <variable
           name="scoreViewModel"
           type="com.example.android.guesstheword.screens.score.ScoreViewModel" />
   </data>
   <androidx.constraintlayout.widget.ConstraintLayout
```



리스너 바인딩을 버튼에 걸어준다. 

```kotlin
<Button
   android:id="@+id/play_again_button"
   ...
   android:onClick="@{() -> scoreViewModel.onPlayAgain()}"
   ... />
```



ScoreFragment  안에서 viewModel 을 초기화 해주자. 그리고 `binding.socreViewModel` 이라는 바인딩 변수를 초기화 해준다. 

```kotlin
viewModel = ...
binding.scoreViewModel = viewModel
```



이제 아래의 코드를 제거할 수 있다. 

```kotlin
binding.playAgainButton.setOnClickListener {  viewModel.onPlayAgain()  }
```





### 데이터 바인딩에 LiveData 추가하기 

ViewModel 객체와 함께 사용되는 LiveData 와 데이터바인딩은 같이 잘 작동한다. ViewModel 안에 있는 LiveData 를 XML 파일 안에다가 넣어주자. 

LiveData 와 데이터 바인딩을 같이 사용하면 Observer 함수를 사용하지 않고도 UI에 변화를 notify 할 수 있다. 

```kotlin
<TextView
   android:id="@+id/word_text"
   ...
   android:text="@{gameViewModel.word}"
   ... />
```

만약 LiveData 가 null 이면 빈 스트링을 디스플레이 한다. 



GameFragment 안에서 onCreateView() 안에다가 gameViewModel 을 포기화 한 후에, Fragment View 를 바인딩 변수의 lifecycle owner 로 등록하자. 이는 LiveData 객체의 scope 를 정의해주고, 이로써 game_fragment.xml 의 뷰를 자동으로 업데이트 하는 것이 가능해진다. 

```kotlin
binding.gameViewModel = ...
// Specify the fragment view as the lifecycle owner of the binding.
// This is used so that the binding can observe LiveData updates
binding.lifecycleOwner = viewLifecycleOwner
```



아래의 Observer ... 를 없앨 수 있다. 

```kotlin
/** Setting up LiveData observation relationship **/
viewModel.word.observe(viewLifecycleOwner, Observer { newWord ->
   binding.wordText.text = newWord
})
```



데이터 바인딩을 사용하면 아래와 같이 String.valueOf() 와 같은 메서드도 xml 안에서 사용 가능하다. 

```kotlin
<TextView
   android:id="@+id/score_text"
   ...
   android:text="@{String.valueOf(scoreViewModel.score)}"
   ... />
```



마찬가지로 lifecycle owner 를 등록해준다. 

```kotlin
binding.scoreViewModel = ...
// Specify the fragment view as the lifecycle owner of the binding.
// This is used so that the binding can observe LiveData updates
binding.lifecycleOwner = viewLifecycleOwner
```



아래의 Observer ... 를 없앨 수 있다. 

```kotlin
// Add observer for score
viewModel.score.observe(viewLifecycleOwner, Observer { newScore ->
   binding.scoreText.text = newScore.toString()
})
```





String formatting 과 data binding 을 함께 사용할수도 있다. 

strings.xml 파일에 아래와 같은 포맷 스트링이 있다 하자 

```kotlin
<string name="quote_format">\"%s\"</string>
<string name="score_format">Current Score: %d</string>
```



layout 파일에다가 위의 리소스로 포맷을 하라고 명시해준다. 

```kotlin
<TextView
   android:id="@+id/word_text"
   ...
   android:text="@{@string/quote_format(gameViewModel.word)}"
   ... />
   
   <TextView
   android:id="@+id/score_text"
   ...
   android:text="@{@string/score_format(gameViewModel.score)}"
   ... />
```





#### 데이터 바인딩 보충 

![img](https://codelabs.developers.google.com/codelabs/kotlin-android-training-data-binding-basics/img/204bd94c4dd5dd37.jpeg)

findViewById() 로 뷰를 참조하던 시절이 있다. 이 함수를 사용할 때마다 안드로이드 시스템은 뷰 계층 구조를 런타임에 traverse 하면서 뷰를 찾는다. 만약 앱 안에 뷰가 몇 개 없다면 문제가 되지 않으나, 프로덕션 앱은 뷰가 매우 많다. 그리고 최적의 디자인을 한다 하더라도 nested View 는 피할 수 없을 것이다. 

변수에 뷰들을 캐싱하는 것이 도움이 될 수는 있으나, 여전히 각 뷰와 관련된 변수들을 초기화해야 한다. 

하나의 해결책은 각 뷰에 대한 참조를 갖고 있는 객체 하나를 만드는 것이다. 이는 `BindingObject` 라 불리운다. 이는 앱 전체 안에서 사용 가능하다. 이 기법을 `data binding` 이라 부른다. 

바인딩 객체가 생성이 되면, 뷰와 다른 데이터들을 이 바인딩 객체를 통해서 접근할 수 있다. 이로써 View hierarchy 를 traverse 할 필요가 없어진다. 



Data Binding 은 다음과 같은 장점이 있다. 

* 코드가 짧아지고, 코드의 가독성이 높아지며 findViewById 를 사용 하는 것 보다 유지하기 쉬워진다. 
* 데이타와 뷰가 명확히 분리된다. 
* 안드로이드 시스템은 뷰 계층구조를 오직 한 번만 traverse 한다. 그리고 이 과정은 app startup 때 일어나지, 유저가 앱과 interact 하는 runtime 동안 일어나지 않는다. 
* View 를 접근하는 데 있어 type safety 가 보장된다. (Type Safety 란 컴파일러가 컴파일링 하는 동안 타입을 validate 하며, 변수에 잘못된 타입을 할당했을 때 error 를 발생시키는 것을 일컫는다.)



데이터 바인딩 사용하기 

`build.gradle(Module : app)` 에다가 

```groovy
dataBinding {
    enabled = true
}
```



레이아웃 감싸기 

```xml
<layout>
   <LinearLayout ... >
   ...
   </LinearLayout>
</layout>
```



Binding 객체를 형성한다. 

the name is derived from the name of the layout file, that is, `activity_main + Binding`.

```kotlin
private lateinit var binding: ActivityMainBinding
```



setContentView 함수를 다음과 같이 대체한다. 

```kotlin
import androidx.databinding.DataBindingUtil
...
binding = DataBindingUtil.setContentView(this, R.layout.activity_main)
```

위의 과정을 통해서 바인딩 객체가 생겨난다. 

`setContentView()` 함수를 DataBindingUtil 클래스로부터 사용해 activity_main 와 MainActivity 사이에 연결다리를 놓아준다. 



아래와 같이 뷰에 접근할 수 있다. 

```kotlin
binding.nicknameText.text = binding.nicknameEdit.text
binding.nicknameEdit.visibility = View.GONE
binding.doneButton.visibility = View.GONE
binding.nicknameText.visibility = View.VISIBLE
```

혹은 좀 더 코틀린 스럽게 

```kotlin
binding.apply {
   nicknameText.text = nicknameEdit.text.toString()
   nicknameEdit.visibility = View.GONE
   doneButton.visibility = View.GONE
   nicknameText.visibility = View.VISIBLE
}
```



변수 생성하기 

```kotlin
<data>
  <variable
       name="myName"
       type="com.example.android.aboutme.MyName" />
</data>
```

뷰에 꽂아주기 

```kotlin
android:text="@={myName.name}"
```

데이터 만들고 할당하기 

```kotlin
private val myName: MyName = MyName("Aleks Haecky")
...
binding.myName = myName
```





---

### LiveData Transformations 

라이브 데이터에 특정한 calculation 을 수행하거나, 데이터의 일부만 디스플레이 하거나, 데이터의 rendition(?) 을 변경하고 싶을 수 도 있다. 예를들어 word LiveData 에 맞은 단어가 아니라 맞은 단어의 개수를 보여주고 싶을 수 도 있다. 



* `Transformations.map` 
* `Transformations.switchMap` 

위의 클래스를 이용해서 위와같은 연산을 할 수 있다. 

코드랩에서는 `Transformations.map` 을 이용해 LiveData 에 변형을 가해서 시간이 얼마나 흘렀는지 화면에 디스플레이 할 것이다. 



`CountDownTimer` 이라는 유틸리티 클래스를 활용해서 타이머를 만들자 



GameViewModel 클래스 안에다가 companion object 들을 정의하자 

```kotlin
companion object {

   // Time when the game is over
   private const val DONE = 0L

   // Countdown time interval
   private const val ONE_SECOND = 1000L

   // Total time for the game
   private const val COUNTDOWN_TIME = 60000L

}
```



시간을 저장하기 위해서 아래와 같이 변수와 backing property 를 만들어주자. 

```kotlin
// Countdown time
private val _currentTime = MutableLiveData<Long>()
val currentTime: LiveData<Long>
   get() = _currentTime
```



timer 변수를 선언하고, init 블럭 안에다가 해당 변수를 초기화하자 

```kotlin
private val timer: CountDownTimer

... 

// Creates a timer which triggers the end of the game when it finishes
timer = object : CountDownTimer(COUNTDOWN_TIME, ONE_SECOND) {

   override fun onTick(millisUntilFinished: Long) {
       _currentTime.value = millisUntilFinished/ONE_SECOND
   }

   override fun onFinish() {
        _currentTime.value = DONE
       onGameFinish()
   }
}

timer.start()
```



onCleared 에다가 타이머 캔슬을 해주자 

```kotlin
override fun onCleared() {
   super.onCleared()
   // Cancel the timer
   timer.cancel()
}
```



Transformations.map 메서드는 source LiveData 에 data manipulation 을 할 수 있는 방법을 제공해준다. 이는 LiveData 객체로 결과를 리턴한다. 이 Transformation 들은 observer 가 반환된 LiveData 를 observe 하지 않는 이상 수행되지 않는다. 

위 함수는 Main Thread 에서 수행되므로 long running task 들은 하지 않도록 주의하자. 

```kotlin
// The String version of the current time
val currentTimeString = Transformations.map(currentTime) { time ->
   DateUtils.formatElapsedTime(time)
}
```

`DateUtils.formatElapsedTime()` 함수는 밀리세컨드를 "00분:00초"(MM:SS)와 같은 형식으로 만들어준다. 

