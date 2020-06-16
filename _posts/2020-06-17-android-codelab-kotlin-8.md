---
layout: post
title: "Android Codelab 정리 : Retrofit, RecyclerView, DataBinding"
author: "Sara Han"
categories : Android
comments : false
---

# Android Kotlin Fundamentals 08 

### 정리한 자료들 

* [Android Kotlin Fundamentals 08.1: Getting data from the internet](https://codelabs.developers.google.com/codelabs/kotlin-android-training-internet-data/index.html?index=..%2F..android-kotlin-fundamentals#0)

* [Android Kotlin Fundamentals 08.2: Loading and displaying images from the internet](https://codelabs.developers.google.com/codelabs/kotlin-android-training-internet-images/#0)

* [Android Kotlin Fundamentals 08.3 Filtering and detail views with internet data](https://codelabs.developers.google.com/codelabs/kotlin-android-training-internet-filtering/#0)

### 더 보충할 자료들 

* [Binding Adpaters Documentation](https://developer.android.com/topic/libraries/data-binding/binding-adapters#automatic-setting)
* [Kotlin Coroutines Documentation](https://kotlinlang.org/docs/reference/coroutines/coroutine-context-and-dispatchers.html)

* [Kotlin Coroutines Codelab](https://codelabs.developers.google.com/codelabs/kotlin-coroutines/#0)

## 08.1 Getting data from the internet 

<img src="https://codelabs.developers.google.com/codelabs/kotlin-android-training-internet-data/img/b31d2df6e0f5e858.png" alt="img" style="zoom:25%;" />

<img src="https://codelabs.developers.google.com/codelabs/kotlin-android-training-internet-data/img/613d3568c44757d7.png" alt="img" style="zoom:25%;" />

위와 같은 화면을 만드는 코드랩이다. 

샘플 앱에서는 두 개의 메인 모듈이 있다 : 

* OverviewFragment 에서는 땅들의 사진을 보여주고 이는 RecyclerView 를 이용해 디스플레이 한다. 

* DetailViewFragment 에서는 각 땅들에 대한 정보를 보여준다. 



![img](https://codelabs.developers.google.com/codelabs/kotlin-android-training-internet-data/img/c5333bd8ad9bc1d5.png)

이 샘플 앱에서는 각 프래그먼트에 속한 뷰모델이 있다. 뷰 모델은 네트워크 레이어와 직접적으로 소통할 것이다. 

디테일 뷰모델은 하나의 땅에 대한 디테일 정보를 갖고 있게 될 것이다. 

각 뷰모델은 LiveData 를 사용하고, app 의 UI를 업데이트 하기 위해 life-cycle aware data binding 을 사용한다. 

두개의 프래그먼트를 이동하기 위해서 Navigation Component 를 사용하고, 선택된 땅에 대한 정보를 argument 로 넘겨준다. 



> The response from a web service is commonly formatted in [JSON](https://www.json.org/), an interchange format for representing structured data. You learn more about JSON in the next task, but the short explanation is that a JSON object is a collection of key-value pairs, sometimes called a *dictionary*, a *hash map*, or an *associative array*. A collection of JSON objects is a JSON array, and it's the array you get back as a response from a web service.

* associative array 라는 말은 처음 들어본다. 



### Retrofit 

retrofit 은 웹 서비스 API를 만들기 위해서 필수적으로 다음의 두 가지를 필요로 한다. 

1. base URI for the web service 
2. converter factory 



Converter 는 retrofit 에게 웹 서비스에서 데이터를 가져왔을 때 뭘 하는지 말해주는 역할을 한다. 이 경우에서는 retrofit 이 JSON 응답을 웹 서비스로부터 가져와서 String 으로 이를 반환하기를 원한다. 

레트로핏은 `ScalarsConverter` 라는 것이 있고, 이는 스트링과 다른 원시형 데이터들을 지원한다. 

```kotlin
private val retrofit = Retrofit.Builder()
   .addConverterFactory(ScalarsConverterFactory.create())
   .baseUrl(BASE_URL)
   .build()
```



retrofit 이 HTTP 요청을 통해서 어떻게 웹서버와 소통하는지 정의하는 인터페이스를 만들어라. 

```kotlin
interface MarsApiService {
    @GET("realestate")
    fun getProperties():
            Call<String>
}
```

retrofit 은 위 메소드를 통해서 엔드포인트와 베이스 URL 을 연결하고, Call 객체를 만든다. Call 객체는 요청을 시작하는 데 사용된다. 



retrofit 서비스가 초기화 되도록 public object 를 선언하자. 

```kotlin
object MarsApi {
    val retrofitService : MarsApiService by lazy { 
       retrofit.create(MarsApiService::class.java) }
}
```

`create()` 함수는 위의 인터페이스와 함께 레트로핏 서비스를 생성한다. 

위의 생성하는 call 은 expensive 할 뿐더러 앱은 오직 하나의 retrofit service instance 가 필요하기 때문에 public object 로서 앱의 전체에 서비스를 노출한다. 

싱글톤이므로 계속해서 같은 서비스 객체를 참조하게 된다. 



ViewModel 에서 웹 서비스를 Call하자. 

```kotlin
MarsApi.retrofitService.getProperties().enqueue( 
   object: Callback<String> {
       override fun onFailure(call: Call<String>, t: Throwable) {
   			_response.value = "Failure: " + t.message
		}
       override fun onFailure(call: Call<String>, t: Throwable) {
	   		_response.value = "Failure: " + t.message
		}
})
```

Call 에다가 Callback 을 enqueue 한다. 



현재까지의 코드로는 커다란 JSON String 이 반환된다. 



JSON String 을 Kotlin 객체로 만들어주는 Moshi 라는 라이브러리를 사용해서 변환하자. 

```groovy
implementation "com.squareup.moshi:moshi:$version_moshi"
implementation "com.squareup.moshi:moshi-kotlin:$version_moshi"

implementation "com.squareup.retrofit2:converter-moshi:$version_retrofit"
```



아래의 라이브러리는 삭제하도록 한다. 

```groovy
implementation "com.squareup.retrofit2:converter-scalars:$version_retrofit"
```



데이터 모델 클래스를 작성한다. 

```kotlin
data class MarsProperty(
   val id: String, 
   @Json(name = "img_src") val imgSrcUrl: String,
   val type: String,
   val price: Double
)
```

Double 은 JSON 의 숫자들 어떤것이든 표현할 수 있는 데이터타입이다. 



Moshi 객체를 만든다. 

모시의 어노테이션과 kotlin 이 제대로 작동하기 위해서는 KotlinJsonAdapterFactory 가 필요하다. 

```kotlin
private val moshi = Moshi.Builder()
   .add(KotlinJsonAdapterFactory())
   .build()
```



retrofit 객체를 Moshi 로 업데이트 한다 

```kotlin
private val retrofit = Retrofit.Builder()
   .addConverterFactory(MoshiConverterFactory.create(moshi))
   .baseUrl(BASE_URL)
   .build()
```



모델 객체를 리턴하도록 아래와 같이 수정한다. 

```kotlin
interface MarsApiService {
   @GET("realestate")
   fun getProperties():
      Call<List<MarsProperty>>
}
```

콜백 enqueue 한 것도 마찬가지로 위의 모델 클래스로 수정해준다. 



> Now the Retrofit API service is running, but it uses a callback with two callback methods that you had to implement. One method handles success and another handles failure, and the failure result reports exceptions. Your code would be more efficient and easier to read if you could use coroutines with exception handling, instead of using callbacks. Conveniently, Retrofit has a library that integrates coroutines.



레트로핏 콜백보다 코루틴으로 예외 처리를 하는 것이 코드를 더 깔끔하게 만든다. 



코루틴 디펜던시 추가 

```groovy
implementation "org.jetbrains.kotlinx:kotlinx-coroutines-core:$version_kotlin_coroutines"
implementation "org.jetbrains.kotlinx:kotlinx-coroutines-android:$version_kotlin_coroutines"

implementation "com.jakewharton.retrofit:retrofit2-kotlin-coroutines-adapter:$version_retrofit_coroutines_adapter"
```



코루틴 콜어댑터 팩토리를 사용하도록 레트로핏 빌더 수정 

```kotlin
private val retrofit = Retrofit.Builder()
        .addConverterFactory(MoshiConverterFactory.create(moshi))
        .addCallAdapterFactory(CoroutineCallAdapterFactory())
        .baseUrl(BASE_URL)
        .build()
```

📌 CallAdapter 들은 레트로핏에 Call 클래스 말고 다른 것을 리턴할 수 있는 기능을 부여한다. 

이 경우에서는 `CoroutineCallAdapterFactory` 는 Call 객체를 Deferred 로 바꿀 수 있도록 한다. 



Deferred 인터페이스는 코루틴 Job 을 정의하는데, 이는 결과 값을 반환한다. (Deferred 는 Job 을 상속받는다)

Deferred 인터페이스는 await() 이라는 메소드를 갖고 있고, 이는 값이 준비될 때 까지 코드를 블러킹 없이 기다릴 수 있도록 한다.



ViewModel 클래스 안에다가 coroutine job 을 추가한다. 

Main dispatcher 를 이용해서 새로운 job 을 위한 coroutine scope 을 만든다.

```kotlin
private var viewModelJob = Job()

private val coroutineScope = CoroutineScope(
   viewModelJob + Dispatchers.Main )
```



`Dispatchers.Main` 디스패처는 작업을 UI스레드에서 처리한다. Retrofit 이 모든 작업을 백그라운드에서 하기 때문에 scope 를 위한 별도의 스레드를 사용할 이유가 없다. 

이렇게 하면 MutableLiveData 에 값을 쉽게 업데이트 할 수 있다. 

아래와 같이 Call 부분을 수정해준다. 

```kotlin
private fun getMarsRealEstateProperties() {
   coroutineScope.launch {
       var getPropertiesDeferred = 
          MarsApi.retrofitService.getProperties()
       try {          
           _response.value = 
              "Success: ${listResult.size} Mars properties retrieved"
       } catch (e: Exception) {
           _response.value = "Failure: ${e.message}"
       }
   }
}
```



ViewModel 이 destroy 되면 데이터를 로딩 하는 것이 멈춰야 한다. 따라서 onCleared() 함수를 오버리이딩 해서 Job 을 캔슬할 수 있도록 한다. 

```kotlin
override fun onCleared() {
   super.onCleared()
   viewModelJob.cancel()
}
```



---



## 08.2 : Loading and displaying images from the internet 

Glide 추가하는 부분은 생략 



Binding Adapter 를 만들어서 글라이드를 호출하자. 

이미지를 위한 URL 이 있으므로 Glide 를 이용해서 이미지를 띄우기만 하면 된다. 

Binding Adapter는 extension method 로, view 와 bind된 data 사이에 앉아 있는 것이다. 데이터가 바뀌면 이를 위한 custom behavior 를 제공할 수 있도록 해준다. 

이 경우에서는 custom behavior 는 ImageView 에다가 Glide 를 이용해서 이미지를 로드하는 것이다. 

```kotlin
@BindingAdapter("imageUrl")
fun bindImage(imgView: ImageView, imgUrl: String?) {
    imgUrl?.let { 
        val imgUri = imgUrl.toUri().buildUpon().scheme("https").build()
        Glide.with(imgView.context)
            .load(imgUri)
            .into(imgView)
    }
}
```

최종적은 Uri 객체가 HTTPS 스킴을 사용하게 하도록 하고 싶다. 왜냐하면 이미지를 가져오는 서버가 해당 scheme 을 요구하기 때문이다. 이를 위해서는 `buildUpon` 을 사용해서 빌드를 해주면 된다. 

`toUri()` 는 kotlin extension function 이고, Android KTX core 라이브러리에서 온 것이다. 그래서 그냥 String class 의 일부인 것 처럼 보인다. 



데이터 바인딩 추가해주기 

```xml
<data>
   <variable
       name="viewModel"
       type="com.example.android.marsrealestate.overview.OverviewViewModel" />
</data>

...

app:imageUrl="@{viewModel.property.imgSrcUrl}"
```



바인딩 클래스를 임포트 하자 

```kotlin
val binding = GridViewItemBinding.inflate(inflater)
```



에러 처리를 위해서 bindImage() 를 좀 더 구체화 하자 

```kotlin
@BindingAdapter("imageUrl")
fun bindImage(imgView: ImageView, imgUrl: String?) {
    imgUrl?.let {
        val imgUri = 
           imgUrl.toUri().buildUpon().scheme("https").build()
        Glide.with(imgView.context)
                .load(imgUri)
                .apply(RequestOptions()
                        .placeholder(R.drawable.loading_animation)
                        .error(R.drawable.ic_broken_image))
                .into(imgView)
    }
}
```

> RequestOptions() 에다가 place holder 와 error 를 묶어서 처리해줄 수 있다. 



뷰모델을 `LiveData<List<MarsProperty>>` 를 갖고 있도록 업데이트 해주자 



네트워크 콜을 하는 부분은 아래와 같이 수정된다 

```kotlin
try {
   var listResult = getPropertiesDeferred.await()
   _response.value = "Success: ${listResult.size} Mars properties retrieved"
   _properties.value = listResult
} catch (e: Exception) {
   _response.value = "Failure: ${e.message}"
}
```



레이아웃과 프래그먼트를 업데이트 한다 

```xml
<variable
   name="property"
   type="com.example.android.marsrealestate.network.MarsProperty" />
   
   ...
   
   app:imageUrl="@{property.imgSrcUrl}"
```



OverviewFragment 의 binding 부분을 다시 수정한다. 

```kotlin
val binding = FragmentOverviewBinding.inflate(inflater)
 // val binding = GridViewItemBinding.inflate(inflater)
```



RecyclerView 를 xml 에 넣어준다 

```xml
<androidx.recyclerview.widget.RecyclerView
            android:id="@+id/photos_grid"
            android:layout_width="0dp"
            android:layout_height="0dp"
            android:padding="6dp"
            android:clipToPadding="false"
            app:layoutManager=
               "androidx.recyclerview.widget.GridLayoutManager"
            app:layout_constraintBottom_toBottomOf="parent"
            app:layout_constraintLeft_toLeftOf="parent"
            app:layout_constraintRight_toRightOf="parent"
            app:layout_constraintTop_toTopOf="parent"
            app:spanCount="2"
            tools:itemCount="16"
            tools:listitem="@layout/grid_view_item" />
```

* 레이아웃 매니저를 바로 지정해줄 수 있다. 

* 그리드 모양을 위한 span count 도 지정 가능하다 .



RecyclerView 만드는 부분 생략. 



ViewHolder 에다가 Binding 객체를 넘겨준다. 

> You need the `GridViewItemBinding` variable for binding the `MarsProperty` to the layout, so pass the variable into the `MarsPropertyViewHolder`. Because the base `ViewHolder` class requires a view in its constructor, you pass it the binding root view.

```kotlin
class MarsPropertyViewHolder(private var binding: 
                   GridViewItemBinding):
       RecyclerView.ViewHolder(binding.root) {
	
	fun bind(marsProperty: MarsProperty) {
 	  binding.property = marsProperty
      binding.executePendingBindings()	
	}
}
```

>  Call `executePendingBindings()` after setting the property, which causes the update to execute immediately.



 Binding Adpater 에 RecyclerView 를 위한 행동을 정의해준다. 

Binding Adapter 를 이용해서 RecyclerView 에다가 데이터를 set 하면 데이터 바인딩이 자동으로 해당 객체들에 대한 LiveData 를 관찰한다. 

바인딩 어댑터는 MarsProperty 리스트가 변경되면 자동으로 호출된다. 

```kotlin
@BindingAdapter("listData")
fun bindRecyclerView(recyclerView: RecyclerView, 
    data: List<MarsProperty>?) {
    val adapter = recyclerView.adapter as PhotoGridAdapter
	adapter.submitList(data)
}
```

> call `adapter.submitList()` with the data. This tells the `RecyclerView` when a new list is available.



XML 파일의 RecyclerView 에다가 아래의 바인딩을 추가해준다. 

```xml
app:listData="@{viewModel.properties}"
```



>  In `onCreateView()`, just before the call to `setHasOptionsMenu()`, initialize the `RecyclerView` adapter in `binding.photosGrid` to a new `PhotoGridAdapter` object.

```kotlin
binding.photosGrid.adapter = PhotoGridAdapter()
```



네트워크 상태를 확인해주기 위한 바인드 어댑터를 정의한다. 

에러, 로딩, 완료 시 어떤 이미지가 뜰지 View 에다가 행동을 넣어주는 것이다. 



OverviewViewModel 클래스 안에다가 Status 를 추가해준다. 

```kotlin
enum class MarsApiStatus { LOADING, ERROR, DONE }

...
private val _status = MutableLiveData<MarsApiStatus>()

val status: LiveData<MarsApiStatus>
   get() = _status
   
...

try {
    _status.value = MarsApiStatus.LOADING
   var listResult = getPropertiesDeferred.await()
   _status.value = MarsApiStatus.DONE
   _properties.value = listResult
} catch (e: Exception) {
   _status.value = MarsApiStatus.ERROR
     _properties.value = ArrayList()
}
```

> After the error state in the `catch {}` block, set the `_properties` `LiveData` to an empty list. This clears the `RecyclerView`.



에러 및 로딩 처리를 위한 바인딩 어댑터 

```kotlin
@BindingAdapter("marsApiStatus")
fun bindStatus(statusImageView: ImageView, 
          status: MarsApiStatus?) {
          
 when (status) {
   MarsApiStatus.LOADING -> {
      statusImageView.visibility = View.VISIBLE
      statusImageView.setImageResource(R.drawable.loading_animation)
   }
   ... //다른 status 도 처리해준다.
}
}
```



레이아웃에 status ImageView 를 넣어준다. 

```
<ImageView
   android:id="@+id/status_image"
    android:layout_width="wrap_content"
    android:layout_height="wrap_content"
    app:layout_constraintBottom_toBottomOf="parent"
    app:layout_constraintLeft_toLeftOf="parent"
    app:layout_constraintRight_toRightOf="parent"
    app:layout_constraintTop_toTopOf="parent"
    app:marsApiStatus="@{viewModel.status}" />
```

> Also notice the `app:marsApiStatus` attribute, which has the view call your `BindingAdapter` when the status property in the view model changes.

위의 `@BindingAdapter("marsApiStatus")` 가 `app:marsApiStatus` 와 연결되는 것을 확인하자. 이는 View 가 status 가 바뀔때마다 정의된 어댑터를 호출할 수 있도록 해준다. 



---



## 08.3 Filtering and detail views with internet data

<img src="https://codelabs.developers.google.com/codelabs/kotlin-android-training-internet-filtering/img/4b93ba4767e4a4d4.png" alt="img" style="zoom:25%;" />

필터링 하는 기능 추가할 것. 

<img src="https://codelabs.developers.google.com/codelabs/kotlin-android-training-internet-filtering/img/3732e88c965ac8c3.png" alt="img" style="zoom:25%;" />

디테일 화면 위와같이 수정할 것. 



모델 클래스에 isRental 이라는 속성을 추가한다. type 이 rent 이면 true 를 반환하는 getter 를 넣는다. 

```kotlin
data class MarsProperty(
       val id: String,
       @Json(name = "img_src") val imgSrcUrl: String,
       val type: String,
       val price: Double)  {
   val isRental
       get() = type == "rent"
}
```



View.VISIBLE 과 같은 상수를 xml 안에서 사용할 수 있도록 View class 를 `<data>` 엘리먼트에 추가해준다. 

```kotlin
<import type="android.view.View"/>
```



아래와 같이 ImageView 를 조건에 따라 보이거나 안보이도록 설정해준다. 

```kotlin
<layout xmlns:android="http://schemas.android.com/apk/res/android"
       xmlns:app="http://schemas.android.com/apk/res-auto"
       xmlns:tools="http://schemas.android.com/tools">
   <data>
       <import type="android.view.View"/>
       <variable
           name="property"
           type="com.example.android.marsrealestate.network.MarsProperty" />
   </data>
   <FrameLayout
       android:layout_width="match_parent"
       android:layout_height="170dp">

       <ImageView
           android:id="@+id/mars_image"
           android:layout_width="match_parent"
           android:layout_height="match_parent"
           android:scaleType="centerCrop"
           android:adjustViewBounds="true"
           android:padding="2dp"
           app:imageUrl="@{property.imgSrcUrl}"
           tools:src="@tools:sample/backgrounds/scenic"/>

       <ImageView
           android:id="@+id/mars_property_type"
           android:layout_width="wrap_content"
           android:layout_height="45dp"
           android:layout_gravity="bottom|end"
           android:adjustViewBounds="true"
           android:padding="5dp"
           android:scaleType="fitCenter"
           android:src="@drawable/ic_for_sale_outline"
           android:visibility="@{property.rental ? View.GONE : View.VISIBLE}"
           tools:src="@drawable/ic_for_sale_outline"/>
   </FrameLayout>
</layout>

```

* **android:visibility="@{property.rental ? View.GONE : View.VISIBLE}" 부분을 주목하자.** 



필터링 API 를 사용하기 위해서 API Service 클래스에 다음과 같이 추가해준다. 



매개변수가 있는 Enum class 이다. 

```kotlin
enum class MarsApiFilter(val value: String) {
   SHOW_RENT("rent"),
   SHOW_BUY("buy"),
   SHOW_ALL("all") }
```



```
fun getProperties(@Query("filter") type: String):  
```



OverviewViewModel 을 업데이트 한다. 위에서 만든 Filter 이넘 클래스 타입을 매개변수로 받는다. 

```kotlin
private fun getMarsRealEstateProperties(filter: MarsApiFilter) {
```



Retrofit 서비스의 getProperties() 메소드를 filter query 도 가능하게 수정해준다. 

```kotlin
var getPropertiesDeferred = MarsApi.retrofitService.getProperties(filter.value)
```



ViewModel 의 init 블럭에다가 SHOW_ALL 로 설정해준다. 

```kotlin
init {
   getMarsRealEstateProperties(MarsApiFilter.SHOW_ALL)
}
```



필터를 업데이트 할 수 있도록 updateFilter() 메서드를 추가해주자. 

```kotlin
fun updateFilter(filter: MarsApiFilter) {
   getMarsRealEstateProperties(filter)
}
```



Menu 추가하는 부분 생략 



onOptionsItemSelected 에다가 viewModel.updateFilter() 를 추가해준다. 

```kotlin
override fun onOptionsItemSelected(item: MenuItem): Boolean {
   viewModel.updateFilter(
           when (item.itemId) {
               R.id.show_rent_menu -> MarsApiFilter.SHOW_RENT
               R.id.show_buy_menu -> MarsApiFilter.SHOW_BUY
               else -> MarsApiFilter.SHOW_ALL
           }
   )
   return true
}
```



디테일 페이지 만들어주기 



디테일 ViewModel 부터 만들어준다. 

````kotlin
class DetailViewModel( marsProperty: MarsProperty,
                     app: Application) : AndroidViewModel(app) {
                     
   private val _selectedProperty = MutableLiveData<MarsProperty>()
val selectedProperty: LiveData<MarsProperty>
   get() = _selectedProperty
   
    init {
        _selectedProperty.value = marsProperty
    }
}
````

* **넘어온 정보로 init 블럭을 실행해준다.** 



디테일 레이아웃에 Data Binding 을 추가해준다. 

```xml
<data>
   <variable
       name="viewModel"
       type="com.example.android.marsrealestate.detail.DetailViewModel" />
</data>

...
 app:imageUrl="@{viewModel.selectedProperty.imgSrcUrl}"
```

> The binding adapter that loads an image using Glide will automatically be used here as well, because that adapter watches all `app:imageUrl` attributes.

imageUrl 속성으로 인해 이 이미지 뷰도 마찬가지로 Glide 를 이용해서 로딩할 것임. 



Overview ViewModel 에다가 Navigation 을 정의한다. 

```kotlin
private val _navigateToSelectedProperty = MutableLiveData<MarsProperty>()
val navigateToSelectedProperty: LiveData<MarsProperty>
   get() = _navigateToSelectedProperty


...

fun displayPropertyDetails(marsProperty: MarsProperty) {
   _navigateToSelectedProperty.value = marsProperty
}

fun displayPropertyDetailsComplete() {
   _navigateToSelectedProperty.value = null
}
```



PhotoGridAdpater 에다가 custom OnClickListener 를 정의해주자. 이 리스너는 marsProperty 를 매개변수로 받는다. onClick() 함수는 lambda 매개변수로 들어간다. 

```kotlin
class OnClickListener(val clickListener: (marsProperty:MarsProperty) -> Unit) {
     fun onClick(marsProperty:MarsProperty) = clickListener(marsProperty)
}
```



Adpater 에다가 해당 리스너를 property 로 넣는다. 

```kotlin
class PhotoGridAdapter( private val onClickListener: OnClickListener ) :
       ListAdapter<MarsProperty,              
           PhotoGridAdapter.MarsPropertyViewHolder>(DiffCallback) {
```



onBindViewHolder 안의 grid item 에다가 클릭 리스너를 추가하자. 

```kotlin
override fun onBindViewHolder(holder: MarsPropertyViewHolder, position: Int) {
   val marsProperty = getItem(position)
   holder.itemView.setOnClickListener {
       onClickListener.onClick(marsProperty)
   }
   holder.bind(marsProperty)
}
```



```kotlin
binding.photosGrid.adapter = PhotoGridAdapter(PhotoGridAdapter.OnClickListener {
   viewModel.displayPropertyDetails(it)
})
```

> Open `overview/OverviewFragment.kt`. In the `onCreateView()` method, replace the line that initializes the `binding.photosGrid.adapter` property with the line shown below.

> This code adds the `PhotoGridAdapter.onClickListener` object to the `PhotoGridAdapter` constructor, and calls `viewModel.displayPropertyDetails()` with the passed-in `MarsProperty` object. This triggers the `LiveData` in the view model for the navigation.



현재는 PhotoGridAdapter 를 통해서 클릭 리스너를 만들었는데, MarsProperty 객체를 디테일 페이지로 넘겨주지는 않는다. 이를 위해서는 navigation component 의 Safe Args 가 필요하다. 



Navigation 그래프 만드는 부분은 생략 



넘겨주는 객체가 Parcelable 이 아니면 navigation component 에서 에러를 낸다. Parcelable 인터페이스는 객체를 serialize 할 수 있게 한다. 이를 통해 fragment 와 activity 사이에서 데이터가 오가는 것이 가능해진다. 

```kotlin
@Parcelize
data class MarsProperty (
       val id: String,
       @Json(name = "img_src") val imgSrcUrl: String,
       val type: String,
       val price: Double) : Parcelable {
```

* @Parcelize 는 거추장스러운 메서드들을 자동으로 구현해주고 
* : Parcelable 로 인터페이스를 구현하는 것이다. 



it 이 null 이 아니라면 화면을 이동하고, 이동하면 바로 null 로 설정한다. 

```kotlin
viewModel.navigateToSelectedProperty.observe(this, Observer {
   if ( null != it ) {   
      this.findNavController().navigate(
              OverviewFragmentDirections.actionShowDetail(it))             
      viewModel.displayPropertyDetailsComplete()
   }
})
```



DetailFragment 의 onCreateView() 안의 setLifecycleOwner() 에다가 다음 라인을 추가하자. 

```kotlin
 val marsProperty = DetailFragmentArgs.fromBundle(arguments!!).selectedProperty
```

이를 통해 넘어온 safe args 를 회수한다. 



DetailViewModelFactory 를 통해서 DetailViewModel 을 초기화 해준다. 

```kotlin
val viewModelFactory = DetailViewModelFactory(marsProperty, application)

 binding.viewModel = ViewModelProviders.of(
                this, viewModelFactory).get(DetailViewModel::class.java)
```





더 유용한 Detail 페이지를 만들기 위해 아래를 추가한다. 

```xml
<string name="type_rent">Rent</string>
<string name="type_sale">Sale</string>
<string name="display_type">For %s</string>
<string name="display_price_monthly_rental">$%,.0f/month</string>
<string name="display_price">$%,.0f</string>
```



DetailViewModel 에다가 아래와 같이 선택된 땅의 가격에 따라서 String 을 달리하는 Transformation 을 추가하자. format string 이므로 뒤에 `, it.price` 를 넣어준다. 

```kotlin
val displayPropertyPrice = Transformations.map(selectedProperty) {
   app.applicationContext.getString(
           when (it.isRental) {
               true -> R.string.display_price_monthly_rental
               false -> R.string.display_price
           }, it.price)
}
```

> This transformation tests whether the selected property is a rental, using the same test from the first task. If the property is a rental, the transformation chooses the appropriate string from the resources with a Kotlin `when {}` switch. Both of these strings need a number at the end, so you concatenate the `property.price` afterwards.



땅 정보도 다르게 표현하기 위해서 아래와 같이 넣어준다. 

```kotlin
val displayPropertyType = Transformations.map(selectedProperty) {
   app.applicationContext.getString(R.string.display_type,
           app.applicationContext.getString(
                   when (it.isRental) {
                       true -> R.string.type_rent
                       false -> R.string.type_sale
                   }))
}
```



fragment_detail.xml 을 열어서 데이터 바인딩을 넣어주자. 

```xml
<TextView
   android:id="@+id/property_type_text"
...
android:text="@{viewModel.displayPropertyType}"
...
   tools:text="To Rent" />

<TextView
   android:id="@+id/price_value_text"
...
android:text="@{viewModel.displayPropertyPrice}"
...
   tools:text="$100,000" />
```

<img src="https://codelabs.developers.google.com/codelabs/kotlin-android-training-internet-filtering/img/3732e88c965ac8c3.png" alt="img" style="zoom:25%;" />

​										

