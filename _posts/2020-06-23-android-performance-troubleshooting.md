---
layout: post
title: "[Udemy 강좌 정리] Troubleshooting Android Performance"
author: "Sara Han"
categories : Android
comments : false
---

* [Troubleshooting Android Performance 강좌 링크](https://www.udemy.com/course/troubleshooting-android-performance/)

# Section 1 : Understanding and Analyzing Memory Usage

**✔ 루프를 최적화 하자** 

```java
    public static boolean isMovieInList(List<Movie> movies, Movie movie) {

        boolean areTheyEqual = false;

        Comparator<Movie> comparator = new Comparator<Movie>() {
            @Override
            public int compare(Movie o1, Movie o2) {
                String name1 = o1.getName();
                String name2 = o2.getName();

                return (name1.equals(name2) ? 0 : 1);
            }
        };

        for (int i = 0; i < movies.size(); i++) {
            Movie movie1 = movies.get(i);
            areTheyEqual = Objects.compare(movie1, movie, comparator) == 0;
        }

        return areTheyEqual;
    }

```

Comparator 객체가 루프를 통과하면서 계속해서 생성된다. 루프의 밖에 놓으면 계속해서 재 생성할 필요가 없어진다. 

💡 Keep your for loops as lean as possible! 



Kotlin 에서 루프 사용하기

* `forEach` 
* `forEachIndexed{ index, string -> ... }` 

💡 Java의 루프보다 훨씬 읽기 편하다. 



**✔ StackOverflowException 을 주의하자** 

안드로이드 앱을 실행하면 `onCreate() -> createLayout() -> initList() -> initHeader() ...` 이렇게 점점 쌓여져 가는 Stack 이 형성된다. 

문제는 Stack 이 100MB로 제한되어 있다는 것이다. 따라서 만약 Stack Limit 을 초과하는 함수를 호출하게 되면 StackOverflowException 이 발생할 수 있다. 

* `java.lang.OutOfMemoryError` 이 발생함. 

대표적으로 (탈줄 조건이 없는) 재귀적으로 호출되는 코드를 작성하거나, 무한 루프를 돌리면 이와같은 문제가 발생한다. 

💡 안드로이드 에서는 `findViewById()` 와 같은 함수의 Cost 가 높다.  



**✔ Loading Images**

* 그냥 로딩하면 안되는 이유? 
  * 사진의 화질이 좋으면 수백 MB의 사이즈를 차지할 수 있다. 
  * 단순히 `setImageResource` 를 호출하면 여러 최적화 작업을 거치게 된다. 예전 안드로이드 기기에서는 메모리 부족 에러가 발생할 수 도 있음. 

```java
...
//좋지 않은 방법 
//        moviesViewHolder.poster.setImageResource(movie.getPoster());

        moviesViewHolder.poster.setImageBitmap(
                Util.decodeSampledBitmapFromResource(context.getResources(), movie.getPoster(), 230, 180));
```

* `context.getResources()` : res 폴더 안에 있는 자원들에 접근할 수 있도록 함. 
* 230, 180 과 같이 W H 를 지정해 준다. 
* 이미지를 더 효율적으로 로딩하긴 하는데 ... 픽셀이 작아지면 이미지가 깨짐. 



Util 클래스 안의 최적화 코드를 확인해 보자. 

* 구글에서 제공한 image optimization 코드임. 

```java
import android.content.res.Resources;
import android.graphics.Bitmap;
import android.graphics.BitmapFactory;

public class Util {


    public static Bitmap decodeSampledBitmapFromResource(Resources res, int resId,
                                                         int reqWidth, int reqHeight) {

        // First decode with inJustDecodeBounds=true to check dimensions
        final BitmapFactory.Options options = new BitmapFactory.Options();
        options.inJustDecodeBounds = true;
        BitmapFactory.decodeResource(res, resId, options);

        // Calculate inSampleSize : 이미지를 어떤 ratio 로 변환하고 싶은건지
        options.inSampleSize = calculateInSampleSize(options, reqWidth, reqHeight);

        // Decode bitmap with inSampleSize set
        options.inJustDecodeBounds = false;
        //새로운 Options 객체를 넣고 decode 한다. 
        return BitmapFactory.decodeResource(res, resId, options);
    }


    private static int calculateInSampleSize(BitmapFactory.Options options, int reqWidth, int reqHeight) {
        // Raw height and width of image
        final int height = options.outHeight;
        final int width = options.outWidth;
        int inSampleSize = 1;

        if (height > reqHeight || width > reqWidth) {
//요청된 사이즈에 도달할 때 까지 2로 나눈다. 
            final int halfHeight = height / 2;
            final int halfWidth = width / 2;

            // Calculate the largest inSampleSize value that is a power of 2 and keeps both
            // height and width larger than the requested height and width.
            while ((halfHeight / inSampleSize) >= reqHeight
                    && (halfWidth / inSampleSize) >= reqWidth) {
                inSampleSize *= 2;
            }
        }

        return inSampleSize;
    }

}

```



💡 **가장 좋은 이미지 로딩 방법 : Glide 라이브러리 사용** 

```java
  Glide.with(context).load(movie.getPoster()).into(moviesViewHolder.poster);
```

* `load()` 함수에는 URL 를 넣을 수 도 있음. 많은 resource 들을 넣을 수 있도록 오버라이딩 되어 있음 



**✔ Memory Profiler 사용하기** 

* 메모리, CPU, 네트워크를 분석하기 위한 툴. 
* heap dump 를 캡처할 수 있고 
* GC 등 여러가지 기능 있음 



**✔ Tracking Allocations** 

![image-20200623132029929]({{site.baseurl}}/assets/img/apt/image-20200623132029929.png)

* Allocations : Heap 에 있는 객체의 수 
  * Android  System, Framework 안에 있는 객체들도 포함 
* Native Size 
* Shallow Size 



<img src="{{site.baseurl}}/assets/img/apt/image-20200623132300665.png" alt="image-20200623132300665" style="zoom: 33%;" />

비트맵이라는 객체를 클릭하면 해당 객체의 참조 depth 를 확인할 수 있다. 



#  Section 2 : Avoiding Memory Leaks 

30 bytes 메모리가 있다고 할때 

<img src="{{site.baseurl}}/assets/img/apt/image-20200623132511260.png" alt="image-20200623132511260" style="zoom:33%;" />

<img src="{{site.baseurl}}/assets/img/apt/image-20200623132519695.png" alt="image-20200623132519695" style="zoom:33%;" />

<img src="{{site.baseurl}}/assets/img/apt/image-20200623132527580.png" alt="image-20200623132527580" style="zoom:33%;" />

위와 같은 방식으로 메모리가 쌓여 나간다고 하자. 



<img src="{{site.baseurl}}/assets/img/apt/image-20200623132544919.png" alt="image-20200623132544919" style="zoom:33%;" />

근데 중간에 int 가 필요 없어진다면 Garbage Collector 가 이를 수거해 간다. 



<img src="{{site.baseurl}}/assets/img/apt/image-20200623132658390.png" alt="image-20200623132658390" style="zoom:33%;" />

유저가 스크롤 다운 하면 위의 비트맵은 더이상 사용되지 않지만 GC가 수거할 수 없다. 



**✔ Leak Canary 사용하기** 

release Implementation, debug Implementation 구분해서 dependency 추가하기 

```groovy
debugImplementation 'com.squareup.leakcanary:leakcanary-android:1.6.3'
releaseImplementation 'com.squareup.leakcanary:leakcanary-android-no-op:1.6.3'
```



코드 추가하기 

```kotlin
...

    override fun onCreate(savedInstanceState: Bundle?) {

        if (LeakCanary.isInAnalyzerProcess(this)) {
            return
        }
        LeakCanary.install(application)

        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_main)
...
```

* 처음 코드가 수행되는 곳에다가 위의 코드를 넣는다. 
* Leaks 라는 새로운 앱이 설치됨. 



Movie 객체를 Main 에서 Detail 로 넘기는 상황

Movie 디테일 Activity 에다가 아래의 함수를 정의하고, 

```java
    public static void launchMovieActivity(Context context, Movie movie) {
        final Intent intent = new Intent(context, MovieActivity.class);
        intent.putExtra("movie", movie);
        context.startActivity(intent);
    }

```



Main 의 어댑터에서 위의 함수 호출 

```java
        moviesViewHolder.poster.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View v) {
                MovieActivity.launchMovieActivity(context, movie);
            }
        });
```



* Main Activity 의 Context 가 leak 되었다고 탐지한다. 



디테일 화면의 레이아웃 MovieView 에서 context 를 참조해야 한다. 

```java
public class MovieView extends FrameLayout {

    private TextView name;
    private ImageView poster;

    public MovieView(Context context) {
        this(context, null);
    }

    public MovieView(Context context, AttributeSet attrs) {
        this(context, attrs, 0);
    }

    public MovieView(Context context, AttributeSet attrs, int defStyleAttr) {
        super(context, attrs, defStyleAttr);
        init(context);
    }
    ...
```



MovieActivity 안에서 MovieView 를 static 으로 선언하면, Activity 가 destroy 되어도 뷰가 참조하는 context 가 살아있게 되므로 memory leak 이 발생한다. 따라서 아래와 같이 인스턴스 변수로 참조하도록 하자. 

```java
MovieView movieView = findViewById(R.id.movieview);
movieView.setName(movie.getName());
movieView.setPosterResource(movie.getPoster());
```





**✔ Common Memory Leaks : Static References** 

Context 클래스는 context 를 유지하는 클래스이다. 뷰가 어떤 액티비티에 있는지 알려면 context 를 알아야 함. 

💡 Context 를 참조하는 것들을 static 으로 선언하지 말 것. 

Movie Activity 에 들어갔다가 나오면, MovieView 는 여전히 살아있을 것이다. 모든 View 를 생성하기 위해서는 Context 가 필요한데, 만약 View 가 계속해서 메모리에 남아있으면 Activity 가 destroy 되어도 View 객체가 메모리에 있을 것이다. 



**✔ 익명 클래스 사용을 조심하자** 

코드를 백그라운드에서 실행해야 하는 경우, 

![image-20200623134314923]({{site.baseurl}}/assets/img/apt/image-20200623134314923.png)

위와 같이 익명 클래스로 만든다면 메모리 누수가 발생한다. 

💡 따라서 별도의 `static class` 로 만들어서 `extends AsyncTask` 와 같이 만들어 주자. 

![image-20200623134403350]({{site.baseurl}}/assets/img/apt/image-20200623134403350.png)



**✔ Non-static Inner class 들을 조심하자**

inner 클래스에서 outer class 의 인스턴스 변수를 참조할 때 메모리 누수가 발생할 수 있음. 

💡 이 문제를 해결하려면 inner class 를 static 으로 만들면 된다. outer class 의 변수를 참조하려면 static inner class 의 생성자로 넘겨주면 된다. 

![image-20200623134655022]({{site.baseurl}}/assets/img/apt/image-20200623134655022.png)





# Section 3 : Managing Cloud Connectivity Issues 

**✔ 백그라운드에서 네트워크 처리하기** 

네트워크 요청은 Blocking IO 이므로 백그라운드 스레드에서 처리해야 한다. 네트워크 속도가 일정하지 않을 수 있기 때문에도 그렇다. 

*`AsyncTask` 를 이용해서 쉽게 백그라운드 스레드에서 네트워크 처리를 할 수 있다. (DEPRECATED)*



**✔ 네트워크 프로파일러로 네트워크 결과 분석하기** 

![image-20200623140310694]({{site.baseurl}}/assets/img/apt/image-20200623140310694.png)

![image-20200623140318943]({{site.baseurl}}/assets/img/apt/image-20200623140318943.png)

다운로드 된 결과가 뭔지도 확인할 수 있다. 

Response 를 더 구체적으로 확인할 수 있어서 유용함. 



**✔ 리퀘스트가 진행되는 동안 앱 사용하기** 

긴 프로그래스가 진행되는 동안 유저에게 이를 알려주어야 한다. 반응을 해줘야 함. 

![image-20200623140538304]({{site.baseurl}}/assets/img/apt/image-20200623140538304.png)

위와 같이 place holder 를 사용해서 로딩중임을 알려주는 이미지를 띄워준다. 

`.error()` , `fallbackImage()` 와 같은 옵션들을 적극적으로 활용하자. 



# Section 4 : UI Analysis and Optimization 

**✔ UI Thread as a Whole** 

UI 스레드는 유저의 클릭 이벤트와 뷰를 띄우는 것들을 처리해준다. 그래서 network, database task 들은 백그라운드 스레드에서 처리하도록 한다. 

<img src="{{site.baseurl}}/assets/img/apt/image-20200623140946537.png" alt="image-20200623140946537" style="zoom:33%;" />

* Main Thread 에서 위의 코드가 돌아가는 것을 Debugger 에서 확인할 수 있다. 



💡 만약 Thread 에다가 이름을 주고 싶으면 Runnable 다음에 이름을 인자 값으로 넘겨주어라. 

![image-20200623141127841]({{site.baseurl}}/assets/img/apt/image-20200623141127841.png)



만약 쓰레드 코드에 Log 를 찍고 싶다면 그냥 `Log.d` 똑같이 찍어주고, `Thread.currentThread().name` 을 호출해주면 현재 스레드의 이름이 나온다. 

![image-20200623141223471]({{site.baseurl}}/assets/img/apt/image-20200623141223471.png)





**✔ View Hierarchies** 

레이아웃은 Root 레이어에서 네스팅 되는 식으로 구성된다. Linear, Relative 와 같은 레이아웃을 사용하면 네스팅 되는 정도가 깊어질 수 있음. 

💡 Constraint Layout 을 사용하면 one level 로 레이아웃을 구성할 수 있음. 

퍼포먼스가 훨씬 좋아진다. 

<img src="{{site.baseurl}}/assets/img/apt/image-20200623141757043.png" alt="image-20200623141757043" style="zoom: 50%;" />

💡 오른쪽 클릭하면 Convert to Constraint Layout 할 수 있음. 



✔ **레이아웃 인스펙터로 레이아웃 디버깅하기** 

Tools -> Layout Inspector 를 누르자. 

에뮬레이터 실행해서 디버깅할 앱을 선택, 디버깅할 액티비티를 선택하자. 

![image-20200623142013557]({{site.baseurl}}/assets/img/apt/image-20200623142013557.png)

위와 같이 레이아웃 인스펙터가 켜진다. 

DecorView 안에 레이아웃 들이 나옴. 

Properties 테이블 안에는 뷰의 속성들에 대한 분석이 나옴 

<img src="{{site.baseurl}}/assets/img/apt/image-20200623142216688.png" alt="image-20200623142216688" style="zoom:50%;" />

XML 에서 만들긴 하지만, kotlin 코드를 통해서 동적으로 뷰들을 바꿀 수 있다. 

💡 동적으로 데이터를 로딩했을 때 UI가 예상과 다르게 나오면 레이아웃의 Snapshot 을 찍어서 수정할 수 있도록 하는 툴임!



💡 만약 UI mock up 을 디자이너에게 받으면 이를 overlay 할 수 있는 기능도 있음. Alpha slider 있어서 투명도 조절도 할 수 있음! 

![image-20200623142433523]({{site.baseurl}}/assets/img/apt/image-20200623142433523.png)





**✔ Using Developer Options to get More Info on your UI** 

에뮬레이터에 디벨로퍼 옵션이 켜져 있을 것이다. 

`디벨로퍼 옵션 키는 방법 : Build Number 에 7번 클릭하기` 

Developer Option 중 유용한 것들 

* Quick Settings developer tiles : Show layout bounds, Profile GPU Rendering ... 
* Show Tabs 도 유용함. 
* Pointer Loaction 도 유용함. 탭의 X Y 좌표에 대한 라인을 그려준다. 
* Force RTL 을 하면 Right to Left 로 레이아웃이 바뀜 



**GPU Overdraw** 

블루, 퍼플 : one layer of GPU rendering 

그린 : 하나의 레이어 위에 또 그려진 것 

레드 : 레이어가 3개 이상인 것을 의미 

💡 레드가 많으면 아주 좋지 않다는 것. 필요하지 않은 레이어를 많이 그리고 있다는 의미임. 

레이어가 너무 많으면 컴퓨테이션이 많아져서 성능이 떨어짐. 



etc. Running Services 를 보면 백그라운드에서 뭐가 돌아가는 지 확인할 수 있음. 어떤 앱이 메모리를 얼만큼 차지하는 지도 확인 가능함. 



# Section 5 : Battery Optimizations 

* 도즈 모드, 앱 스탠드바이 

**Doze Mode** 

안드로이드 6.0 에서부터 등장함. 배터리 수명을 늘려줌. defer 된 태스크들이 완료되도록 하기 위해서 doze 모드를 주기적으로 exit 함. 시스템은 펜딩된 싱크, 잡, 알람들을 수행하고 앱들이 네트워크에 액세스 할 수 있도록 한다. 

시스템은 maintenance window 를 덜 자주 스케줄하고, 이를 통해 배터리 사용을 줄인다. 유저가 폰을 키면 doze mode 가 꺼짐. 

* Maintenance window 가 open up 하면 밀린 작업들을 처리함. 

![image-20200623143656989]({{site.baseurl}}/assets/img/apt/image-20200623143656989.png)

**App Standby** 

이 또한 안드로이드 6에서 소개됨. 최근 사용되지 않은 앱들에 대해서 백그라운드 네트워크 작업들을 defer 함. 앱이 사용되지 않으면 시스템이 앱이 idle 하다고 판단할 수 있도록 함. 

앱이 idle 하다는 판단은 다음과 같이 이루어진다. 

* 사용자가 certain period of time 동안 앱을 만지지 않았다 
* 유저가 명시적으로 앱을 런칭하지 않았다 
* 앱은 foreground 에 활성화된 프로세스가 없다
* 앱은 락 스크린 혹은 알람창에 알림을 띄우지 않는다
* 앱은 활성화된 디바이스 어드민 앱이 아니다



![image-20200623144055617]({{site.baseurl}}/assets/img/apt/image-20200623144055617.png)

배터리 사용을 위의 정책들에 맞게 최적화 해야 한다. 



**✔ Lazy First** 

어떻게 하면 연산을 줄일 수 있는지 탐색하는 것. 

#### 1. Reduce : 캐싱해서 다운로드를 줄일 수 있는가? 

<img src="{{site.baseurl}}/assets/img/apt/image-20200623144256431.png" alt="image-20200623144256431" style="zoom: 33%;" />

#### 2. Defer : 액션을 바로 해야 하는가? 충전 할 때 까지 기다릴 수 있는가? 

<img src="{{site.baseurl}}/assets/img/apt/image-20200623144328775.png" alt="image-20200623144328775" style="zoom:33%;" />

#### 3. Coalesce : 태스크들을 배치로 처리할 수 있는가? 

<img src="{{site.baseurl}}/assets/img/apt/image-20200623144400724.png" alt="image-20200623144400724" style="zoom:33%;" />

> CPU, Radio, Screen 을 많이 사용해야 하는 경우 위의 세 가지를 고려하자. 

![image-20200623144523785]({{site.baseurl}}/assets/img/apt/image-20200623144523785.png)



![image-20200623144534655]({{site.baseurl}}/assets/img/apt/image-20200623144534655.png)

> Battery Historian 

검정색 라인이 베터리 수명임. 

![image-20200623144632063]({{site.baseurl}}/assets/img/apt/image-20200623144632063.png)





✔ **Android : An ecosystem** 

다른 앱들과 조화롭게 사용되어야 한다. 베터리를 너무 많이 잡아먹거나 하면 안됨. 

만약 유저가 베터리와 네트워크를 다른 앱에 사용하고 싶다면 그럴 수 있도록 앱을 만들어야 함. 

💡 알람을 너무 많이 사용하지 말 것. 



![image-20200623144920076]({{site.baseurl}}/assets/img/apt/image-20200623144920076.png)

다크모드와 같은 셋팅들을 지원해줄 것. 알람을 보내는 것 선택할 수 있도록 하기. 

그리고, 유저의 입장에서 생각해보기. 



---



![img](https://udemy-certificate.s3.amazonaws.com/image/UC-c80f8c7e-7604-414d-ba51-4fff080513ff.jpg)
