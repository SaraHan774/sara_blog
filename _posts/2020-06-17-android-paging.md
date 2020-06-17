---
layout: post
title: "Android Codelab 정리 : Paging (1)"
author: "Sara Han"
categories : Android
comments : false
---


# Android Paging (1)

**안드로이드 페이징 코드랩 정리**

* [Codelab : Android Paging](https://codelabs.developers.google.com/codelabs/android-paging/index.html?index=..%2F..%2Findex#0)

* [Documentation : Paging](https://developer.android.com/topic/libraries/architecture/paging)


### Paging Library Main Components 

* `PagingData` : paginated data 를 위한 container. 데이터를 refersh 하면 별도의 상응하는 `PagingData` 를 갖게 된다. 
* `PagingSource` : PagingData 의 스트림에다가 데이터의 sanpshot 을 로딩하는 base class 이다. 
* `Pager.flow` : `PagingConfig` 에 기초하여 `Flow<PagingData>` 를 만들고, implement 된 `PagingSource` 를 어떻구 구현하는지에 관한 function 을 만든다. 
* `PagingDataAdapter` : RecyclerView 에다가 `PagingData` 를 present 하는 `RecyclerView.Adapter` 이다. `PagingDataAdapter` 는 Kotlin `Flow`, `LiveData`, 그리고 RxJava 의 `Flowable` 혹은 `Observable` 과 연결될 수 있다. 
  * `PagingDataAdapter` 는 페이지들이 로드되면 내부의 `PagingData` 의 로딩 이벤트를 listen 한다. 그리고 `DiffUtil` 을 백그라운드 스레드에서 기동하여 업데이트된 내용들을`PagingData` 객체의 형태로 받을 수 있도록 한다. 
* `RemoteMediator` : network 와 database 로부터 pagination 을 구현하도록 도와준다. 



### Define the source of data 

`PagingSource` 구현은 데이터의 source 를 정의하고, 그 source 로부터 어떻게 데이터를 회수하는지 정의한다. 

`PagingData` 객체는 `PagingSource` 로부터 데이터를 query 하고, 이는 유저가 RecyclerView 를 스크롤 함에 따라서 발생하는 loading hint 로부터 트리거된다. 

현재 예시에서는 GithubRepository 가 많은 책임을 떠안고 있다. 

* 데이터를 GithubService 로부터 로드하고, 여러개의 request 가 동시에 트리거 되지 않도록 한다. 
* 회수된 데이터에 대한 in memory cache 를 유지한다. 
* 요청될 페이지에 대한 정보를 유지한다. 



`PagingSource` 를 만들기 위해서 아래의 세 가지를 정의해야 한다 : 

* Paging Key 의 Type : 예시의 경우에서, Github API 는 1-based index number 를 페이지에 사용한다. 따라서 type 은 Int 이다. 
* Load 될 데이터의 Type : 예시의 경우 Repo 아이템을 로드한다. 
* 어디서 데이터가 retrive 되는지 : `GithubService` 에서 데이터를 받아온다. 데이터 소스는 특정 쿼리에 한덩될 것이므로 우리는 GithubService 에 쿼리를 같이 넘겨줄 수 있도록 해야한다. 



`data` 패키지 에다가 `PagingSource` 의 구현을 만들자.

```kotlin
class GithubPagingSource(
        private val service: GithubService,
        private val query: String
) : PagingSource<Int, Repo>() { 
    override suspend fun load(params: LoadParams<Int>): LoadResult<Int, Repo> {
        TODO("Not yet implemented")
    }
}
```

* `load` 함수를 구현해야 한다. 이 함수는 async load 를 트리거 하기 위해 호출될 것이다. `LoadParams` 객체는 load operation 과 관련된 정보들을 유지한다, including the following : 
  * Key of the page to be loaded : 만약 load 가 처음 호출된 경우면 LoadParams.key 는 null 일 것이다. 이 경우, initial page key 를 정의해주어야 한다. 
  * Load Size : 몇 개의 아이템을 로드할지 요청된 아이템들의 개수



* Load 함수는 `LoadResult` 를 반환한다. 이것은 `RepoSearchResult` 의역할을 대체할 것이다. 왜냐하면 `LoadResult` 가 다음 중 하나의 타입을 가질 수 있기 때문이다. 
  * `LoadResult.Page` 만약 결과가 Successful 하다면 
  * `LoadResult.Error` 에러의 경우. 

```kotlin
/**
 * RepoSearchResult from a search, which contains LiveData<List<Repo>> holding query data,
 * and a LiveData<String> of network error state.
 */
data class RepoSearchResult(
    val data: LiveData<List<Repo>>,
    val networkErrors: LiveData<String>
)
```

> 위의 코드가 LoadResult 하나로 필요 없어짐. 



`LoadResult.Page` 를 생성할 때 만약 리스트가 상응하는 방향으로 로드될 수 없다면 nextKey 혹은 prevKey 에 null 을 넘겨주어라. 

예를 들어, 우리의 경우, 네트워크 응답이 성공적일 수 도 있지만 리스트가 비어있다. 우리는 로드할 데이터가 없는 것이다. 따라서 nextKey 는 null 이다. 

GithubPagingSource 는 다음과 같이 작성할 수 있다. 

```kotlin
// GitHub page API is 1 based: https://developer.github.com/v3/#pagination
private const val GITHUB_STARTING_PAGE_INDEX = 1

class GithubPagingSource(
        private val service: GithubService,
        private val query: String
) : PagingSource<Int, Repo>() {

    override suspend fun load(params: LoadParams<Int>): LoadResult<Int, Repo> {
        val position = params.key ?: GITHUB_STARTING_PAGE_INDEX
        val apiQuery = query + IN_QUALIFIER
        return try {
            val response = service.searchRepos(apiQuery, position, params.loadSize)
            val repos = response.items 
            LoadResult.Page(
                    data = repos,
                    prevKey = if (position == GITHUB_STARTING_PAGE_INDEX) null else position - 1,
                    nextKey = if (repos.isEmpty()) null else position + 1
            )
        } catch (exception: IOException) {
            return LoadResult.Error(exception)
        } catch (exception: HttpException) {
            return LoadResult.Error(exception)
        }
    }
}
```





### Build and Configure PagingData 

현재 우리의 구현에서는 `Flow<RepoSearchResult>` 를 사용해서 ViewModel 에다가 데이터를 전달해준다. ViewModel 은 데이터를 받아서 LiveData 로 Transform 하고, UI에 이를 노출한다. 

디스플레이된 리스트의 끝에 도달하면 더 많은 데이터가 네트워크로부터 로드되고, `Flow<RepoSearchResult>` 는 이전에 회수된 데이터에다가 최근 로드된 데이터까지 포함하고 있게 된다. 



`RepoSearchResult` 는 success 과 error case 두 가지 모두를 encapsulate 한다. 성공 케이스는 repository data 를 갖고 있다. error case 는 Exception 로직음 담게 된다. 

Paging 3.0 부터는 `RepoSearchResult` 가 더 이상 필요 없다. 왜냐하면 라이브러리는 LoadResult 로 성공, 에러 두가지 경우를 모델링 하기 때문이다. 



`PagingData` 를 생성하기 위해서, 어떤 API를 사용해서 `PagingData` 를 앱의 다른 레이어들에 전달할 것인지 결정해야 한다. 

![image-20200617141838235]({{site.baseurl}}/assets/img/image-20200617141838235.png)

현재 우리는 이미 Flow 를 앱에서 사용하고 있으므로 이 접근 방식을 유지할 것이다.

`Flow<PagingData<Repo>>` 와 같이 고쳐주기만 하면 된다. 



어떤 `PagingData` 빌더를 사용하든간에, 당신은 다음의 매개변수들을 전달해 주어야 한다 : 

* `PagingConfig` : 이 클래스는 `PagingSource` 로부터 어떻게 데이터를 로드할 것인지에 대한 옵션을 설정할 수 있게 해준다. 예를 들어 몇개까지 미리 로드 할 것인지, 처음 로드할때는 얼만큼의 사이즈를 요청할 것인지 등등이 있다. 필수 매개변수는 Page size 이다. Page Size 는 각 페이지에 몇 개의 아이템이 로드될 것인지에 관한 정보이다. default 로 paging 은 로드한 모든 페이지들을 메모리에 유지할 것이다. 유저가 scroll 함에 따라서 메모리를 낭비하지 않도록 `maxSize` 매개변수를 `PagingConfig` 에 설정하도록 하자. 
* `PagingSource` 를 어떻게 생성하는지에 관한 function : 우리의 경우, 우리는 새로운 쿼리가 들어올 때마다 새로운 `PagingSource` 를 생성할 것이다. 



### 주의 

* `PagingConfig.pageSize` 는 수 페이지 정도 분량의 아이템의 개수로 정의해주는게 바람직하다. 만약 페이지가 너무 적으면 리스트가 깜빡거릴 수 도 있다. (as pages' content does not cover the full screen.) 큰 페이지 사이즈는 로딩 효율에 좋지만 리스트가 업데이트 되면 latency 가 발생할 수 있다. 
* default 로 `PagingConfig.maxSize` 는 unbounded 되어있다. 따라서 페이지들은 절대 drop 되지 않는다. (메모리에서 제거되지 않음) 따라서 만약 page 들을 drop 하고 싶다면 maxSize 를 충분히 큰 숫자로 지정하도록 하자. 최소 값은 `pageSize + prefecthDistance * 2` 이다. 



### GithubRepository 를 수정하자. 

```kotlin
fun getSearchResultStream(query: String): Flow<PagingData<Repo>> {
    return Pager(
          config = PagingConfig(pageSize = NETWORK_PAGE_SIZE),
          pagingSourceFactory = { GithubPagingSource(service, query) }
    ).flow
}
```

* Pager.flow 를 사용한다. 



### GithubRepository 를 클린업 하자. 

Paging 3.0 은 많은 것들을 우리에게 제공한다 : 

* in-memory cache 를 핸들링 해주고 
* 유저가 리스트의 끝에 가까이 가면 데이터를 요청하도록 한다. 

이뜻은 현재 우리의 GithubRepository 에 있는 모든 것들을 지워도 된다는 것이다. 아래와 같이 수정하자. 

```kotlin
class GithubRepository(private val service: GithubService) {

    fun getSearchResultStream(query: String): Flow<PagingData<Repo>> {
        return Pager(
                config = PagingConfig(pageSize = NETWORK_PAGE_SIZE),
                pagingSourceFactory = { GithubPagingSource(service, query) }
        ).flow
    }

    companion object {
        private const val NETWORK_PAGE_SIZE = 50
    }
}
```



> REF. 원래의 레포지토리 클래스 

```kotlin
package com.example.android.codelabs.paging.data

import android.util.Log
import androidx.lifecycle.MutableLiveData
import com.example.android.codelabs.paging.api.GithubService
import com.example.android.codelabs.paging.api.searchRepos
import com.example.android.codelabs.paging.db.GithubLocalCache
import com.example.android.codelabs.paging.model.RepoSearchResult

/**
 * Repository class that works with local and remote data sources.
 */
class GithubRepository(
    private val service: GithubService,
    private val cache: GithubLocalCache
) {

    // keep the last requested page. When the request is successful, increment the page number.
    private var lastRequestedPage = 1

    // LiveData of network errors.
    private val networkErrors = MutableLiveData<String>()

    // avoid triggering multiple requests in the same time
    private var isRequestInProgress = false

    /**
     * Search repositories whose names match the query.
     */
   //페이징 라이브러리가 대신 해준다. 
    fun search(query: String): RepoSearchResult {
        Log.d("GithubRepository", "New query: $query")
        lastRequestedPage = 1
        requestAndSaveData(query)

        // Get data from the local cache
        val data = cache.reposByName(query)

        return RepoSearchResult(data, networkErrors)
    }

    fun requestMore(query: String) {
        requestAndSaveData(query)
    }
//페이징 라이브러리가 대신 처리해준다 (캐싱)
    private fun requestAndSaveData(query: String) {
        if (isRequestInProgress) return

        isRequestInProgress = true
        searchRepos(service, query, lastRequestedPage, NETWORK_PAGE_SIZE, { repos ->
            cache.insert(repos) {
                lastRequestedPage++
                isRequestInProgress = false
            }
        }, { error ->
            networkErrors.postValue(error)
            isRequestInProgress = false
        })
    }

    companion object {
        private const val NETWORK_PAGE_SIZE = 50
    }
}

```





### Request and cache PagingData in the ViewModel 

우리의 SearchRepositoriesViewModel 로부터 우리는 repoResult 를 노출시킨다 :`LiveData<RepoSearchResult>`. 

repoResult 의 역할은 검색 결과들에 대한 인메모리 캐시로서 작동하며, Configuration change 를 견디는 것이다. 

Paging 3.0 부터는 Flow 를 LiveData 로 변환하지 않아도 된다. 대신 SearchRepositoriesViewModel 은 private `Flow<PagingData<Repo>>` 를 멤버변수로 갖게 될 것이고 이는 repoResult 와 마찬가지의 역할을 수행한다. 

새로운 쿼리마다 다른 LiveData 객체를 사용하는 대신 우리는 그냥 String 을 사용할 수 있다. 이런 방식은 새로운 검색 쿼리가 들어왔을 때 현재 쿼리와 같다면 그냥 존재하는 Flow 를 리턴하도록 보장해준다. repository.getSearchResultStream() 을 호출해서 쿼리가 기존 쿼리와 다른지 확인할 수 있다. 

`Flow<PagingData>` 는 `cachedIn()` 메소드를 제공한다. 이 메소드는 `CoroutineScope` 안에서 `Flow<PagingData>` 를 캐싱할 수 있도록 도와준다.  ViewModel 안이므로 `viewModelScope` 를 사용하자. 

> **Note:** If you're doing any operations on the `Flow`, like `map` or `filter`, make sure you call `cachedIn` after you execute these operations to ensure you don't need to trigger them again.



ViewModel 안의 searchRepo 함수를 아래와 같이 수정하자. 

```kotlin
class SearchRepositoriesViewModel(private val repository: GithubRepository) : ViewModel() {

    private var currentQueryValue: String? = null

    private var currentSearchResult: Flow<PagingData<Repo>>? = null

    fun searchRepo(queryString: String): Flow<PagingData<Repo>> {
        val lastResult = currentSearchResult
        if (queryString == currentQueryValue && lastResult != null) {
            return lastResult
        }
        currentQueryValue = queryString
        val newResult: Flow<PagingData<Repo>> = repository.getSearchResultStream(queryString)
                .cachedIn(viewModelScope)
        currentSearchResult = newResult
        return newResult
    }
}
```

Now, let's see what changes we made to `SearchRepositoriesViewModel`:

- Added the new query `String` and search result `Flow` members.
- Updated the `searchRepo()` method with the functionality described previously.
- **Removed `queryLiveData` and `repoResult` as their purpose is covered by Paging 3.0 and `Flow`.**
- Removed the `listScrolled()` since the Paging library will handle this for us.
- Removed the `companion object` because `VISIBLE_THRESHOLD` is no longer needed.
- Removed `repoLoadStatus`, since Paging 3.0 has a mechanism for tracking the load status as well, as we'll see in the next step.



기존의 ViewModel 클래스 

```kotlin
class SearchRepositoriesViewModel(private val repository: GithubRepository) : ViewModel() {

    companion object {
        //필요 없음 
        private const val VISIBLE_THRESHOLD = 5
    }

    //페이징 라이브러리로 인해 필요 없어짐
    private val queryLiveData = MutableLiveData<String>()
    //페이징 라이브러리로 인해 필요 없어짐
    //Flow<PagingData<Repo>> 가 처리해준다. 
    private val repoResult: LiveData<RepoSearchResult> = Transformations.map(queryLiveData) {
        repository.search(it)
    }

    val repos: LiveData<List<Repo>> = Transformations.switchMap(repoResult) { it -> it.data }
    val networkErrors: LiveData<String> = Transformations.switchMap(repoResult) { it ->
        it.networkErrors
    }

    /**
     * Search a repository based on a query string.
     */
    //검색할 때 페이징 라이브러리를 동원해서 하므로 searchRepo 만 
    //페이징 라이브러리의 설명에 맞게 고쳐주면 됨. 
    fun searchRepo(queryString: String) {
        queryLiveData.postValue(queryString)
    }

    //페이징 하는 것 따로 작성해주지 않아도 됨. 
    fun listScrolled(visibleItemCount: Int, lastVisibleItemPosition: Int, totalItemCount: Int) {
        if (visibleItemCount + lastVisibleItemPosition + VISIBLE_THRESHOLD >= totalItemCount) {
            val immutableQuery = lastQueryValue()
            if (immutableQuery != null) {
                repository.requestMore(immutableQuery)
            }
        }
    }

    /**
     * Get the last query value.
     */
    fun lastQueryValue(): String? = queryLiveData.value
}
```



### Make the Adpater work with PagingData 

`PagingData` 를 RecyclerView 에다가 바인드 하기 위해서는 `PagingDataAdpater` 를 사용하도록 하자. **`PagingDataAdapter` 는 `PagingData` 의 내용이 로드되고 그것이 RecyclerView 를 업데이트 하라고 시그널 할 때마다 notify 된다.** 



기존의 RecyclerView Adapter 

```kotlin
package com.example.android.codelabs.paging.ui

import android.view.ViewGroup
import androidx.recyclerview.widget.DiffUtil
import androidx.recyclerview.widget.ListAdapter
import com.example.android.codelabs.paging.model.Repo

/**
 * Adapter for the list of repositories.
 */
class ReposAdapter : ListAdapter<Repo, androidx.recyclerview.widget.RecyclerView.ViewHolder>(REPO_COMPARATOR) {

    override fun onCreateViewHolder(parent: ViewGroup, viewType: Int): androidx.recyclerview.widget.RecyclerView.ViewHolder {
        return RepoViewHolder.create(parent)
    }

    override fun onBindViewHolder(holder: androidx.recyclerview.widget.RecyclerView.ViewHolder, position: Int) {
        val repoItem = getItem(position)
        if (repoItem != null) {
            (holder as RepoViewHolder).bind(repoItem)
        }
    }

    companion object {
        private val REPO_COMPARATOR = object : DiffUtil.ItemCallback<Repo>() {
            override fun areItemsTheSame(oldItem: Repo, newItem: Repo): Boolean =
                    oldItem.fullName == newItem.fullName

            override fun areContentsTheSame(oldItem: Repo, newItem: Repo): Boolean =
                    oldItem == newItem
        }
    }
}

```

`PagingData` stream 을 이용할 수 있도록 어댑터를 수정해주자. 현재의 어댑터는 ListAdapter 를 구현하고 있는데, 이를 `PagingDataAdapter` 를 구현하도록 수정한다. 

나머지 코드들은 수정할 필요가 없다

```kotlin
class ReposAdapter : PagingDataAdapter<Repo, RecyclerView.ViewHolder>(REPO_COMPARATOR) {
// body is unchanged
```

---

나머지 부분들은 다음 편에서 계속 ... 

* [여기서부터 계속 이어집니다](https://codelabs.developers.google.com/codelabs/android-paging/index.html?index=..%2F..%2Findex#8)