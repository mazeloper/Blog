---
description: >-
  Koin은 입사 후 이전 퇴사자가 사용한 것을 참고하여 새로운 프로젝트에 적용하였지만 사실 제대로 알고 사용하지못했다. 해당 글을 작성하면서
  나 또한 열심히 알아볼 예정이다 😱
---

# Koin

![https://insert-koin.io/](https://insert-koin.io/img/koin\_new\_logo.png)

## Koin 이란?

Koin은 **Kotlin DSL**로 만들어진 런타임 시 종속항목을 연결하는 대표적인 DI 라이브러리중 하나이다.

_장점_

* 러닝커브가 다른 DI 라이브러리 보다 현저히 낮다.
* Annotation 과정이 없어서 컴파일이 빠르다.

_단점_

* **리플렉션**을 이용해 의존성 주입을 하다보니 앱 성능이 저하된다.
* 런타임 시 의존성을 주입하다보니 에러가 발생할 수 있다.

#### DSL

위에서 언급한 `DSL` 이란, **Domain Specific Language** 의 약자로 특정 도메인에 특화된 언어이다.\
DSL은 상대적으로 자연어에 가까워 가독성을 높일 수 있다.

* 람다 표현식 - 선언되지 않았지만 즉시 표현식으로 전달되는 함수
* 고차함수 - 함수를 매개변수로 취하거나 함수를 반환하는 함수
* 확장함수 - 기종에 정의된 클래스에 새로운 함수가 추가되는 기능

#### ~~다 적지는 않았지만, 코드를 간결하게 작성할 수 있는 기능을 많이 제공한다.~~

#### 리플렉션 (Reflection)

리플렉션이란 런타임에 클래스, 필드 및 메서드를 검사, 로드 및 상호 작용하는 기능의 이름이다.

_장점_

* 생성자, 메서드, 필드를 재귀적으로 조작

단점

* 컴파일 시점에 가능한 타입 확인이 불가능
* 재귀적인 접근을 필요로 하는 코드는 가독성이 떨어짐
* 처리 서능이 늦음

~~Koin에 관한 포스팅이므로 리플렉션의 대한 더 자세한 설명은~~ [~~https://kotlinlangorg/docs/reflection.html~~](https://kotlinlang.org/docs/reflection.html) ~~를~~ \
~~참고하면 된다.~~

### Koin 설정

```kotlin
// Add Maven Central to your repositories if needed
repositories {
    mavenCentral()
}

// build.gradle (:app)
dependencies {
    ..
    def koin_version = "3.2.0"
    implementation "io.insert-koin:koin-android:$koin_version"
}
```

### Koin 시작하기

`KoinApplication` 인스턴스는 Koin 컨테이너 인스턴스 구성이다. 로깅, 속성 로드, 모듈을 구성할 수 있다.

`KoinApplication { }`  - KoinApplication 컨테이너 구성 생성

`startKoin { }` - GlobalContext API를 사용할 수 있도록 KoinApplication 컨테이너 구성을 생성 및 등록

```kotlin
class MyApplication : Application() {

    override fun onCreate() {
        super.onCreate()

        initKoin()
    }

    private fun initKoin() {
        startKoin {
            //inject Android context
            // Context 인스턴스를 필요로 하는 표현식을 간단히 작성할 수 있다.
            androidContext(this@MyApplication)
            // default Logger - Level.Info
            androidLogger()
            /* androidFileProperties() -  use properties from assets/koin.properties */
            
            modules(appModule)
        }
    }
}
```

AndroidManifest.xml - name속성에 작성한 어플리케이션 적용

```kotlin
 <application
        android:name=".MyApplication" ... 
        
        
```

Koin 모듈은 어플리케이션 주입/결합할 정의를 수집한다.

`module { // module content }}` - 코인 모듈 생성



모듈에서 다음 기능을 사용할 수 있다.

* `factory { }` : 팩토리 빈 제공 (매번 새로 생성)
* `single { }` : 싱글톤 빈 제공 (하나만 생성하고 계속 재활용)
* `get()` : 구성 요소 종속성 해결(이름, 범위 또는 매개변수도 사용할 수 있음)
* `bind()` : 주어진 빈 정의에 대해 바인딩할 유형 추가
* `binds()` : 주어진 빈 정의에 대한 유형 배열 추가
* `scope { }` : 정의를 위한 논리적 그룹 정의
* `scoped { }` : 범위에만 존재하는 빈 정의 제공
* `name()` : 정의의 이름을 지정하는데 사용

#### 의존성 주입

```kotlin
class Service()
class Controller(val view: View)

val myModules = module {
    single { Service() }
    // get() 으로 View 인스턴스 해결한다.
    single { Controller(get()) }
}
```

```kotlin
interface Service {
    fun work()
}
class ServiceImpl() : Service {
    fun work() { ... }
}
class Controller(val view: View)

// DI
val modules = modules {
    // ServiceImpl 만 구현
    single { ServiceImpl() }
    // Service 만 구현 _ 1
    single { ServiceImpl() as  Service }
    // Service 만 구현 _ 2
    single<Service> { ServiceImpl() }
    // 하나의 정의에서 여러 유형을 일치
    single { ServiceImpl() } bind Service::class
    // 이름을 지어주어 구별할 수 있음(1-1)
    single(named("test")) { Test(get()) }
    // 파라미터 해결 1
    single { Controller(get()) }
    // 파라미터 해결(2-1)
    single { (view: View) -> Controller(get()) }
}

// 이름을 지어주어 구별할 수 있음(1-2)
// inject : Lazy injected Presenter instance
val service: Service by inject(qualifier = named("test"))

// 파라미터 해결(2-2)
val controller: Controller by inject { parameterOf(view) }

```

더 자세한 기능들에 대해서는 [Koin 공식문서](https://insert-koin.io/docs/reference/introduction) 를 통해 확인할 수 있다.\






{% embed url="https://en.wikipedia.org/wiki/Domain-specific_language" %}
Reference - DSL
{% endembed %}

{% embed url="https://brunch.co.kr/@oemilk/172" %}
Reference - Reflection
{% endembed %}

{% embed url="https://insert-koin.io/docs/reference/introduction" %}
Reference - Koin
{% endembed %}
