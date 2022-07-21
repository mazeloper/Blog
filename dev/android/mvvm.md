# MVVM 패턴

![출처 : https://medium.com/swlh/understanding-mvvm-architecture-in-android-aa66f7e1a70b](../../.gitbook/assets/mvvm\_.png)

### MVVM 이란?

기존 MVC / MVP 디자인 패턴의 단점들을 극복하는 오늘날 가장 많이 사용되고 있는 디자인 패턴 중 하나이다.

**Model-View-ViewModel (MVVM)** 은 어플리케이션의 UI와 비즈니스 로직을 분리하여 유지보수에 용이하며, 테스트를 쉽게 할 수 있는 장점이다.

개별 계층은 아래와 같다.

> **View**
>
> UI에 관련된 작업만을 구현하며, ViewModel 에 사용자의 작업을 알린다. ViewModel 이 비즈니스 로직에 대한 결과값을 반환하지 않으며, ViewModel을 관찰하며 상태에 따른 UI 처리를 한다.

> **ViewModel**
>
> 수명 주기를 고려하여 UI 관련 데이터를 저장하고 관리한다.  Model 과 View 사이의 연결 고리 역할을 한다.

> **Model**
>
> ViewModel 이 요청한 데이터를 저장하거나 Room, SQLite 등 로컬DB 을 사용하거나 네트워크 통신을 통한 데이터를 가져오는 역할을 한다.



### ViewModel

안드로이드 프레임워크는 Activity / Fragment 와 같은 UI 컨트롤러의 수명 주기를 관리한다.

시스템에서 UI 컨트롤러를 제거하거나 다시 만드는 경우, 컨트롤러에 저장된 모든 UI 관련 데이터가 삭제된다.\
디바이스 화면 회전하는 경우 위와 같이 컨트롤러를 제거한 후 새로 생성을 한다. 단순한 데이터를 복원하기 위해서는 `onSaveInstanceState()` 메서드를 사용하여 `onCreate()` 호출 시 데이터를 복원할 수 있다. \
하지만 이 방법은 소량의 데이터에만 적합하다.\




