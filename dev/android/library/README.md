# DI

![https://geekflare.com/fullstack-development-frameworks-libraries/](https://geekflare.com/cdn-cgi/image/width=1000,height=580,fit=crop,quality=80,format=auto,onerror=redirect,metadata=none/wp-content/uploads/2020/03/web-development.jpg)



### DI (Dependency injection / 의존성 주입) 이란?

우선 안드로이드 DI 를 적용하면 훌륭한 앱 아키텍처를 위한 토대를 마련할 수 있다.\
안드로이드 공식문서에 따르면 DI 를 적용하면 아래와 같은 이점이 누릴 수 있다.

* 코드 재사용 가능
* 리팩토링 편의성
* 테스트 편의성

~~이 글을 작성하면서 나 스스로 위에 이점들을 잘 누리고 있었는지 되돌아 볼 것이다.~~

****

우선 클래스에는 다른 클래스 참조가 필요하다. 예를 들어 `Car` 클래스는 `Engine` 클래스 참조가 필요할 수 있다.\
이처럼 필요한 클래스를 종속 항목(_dependencies) 이라고 하며, `Car` 클래스가 실행되기 위해서는 `Engine` 클래스의 인스턴스가 필요하다._

_클래스가 필요한 객체를 얻는 방법은 3가지가 있다._

* 클래스가 필요한 종속 항목 (Engine Class) 을 구성한다. 위 예에서는 Car 클래스는 자체 Engine 인스턴스를 생성하여 초기화한다.
* 다른 곳에서 객체를 가져온다. `Context` getter & `getSystemService`() 와 같은 일부 Android API는 여러한 방식으로 작동한다.
* 객체를 매게변수로 제공받는다. 앱은 클래스가 구성될 때 이러한 종속 항목을 제공하거나 각 종속 항목이 필요한 함수에 전달할 수 있다. 위 예에서는 Car 생성자는 Engine 을 매개변수로 받는다.

**여기서 말하는 3번째 방법이 바로 DI / 의존성주입이다** 🔥\
__이 방법을 사용하면 클래스 인스턴스가 자체적으로 종속 항목을 얻는 대신 클래스의 종속 항목을 받아서 제공한다.

우선 1번과 같은 방법으로 인스턴스화 하는거는 아래 코드와 같다.

```kotlin
class Car {

    private val engine = Engine()

    fun start() {
        engine.start()
    }
}

class Engine {
    fun start() {
        // write code...
    }
}
```

이 예는 Car 클래스가 자체 Engine 을 객체생성 후 사용하기 때문에 DI 가 아니다. 또한 아래와 같은 문제가 발생할 수 있다.

* Car 와 Engine 은 밀접하게 연결되어있다. Car 인스턴스는 한 가지 유형의 Engine 을 사용하므로 서브클래스 또는 대체 구현을 쉽게 사용할 수 없다. Car 가 자체 Engine 을 구성했다면 `Gas` / `Electric` 유형의 엔진에 동일한 Car 를 재사용하는 대신 두 가지 유형의 Car을 생성해야 한다.
* 위 예제와 같이 의존성이 높은 경우 다양한 테스트 케이스에서 테스트를 더욱 어렵게 만든다.



**DI  접근방식**

&#x20; _1. 생성자 삽입_

```kotlin
class Car(private val engine: Engine) {

    fun start() {
        engine.start()
    }
}

fun main(args: Array) {
    val engine = Engine()
    val car = Car(engine)
    car.start()
}  
```

&#x20; _2.   필드 삽입_

```kotlin
class Car() {
    lateinit var engine: Engine

    fun start() {
        engine.start()
    }
}

fun main() {
    val car = Car()
    car.engine = Engine()
    car.start()
```

위 예제와 같은 DI 기반 접근방법의 이점은 다음과 같다.

* Car 의 재사용가능. Engine 의 다양한 구현을 Car 에 전달할 수 있다. 예를 들어 GasEngine 을 사용하는 경우, Engine 클래스를 상속받아 서브클래스를 정의할 수 있다. GasEngine 을 인스턴스로 전달하면 Car 클래스는 아무런 변경없이 사용할 수 있다.
* [테스트 더블](https://en.wikipedia.org/wiki/Test\_double)을 전달하여 다양한 시나리오 테스트를 할 수 있다.

이전 예에서는 라이브러리를 사용하지 않고 직접 생성 및 관리를 하였다. 이를 _직접 종속 | 수동 종속_ _항목 삽입_ 이라고 한다. 종속 항목이 많아 질 수록 종속 항목을 삽입하는 작업이 지루해 질 수 있다. 뿐만 아니라 다음과 같은 문제점들이 있다.

* 대규모 앱의 경우 모든 종속 항목을 가져와 올바르게 연결하려면 대량의 상용구 코드가 필요하다.\
  예로, 자동차를 만들려면 엔진,변속기 기타 부품 엔진에는 실린더, 점화 플로그 등이 필요로 하다.
* 종속 항목을 전달하기 전에 구성할 수 없을 때는 메모리에서 종속 항목의 전체 기간을 관리하는 맞춤 컨테이너를 작성하고 유지해야 한다.



이러한 문제점들을 해결하는 라이브러리들이 존재하며, 이는 두 가지 카테고리로 분류된다.

* 런타임 시 종속 항목을 연결하는 리플렉션 기반 솔루션 - Koin...
* 컴파일 타임에 종속 항목을 연결하는 코드를 생성하는 정적 솔루션 - Dagger / Hilt...



{% embed url="https://developer.android.com/training/dependency-injection?hl=en" %}
_Reference_
{% endembed %}

{% embed url="https://ko.wikipedia.org/wiki/%EC%9D%98%EC%A1%B4%EC%84%B1_%EC%A3%BC%EC%9E%85" %}
_Reference_
{% endembed %}
