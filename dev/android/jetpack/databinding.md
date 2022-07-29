# DataBinding

##

## DataBinding 이란

DataBinding 라이브러리는 프로그래매틱 방식이 아니라 선언적 형식으로 레이아웃의 UI 구성요소를 앱의 데이터 소스와 결합할 수 있는 라이브러리이다.

```kotlin
// default (ui class)
findViewById<TextView>(R.id.sample_text).apply {
    text = viewModel.userName
}

// viewBinding (ui class)
binding.sampleText.text = viewModel.userName

// dataBinding (xml)
<TextView
    android:text="@{viewModel.userName}"
    ... />
```

위 예제는 데이터바인딩의 기본적인 사용방법이다. 코틀린or자바 코드에서 텍스트를 할당하는게 아닌 XML 파일에서\
텍스트를 할당한다. 할당 표현식으로 `@{}` 를 사용한다.

레이아웃 파일(XML) 에서 구성요소를 결합하면 액티비티나 프래그먼트 내에서의 UI 관련 호출을 줄일 수 있어\
**파일이 더욱 단순화되고 유지관리 또한 쉬워진다. 앱 성능이 향상되며 메모리 누수 및 NPE 를 방지 할 수 있다.**

### 시작하기

\# build.gradle(:app)

```kotlin
android {
    ...
    buildFeatures {
        dataBinding true
    }
}
```

\# layout file

```kotlin
<layout xmlns:android="http://schemas.android.com/apk/res/android"
            xmlns:app="http://schemas.android.com/apk/res-auto">
    <data>
        <variable
            name="viewmodel"
            type="com.myapp.data.ViewModel" />
    </data>
    <ConstraintLayout... /> <!-- UI layout's root element -->
</layout>    
```

