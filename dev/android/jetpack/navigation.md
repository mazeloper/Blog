---
description: AAC Jetpack Navigation Component
---

# Navigation

Android Architecture Component Library 중 하나인 `Navigation` 에 대해서 끄젹어봅니다🧐

![https://developer.android.com/guide/navigation/navigation-design-graph](https://developer.android.com/static/images/topic/libraries/architecture/navigation-design-graph-top-level.png)

### Navigation 이란?

Jetpack 중 하나인 Navigation Component 는 액티비티나 프래그먼트 간의 이동을 안정적이고 간단하게 도와주는 컴포넌트이다. 또한 화면간의 이동을 iOS 의 storyboard 처럼 직관화할 수 있어 어플리케이션의 화면흐름을 쉽게 알 수 있다.\
\
Android Document에 명시된 네비게이션 도입으로 인한 장점은 아래와 같다.

* 프래그먼트 트랜잭션 처리
* 화면간의 이동을 올바르게 처리
* 애니메이션과 전에 대한 표준화된 리소스 제공
* 딥 링크 구현 및 처리 용이
* 최소한의 추가작업으로 상단 툴바/ 하단 네비게이션을 포함
* Safe Args - 화면 전환간의 데이터 전달을 전달할 때 안정성을 제공
* ViewModel 지원

#### # 구성요소

> #### Navigation Graph
>
> 모든 네비게이션 관련 정보가 하나의 모여있는 XML 리소스이다.\
> 어플리케이션 내 화면간의 이동에 대해 플로우를 직관화하여 한 눈에 알아보기 쉽게 확인할 수 있다.
>
> #### NavHost
>
> `Navigation Graph`에서 대상을 표시하는 빈 컨테이너이다.\
> 구성요소에는 프래그먼트 대상을 표시하는 기본 `NavHost` 구현인 `NavHostFragment` 가 포함된다.
>
> #### NavController
>
> `NavHost` 에서 앱 탐색을 관리하는 객체이다.\
> `NavController` 는 사용자가 앱 내에서 이동할 때 `NavHost` 에서 대상 화면의 전환을 조종한다

#### # 구현

\- **Navigation 라이브러리 추가** (build.gradle)

```kotlin
def nav_version = "2.5.0"

implementation "androidx.navigation:navigation-fragment-ktx:$nav_version"
implementation "androidx.navigation:navigation-ui-ktx:$nav_version"
```

\- **Navigation Graph 추가**

1. res 디렉토리에서 _New > Android Resource File_ 을 선택한다.
2. 파일명 입력 (ex: nav\_graph)
3. _ResourceType_ 설정을 _Navigation_ 으로 선택한다.

\- **NavHost 추가** (activity\_main.xml)

```xml
<?xml version="1.0" encoding="utf-8"?>
<androidx.constraintlayout.widget.ConstraintLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    tools:context=".MainActivity">


    <androidx.fragment.app.FragmentContainerView
        android:id="@+id/nav_container"
        android:name="androidx.navigation.fragment.NavHostFragment"
        android:layout_width="0dp"
        android:layout_height="0dp"
        app:defaultNavHost="true"
        app:layout_constraintBottom_toBottomOf="parent"
        app:layout_constraintEnd_toEndOf="parent"
        app:layout_constraintStart_toStartOf="parent"
        app:layout_constraintTop_toTopOf="parent"
        app:navGraph="@navigation/nav_graph" />

</androidx.constraintlayout.widget.ConstraintLayout>
```

액티비티에 프래그먼트들의 컨테이너 역할을 하는 `NavHost` 구현인 `NavHostFragment` 를 추가한다.

`android:name` 속성은 NavHost 구현의 클래스 이름을 포함한다.

`app:navGraph` 속성은 NavHostFragment 를 Navigation Graph`(`탐색 그래프) 와 연결한다.\
탐색 그래프는 사용자가 이동할 수 있는 이 NavHostFragment 의 모든 대상을 지정한다

`app:defaultNavHost` 속성을 `true`로 설정하면 NavHostFragment 가 시스템 뒤로가기 버튼을 가로챕니다.\
&#x20;동일한 레이아웃에 여러 호스트가 있다면(예: 창이 2개인 레이아웃) 한 호스트만 기본 NavHost로 지정해야 한다.\
\
\- **Navigation Graph 대상 추가**\
\
이제 네비게이션에서 사용할 프래그먼트들을 nav\_graph.xml 에 추가한다.

nav\_graph.xml 파일을 열고 아래와 사진에 표시된 **버튼**을 클릭하면 프래그먼트 클래스 목록이 노출되며 더블 클릭 시,\
그래프에 해당 프래그먼트가 추가된다.

![](../../../.gitbook/assets/nav\_graph\_1.png)

![프래그먼트를 그래프에 처음 추가하게 되면 tools:layout 속성을 제외하고 위와 같은 소스가 추가된다.](<../../../.gitbook/assets/스크린샷 2022-07-22 오후 2.14.56.png>)

`app:startDestination` 속성은 navHost로 지정된 해당 액티비티가 실행되었을 때, 처음으로 시작하는 화면설정이다. 아래 `fragment` 태그에 정의된 **id** 와 동일해야 한다.\
\
`android:label` 속성은 사용자가 읽을 수 있는 대상 이름을 포함한다. 예로 NavGraph 를 setupWithNavController()를 사용하여 Toolbar와 연결한 경우, UI에 해당 라벨 값이 표시된다.\
\
`tools:layout` 속성은 해당 프래그먼트에 레이아웃을 연결하여 그래프 화면에서 해당 화면을 미리보여줄 수 있다.



\- **화면간의 이동 구현**

프래그먼트를 하나 더 생성하고, 위와 같은 방식으로 탐색 그래프에 대상을 추가한다.

![오른쪽 listFragment는 tools:layout 속성을 주지 않은 상태이다.](<../../../.gitbook/assets/스크린샷 2022-07-22 오후 2.51.03.png>)

프래그먼트를 추가하고 `Navigation Editor` 에서 특정 프래그먼트를 클릭하면  위 사진과 같이 오른쪽 중앙에 큰 동그라미가 표시된다. 해당 버튼을 클릭 한 채로 이동하려는 대상(listFragment) 로 끌고가면 XML 코드에서 `action` 이라는 태그가 생성되며 화면전환 대상 지정은 끝이다.

\
이동 방법은 아래와 같다.&#x20;

```kotlin
override fun onViewCreated(view: View, savedInstanceState: Bundle?) {
        super.onViewCreated(view, savedInstanceState)

        Handler(Looper.getMainLooper()).postDelayed({
            // 화면이동전환_1
            findNavController().navigate(R.id.action_homeFragment_to_listFragment2)
            // 화면이동전환_2 todo: safe-args 때 많이 사용
            val hl = HomeFragmentDirections.actionHomeFragmentToListFragment2()
            findNavController().navigate(hl)
        }, 500)
    }
```

화면전환 1번방식은 nav\_graph 파일에 정의된 액션태그에 아이디와 동일하다\
화면전환 2번방식은 추후에 작성할 safe-args 관련하여 플러그인 설정 후 사용이 가능하다



//
