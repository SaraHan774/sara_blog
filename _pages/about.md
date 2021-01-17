---
layout: post
title: "한가희"
author: "Sara Han"
permalink: "/about"
---

<img src="./assets/img/2020_me_2.jpg" width="300" style="border-radius: 50%"/>

> _반갑습니다, 한가희입니다. 세상에 필요한 안드로이드 개발자가 되기 위해 노력하는 중 입니다._

<br>
2014년 겨울 안드로이드와의 인연이 시작되었습니다. 스타트업을 하는 선배들을 동경하는 마음에 "나도 앱을 만들어볼까?"라는 생각이었습니다. 하지만 예상보다 길어진 대학 입시 탓에 안드로이드를 잊고 살다가 2018년 여름 헬스케어 AI스타트업에서 3주간 경쟁사 보고서를 작성하는 인턴을 하며 말로만 듣던 4차 산업혁명을 몸소 체감하였습니다. 뿐만 아니라 당시 대만에서 가까웠던 친구가 기숙사에서 스타트업을 하는 것을 보고 마음속의 열정이 다시 살아났습니다. 이러한 일련의 일들을 계기로 IT직군에 큰 매력을 느꼈고, 제가 걸어온 길과 전혀 다른 길을 2년 째 걸어오는 중입니다. 외국어 고등학교를 졸업하고 어문계열을 전공했던 만큼 익숙하고 편한 길이 있었지만 저는 제가 생각하는 미래를 위해서 기꺼이 익숙함을 떨쳐낸 바 있습니다. 앞으로도 세상의 변화에 발 맞추어 무한히 진화하는 사람이 되고 싶습니다.
<br><br>

<script src="//code.jquery.com/jquery-3.2.1.min.js"></script>
<script>
function copyToClipboard(val) {
  var t = document.createElement("textarea");
  document.body.appendChild(t);
  t.value = val;
  t.select();
  document.execCommand('copy');
  document.body.removeChild(t);
}
function gmailClickToCopy(){
  copyToClipboard('sarahan774@gmail.com');
  alert('Email address is copied to your clipboard!\n이메일이 클립보드에 복사 되었습니다.');
};
</script>

<br/>

## Contact

<table style="width:100%">
  <tr>
    <th>Gmail</th>
    <th>Github</th>
    <th>Linkedin</th>
  </tr>
  <tr>
    <td>
    <img id="gmailImage" src="./assets/img/gmail.png" width = "30" height = "30" onclick="gmailClickToCopy();">
    </td>
    <td>
    <a href = "https://github.com/SaraHan774">
        <img src="./assets/img/github-image.png" width = "30" height = "30">
        </a>
    </td>
    <td>
    <a href = "https://www.linkedin.com/in/gahee-han-sara">
        <img src="./assets/img/linkedin.png" width = "30" height = "30">
        </a>
    </td>
  </tr>
</table>

<br>

## Education

  <table width="100%">
    <tr>
      <td><a href = "https://www.hanyang.ac.kr/web/eng">
      <img src="./assets/img/HYU_symbol.png" width="120" height = "120">
        </a>
      </td>
      <td><a href ="http://eng.dwfl.hs.kr/">
      <img src="./assets/img/daewon.png" width="120" height = "120"></a></td>
    </tr>
  </table>

#### 한양대학교 중어중문학과 학사 (2021.08 졸업 예정)

- 졸업 평점 3.88/4.5 (2021년 1월 기준)
- 2019.07 ~ 2019.08 홍콩 이공대학교 계절학기 수료
- 2018.02 ~ 2018.07 대만 칭화대학교 교환학생
- 2017.09 ~ 2018.01 중국 칭화대학교 학점교류생
- 성적 우수 장학금 수여 3학기
- **2016.03** 입학

- **컴퓨터소프트웨어학과 및 정보시스템학과 수업 이수**

  - C++ Programming
  - Unix System Programming
  - DBMS
  - Discrete Mathematics

- **기타 프로그래밍 관련 수업 이수**
  - Programming for Engineers (Python)
  - Basic Web Programming with JavaScript & HTML & CSS
  - Data Visualization with Python

#### 대원외국어고등학교 28기 졸업 (2011.03 - 2014.02)

- 일본어 전공
- WHAF (World Hope Asia & Africa Foundation) 봉사 단체 2대 회장

