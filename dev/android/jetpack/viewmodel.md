---
description: Lifecycle-aware component
---

# ViewModel

## ViewModel 이란

#### 안드로이드 프레임워크는 Activity / Fragment 와 같은 UI 컨트롤러의 수명 주기를 관리한다.

시스템에서 UI 컨트롤러를 제거하거나 다시 만드는 경우, 컨트롤러에 저장된 모든 UI 관련 데이터가 삭제된다.\
디바이스 화면 회전하는 경우 위와 같이 컨트롤러를 제거한 후 새로 생성을 한다. 단순한 데이터를 복원하기 위해서는 `onSaveInstanceState()` 메서드를 사용하여 `onCreate()` 호출 시 데이터를 복원할 수 있다. \
하지만 이 방법은 소량의 데이터에만 적합하다.

`ViewModel` 객체는 이러한 상황에서 자동으로 보관되어 액티비티 / 프래그먼트 인스턴스에서 즉시 사용할 수 있다.

### 구현

```kotlin
class MainViewModel : ViewModel() {

    private val _todo: MutableLiveData<String> = MutableLiveData("init")
    val todo: LiveData<String>
        get() = _todo
    
}
```

```kotlin
class MainActivity : AppCompatActivity() {

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_main)
        
        // 초기화 방법 1
        val viewModel: MainViewModel by viewModels()
        // 초기화 방법 2
        val viewModel2 by lazy { ViewModelProvider(this)[MainViewModel::class.java] }

        viewModel.todo.observe(this) { value ->
            Toast.makeText(this, "value : $value", Toast.LENGTH_SHORT).show()
        }
    }
}
```

위에 코드와 같이 뷰모델 클래스는 Jetpack에서 제공하는 `ViewModel` 을 상속하여 만들 수 있고,\
액티비티나 프래그먼트에서 객체생성하여 사용할 수있다.

### 생명주기

![https://developer.android.com/topic/libraries/architecture/viewmodel](../../../.gitbook/assets/viewmodel-lifecycle.png)

ViewModel 은 LifecycleOwners 의 특정 인스턴스화보다 오래 지속되도록 설계되어있다.\
뷰모델 객체에는 View, Activity, Fragment 객체나 Context를 참조하면 안된다. \
_(뷰모델의 라이플사이클은 UI 라이플사이클보다 오래 지속되기 때문에 onCleared() 함수가 호출되기 전,_\
_UI 와 관련 객체나 Context 를 참조하게 되면 GC에서 수거하지 못해, 메모리 누수가 발생한다)_

\
시스템 서비스를 찾는데 ApplicationContext 가 필요한 경우에는 [AndroidViewModel](https://developer.android.com/reference/androidx/lifecycle/AndroidViewModel) 클래스를 확장하여 사용할 수 있다.\
\
ViewModel 객체의 범위는 ViewModel 을 가져올 때 ViewModelProvider 에 전달되는 Lifecycle 로 지정된다.\
범위로 지정된 Lifecycle 이 완전히 끝날때 까지 메모리에 남아있다.\
\
위 [코드](viewmodel.md#undefined)에 액티비티에서 뷰모델을 초기화할 때 ktx 라이브러리를 사용하는 경우 객체의 범위를 쉽게 지정할 수 있다.

### 프래그먼트간의 데이터 공유

하나의 액티비티에서 둘 이상의 프래그먼트를 사용할 때 각각의 뷰모델을 작성하게 되면 둘 이상의 프래그먼트에서의 결합이 쉽게 처리될 수 없는 부분이기에 공유되는 뷰모델을 사용해야 한다.\


```kotlin

class ListFragment : Fragment() {

    // 초기화 방법 1 fragment-ktx library
    private val viewModel1: SharedViewModel by activityViewModels()

    // 초기화 방법 2
    private val viewModel2: SharedViewModel by lazy {
        ViewModelProvider(requireActivity())[SharedViewModel::class.java]
    }

    override fun onCreateView(
        inflater: LayoutInflater,
        container: ViewGroup?,
        savedInstanceState: Bundle?
    ): View? {
        return super.onCreateView(inflater, container, savedInstanceState)

    }
}

class DetailFragment : Fragment() {

    //  Same as above code...
}
```

위 코드와 같이 사용하면 액티비티에 연결된 뷰모델 객체를 공용으로 프래그먼트에서 사용할 수 있다.\
방법1 에서처럼 activity-ktx 와 같이 fragment-ktx 라이브러리를 사용하여 간단하게 인스턴스화 할 수 있고,\
방법2 는 기존 액티비티에서 뷰모델을 연결한거 같이 사용하면 된다. 여기서는 ViewModelProvider에 Lifecycle을 `this` 로 설정하게되면 자기 자신(프래그먼트)에 라이플사이클을 따라가기 때문에 연결된 액티비티 라이플사이클을연결시켜줘야 한다.

{% embed url="https://developer.android.com/topic/libraries/architecture/saving-states?hl=ko" %}
reference
{% endembed %}
