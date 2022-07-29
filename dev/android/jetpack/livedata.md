---
description: Lifecycle-aware component
---

# LiveData

## LiveData 란

LiveData는 관찰 가능한 데이터 홀더 클래스이다. 관찰 가능한 일반 클래스와 달리 LiveData는 수명 주기를 인식한다.\
액티비티, 프래그먼트, 서비스 등 앱 구성요소의 수명 주기를 고려한다. 수명 주기 인식을 통해 LiveData는 활동 수명 주기 상태에 있는 앱 구성요소 관찰자만 업데이트 한다.

`Observer` 클래스로 표현되는 관찰자의 수명 주기가 `STARTED` | `RESUME` 상태이면 LiveData는 관찰자를 활성상태로 간주한다. LiveData는 관찰자에게만 업데이트 정보를 알린다.\
\
`LifecycleOwner` 인터페이스를 구현하는 객체와 페어링된 관찰자를 등록할 수 있다. 이 관계를 사용하면 관찰자에 대응되는 `Lifecycle` 객체의 상태가 `DESTROYED` 로 변경될 때 관찰자를 삭제할 수 있다. LiveData 객체는 액티비티나 프래그먼트에서 특히 유용하게 사용가능하며, 객체를 안전하게 관찰할 수 있고 수명주기가 끝나는 즉시 수신거부가 되어메모리 누수를 걱정 하지 않아도 된다.\
\
LiveData 사용에 대한 이점은 다음과 같다.

> * **UI 와 데이터 상태의 일치 보장**\
>   \- LiveData는 옵저버 패턴을 따르며, 기본 데이터가 변경될 때 Observer 객체에 알린다.\
>   Observer객체에 UI를 업데이트 할 수 있으며 이렇게 하면 앱 데이터가 변경될 때 마다 관찰자가 대신 UI를 업데이트 하므로 추가 작업을 처리할 필요가 없다.
> * **메모리 누수 없음**\
>   관찰자는 Lifecycle 객체에 결합되어 있으며 연결된 수명주기가 끝나면 자동으로 삭제된다
> * **중지된 활동으로 인한 비정상 종료 없음**\
>   위에서 설명했듯이 LiveData 는 STARTED | RESUME 상태일 때 활성상태로 간주하기 때문에, 액티비티가 백스텍에 있을 때나 관찰자가 수명 주기가 비활성화 상태이면 어떠한 LiveData 를 수신하지 않기 때문에 오류가 발생하지 않는다.
> * **수명 주기를 더 이상 수동으로 처리하지 않음**\
>   UI 구성요소는 관련 데이터를 관찰만 할 뿐 관찰을 중지하거나 다시 시작하지 않는다.\
>   LiveData는 관찰하는 동안 관련 수명 주기 상태의 변경을 인식하므로 자동으로 관리가 된다.
> * **최신 데이터 유지**\
>   백그라운드에서 (onPause | onStop ) 포그라운드로 전환되었을 때 LiveData는 다시 활성화 상태가 되면서 관찰자는 최신 데이터를 수신 받게 된다.
> * **리소스 공유**\
>   앱에서 시스템 서비스를 공유할 수 있도록 싱글톤 패턴을 사용하는 LiveData 객체를 확장하여 시스템 서비스에 한 번 연결하면 리소스가 필요한 모든 관찰자가 LiveData 를 볼 수 있다.

### LiveData 사용

LiveData는 일반적으로 ViewModel 클래스 사용된다. _안드로이드 MVVM 디자인 패턴에서 사용된다._

1. `ViewModel` 에 LiveData 객체를 생성한다.

```kotlin
class MyViewModel: ViewModel() {
    // 관찰하는 액티비티나 프래그먼트 등에서 LiveData 를 관찰만 하고 데이터의 값을 변경할 수 없게
    // 값을 주입하는 _resData 는 변경가능 한 Mutable형을 사용하고
    // resData는 값이 불가능한 Immutable형을 사용한다.
    
    private val _resData: MutableLiveData<Int> = MutableLiveData()
    val resData
        get() = _resData
}
```

&#x20; 2\.  Observer 객체를 만든다. `onChange()`메서드는 LiveData 객체가 보유한 데이터가 변경되었을 때, 호출된다.

```kotlin
public interface Observer<T> {
    /**
     * Called when the data is changed.
     * @param t  The new data
     */
    void onChanged(T t);
}
```

```kotlin
class MyActivity: AppCompatActivity() {

    private val val myViewModel by lazy { ViewModelProvider(this)[MyViewModel::class.java] }

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        ...
        
        // observe() 메서드를 통해 LiveData 객체에 Observer 객체를 연결한다.
        // observe() 메서드는 LifecyclewOwner 를 파라미터로 받는다.
        myViewModel.resData.observe(this) { data ->
            println(data)    // initialize liveData
        }
    }
}
```

**관찰자 등록 (observe 메서드) 은 액티비티 기준** `onCreate()`**메서드에서 해주는 것이 좋다.**\
****\
****사용자 활동에 따라 `onResume()`은 여러번 호출될 수 있는 메서드이므로 중복 호출을 하지 않도록 해야한다.\
또한 `STARTED` 상태가 되었을 때, LiveData는 즉시 최신 데이터를 수신한다. 이는 이미 관찰자가 등록되어있을 때만 발생하므로 이전에 설정을 해놓아야 한다.

### LiveData 객체 업데이트

아래 예제는 액티비티 내 버튼 클릭 시 뷰모델에 있는 MutableLiveData 객체에 값을 수정하고,\
해당 데이터를 관찰하는 액티비티에서 UI를 변경한다.

```kotlin
# MyActivity.class
    ...
    override onCreate(saveInstanceState: Bundle?) {
        ...
        
        findViewById<Button>(R.id.btn_ok).setOnClickListener {
            myViewModel.plusNum()
        }
        viewModel.resData.observe(this) {
            findViewById<TextView>(R.id.num_text_view).text(it)
        }
    }
```

```kotlin
# MyViewModel.class
    ...
    fun plusNum() {
        val num = _resData.value ?: 0
        // 라이브데이터 값을 변경하는 메서드는 setValue / postValue() 가 있다.
        _resData.postValue(++num)
        // _resData.setValue(++num)
    }
```

`setValue()` : 해당 메서드는 메인스레드에서 호출된다.\
`postValue()` : 해당 메서드는 백그라운드 스레드에서 작업을 처리하고 메인스레드에 값을 post 하는 방식이다.



// todo ...