## Experience

#### 스타트업 PenguinLAB.INC 의 여행 동행 어플 베타버전 설계 및 구현

<script id="asp-embed-script" data-zindex="1000000" type="text/javascript" charset="utf-8" src="https://spark.adobe.com/page-embed.js"></script><a class="asp-embed-link" href="https://spark.adobe.com/page/2t33j2qmPiNKa/" target="_blank"><img src="https://spark.adobe.com/page/2t33j2qmPiNKa/embed.jpg?buster=1610853517612" alt="여행 동행이 필요할 때, &quot;트래곤&quot;" style="width:100%" border="0" /></a>

- [구글 플레이스토어 링크](https://play.google.com/store/apps/details?id=com.penguinlab.tragon)
- 외주 기간 : 2020-09 ~ 2020-12
- 사용한 기술 : Android, Kotlin, DI with Hilt, Jenkins, Postman, Swagger
- 협업 툴 : Github, Slack, Notion, Zeplin
- 프로젝트 내용
  - Zeplin 으로 시안을 받았으며 모든 화면들을 스스로 구현하였습니다.
  - Node.js 백엔드 서버와 연동된 로그인 구현 (SMS 전화번호 인증 포함)
  - MVI 아키텍처 적용
  - Kotlin 의 Flow 와 Channel 을 이용한 네트워킹 추상 레이어 구현
  - Hilt 를 이용한 의존성 주입
  - Jenkins 를 이용한 구글 플레이스토어 배포
  - 서버 개발자, iOS 개발자, 디자이너, 마케터와 협업

#### IT서비스 개발 동아리 프로그라피 활동 : 애견호텔 예약 어플 설계 및 구현

<script id="asp-embed-script" data-zindex="1000000" type="text/javascript" charset="utf-8" src="https://spark.adobe.com/page-embed.js"></script><a class="asp-embed-link" href="https://spark.adobe.com/page/th9cMGM2Xmrlo/" target="_blank"><img src="https://spark.adobe.com/page/th9cMGM2Xmrlo/embed.jpg?buster=1610854378518" alt="애견호텔 예약은 &quot;마이펫밀리&quot;" style="width:100%" border="0" /></a>

- [구글 플레이스토어 링크](https://play.google.com/store/apps/details?id=com.prography.pethotel&hl=ko)
- [프로그라피 링크](http://prography.org/)
- 프로젝트 기간 : 2020-03 ~ 2020-07
- 사용한 기술 : Android, Kotlin
- 협업 툴 : Github, Slack, Notion, Jira, Postman
- 프로젝트 내용
  - UI 직접 디자인
  - 카카오톡 로그인 및 일반 로그인 구현
  - 동물등록번호 공공 API 연동
  - 메인 화면에서 가격별, 위치별 장소 필터링 구현
  - 예약 및 예약 확인 화면 구현
  - 장소 디테일 화면 구현
  - 리뷰 남기기 기능 구현
  - MVVM 아키텍처 기반의 설계
  - Kotlin 의 Coroutine과 안드로이드의 ViewModel, LiveData 등을 이용한 설계

#### 한양대학교 프로그래밍 동아리 FORIF 소속 안드로이드반 멘토 활동

- 활동 기간 : 2019-09 ~ 2019-12
- 사용한 기술 : Android, Java
- 협업 툴 : Github, Slack
- 활동 내용
  - 총 7회에 걸친 오프라인 강의
    - Java 기초 문법
    - Java 의 어떤 부분들이 안드로이드 프레임웍에서 중요하게 활용되는지
    - Java 디자인 패턴 중 어떤 부분들이 안드로이드 프레임웍에서 자주 등장하는지
    - 안드로이드 레이아웃 기초
    - 안드로이드 아키텍처가 왜 필요한지
    - 안드로이드 네트워킹
    - 안드로이드 로컬 DB 활용하기
    - [강의 자료 링크](https://github.com/SaraHan774/android-study-forif)
  - 학기 말 해커톤 지도
    - 먹방 룰렛 어플 '왓냠'의 제작을 지도하였습니다.
    - [해커톤 앱 깃헙 레포](https://github.com/SaraHan774/WATNYAM)

#### Udacity 의 안드로이드 개발 나노디그리 취득

- [수료증 링크](https://confirm.udacity.com/J54R3E7)
- 이수 기간 : 2019-02 ~ 2019-07
- 사용한 기술 : Android, Java, Firebase, Espresso UI Testing, Google Cloud Platform, etc.
- 학습 내용
  - 안드로이드의 레이아웃 및 기초 개념
  - 안드로이드에서의 HTTP 통신 및 로컬 DB 활용 방법
    - 관련 프로젝트 : [영화 포스터 디스플레이 앱](https://github.com/SaraHan774/PopularMoviesStage2)
  - 위젯 만들기, Tablet 버전 UI 만들기, SimpleExoPlayer 활용하기 등
    - 관련 프로젝트 : [베이킹 레시피 앱](https://github.com/SaraHan774/BakingApp_1)
    - 설명 : Udacity 에서 제공해주는 mp4 파일을 ExoPlayer 로 디스플레이 하여 레시피
      영상들을 디스플레이 하는 앱
  - 배운 것들을 활용한 캡스톤 프로젝트
    - 관련 프로젝트 : [Read Simple Slider](https://github.com/SaraHan774/rss-v1)

#### 삼성 멀티캠퍼스 '안드로이드 & Java' 취업 교육 이수

- [수료증 및 우수상 링크](https://spark.adobe.com/post/1byXgf5MGAASB/)
- 이수 기간 : 2018-07 ~ 2018-08
- 사용한 기술 : Android, Java, SQL
- 학습 내용
  - Java 기초 문법 및 객체지향 프로그래밍 개념
  - 안드로이드 기초 개념
  - 마지막 2주간 팀 프로젝트 진행

## 기타

- 리눅스 (Ubuntu, CentOS)환경 기반 개발 경험 있음
- Git 을 이용한 버전관리 능숙
- 업무와 일상 생활에 지장 없는 **영어** 실력

  - OPIc IH (2018.07)
  - TOEIC 스피킹 200/200 (2018.06)
  - TOEIC 985/990 (2018.02)

- 중급 **중국어** 실력
  - HSK 6급 249/300 (2018.01)

## Past Projects

#### **Assignments / Projects**

- **Scrawling Twitter data with Python (2018.11)**

  - Data scrolling & visualization with tweepy, matplotlib, pandas, konlpy, beautiful soup etc.

- **Simple quiz web app with HTML & javascript (2018.10)**

<br>
#### **Udacity Android Developer Nanodegree Projects**

![read-simple-slider](../assets/img/ReadSimpleSlider.png){:class="img-responsive"}

> Capstone Project : Read Simple Slider App

- **Completed 5 Toy Projects on Fundamental concepts of Android & 1 Capstone Project (2019.01 - 2019.07)**
- Built Apps that include :
  - Network connection for fetching JSON & XML data using Volley, Retrofit library, AsyncTaskLoader, etc.
  - Video display using SimpleExoPlayer
  - LiveData with MVVM architecture
  - Firebase AdMob, Analytics, Real Time Database
  - Paid & Free version configuration in Gradle
  - Google Cloud Endpoint
  - Material Design Specifications
  - Tablet version of the app
  - ListView Widget
  - Room database library
  - UI testing with espresso
  - etc.

#### **Others**

![movie-Database](../assets/img/MovieDatabase.png){:class="img-responsive"}

> Movie Database App made for teaching mentees.

- **Sample App Made for Teaching School Mentees : Movie Database App (2019.08)**
  - Simple Movie Poster App that displays movie information fetched from movie site API
  - Includes Room DB, LiveData, ViewModel
  - Has like button, comment & sharing function

<br><br><br>

<div class="video-container">
  <iframe width="560" height="315" src="https://www.youtube.com/embed/aK5KGjFnbSs" frameborder="0" allow="accelerometer; autoplay; encrypted-media; gyroscope; picture-in-picture" allowfullscreen>
  </iframe>
</div>
> Naver D2 mini DevFest 출품작 : The Slider App

- **Naver D2 mini DevFest Competition : Live Slider App (2019.07)**
  - Simple RSS feed that displays news fetched with retrofit2
  - Live slider with Timer & ViewPager
  - Includes **python flask** server for analysing images with **Google Vision API** and integrated the results with the search bar.

<br/><br/>

#### Art works

- Android 전광판 :)
  <img src="./assets/img/android.jpg" width="300"/>
