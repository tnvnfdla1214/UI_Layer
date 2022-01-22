# UI 레이어

이 포스터는 [앱 아키텍쳐 가이드](https://developer.android.com/jetpack/guide?hl=ko)를 학습한 후 [ToDo](https://github.com/tnvnfdla1214/ToDo) 프로젝트 작성하여 설명하고 있습니다.

## UI 레이어란
사용자 상호작용(예: 버튼 누르기) 또는 외부 입력(예: 네트워크 응답)으로 인해 데이터가 변할 때마다 변경사항을 반영하도록 UI가 업데이트되어야 합니다. 사실상 UI는 데이터 레이어에서 가져온 애플리케이션 상태를 시각적으로 나타냅니다. 

아래의 사진과 같이 UI 레이어는 UI element 와 State holder 가 합쳐진 형태이고 Data 레이어(or Domain 레이어)에 의존합니다.


<div align="center">
<img src = "https://user-images.githubusercontent.com/48902047/150626326-5641c94d-19fd-4195-b568-1a7958f57f0b.png" width="50%" height="50%">
</div>

## UI 레이어 구현 방법
UI라는 용어는 UI라는 용어는 사용하는 API(뷰 또는 Jetpack Compose)와 관계없이 데이터를 표시하는 활동 및 프래그먼트와 같은 UI 요소를 가리킵니다.

구글 아키텍쳐 가이드에서는 UI 레이어를 구현하는 방법을 4가지로 제시합니다.

+ UI 상태를 정의하는 방법
+ UI 상태를 생성하고 관리하기 위한 단방향 데이터 흐름(UDF)
+ UDF 원칙에 따라 관찰 가능한 데이터 유형으로 UI 상태를 노출하는 방법
+ 관찰 가능한 UI 상태를 사용하는 UI를 구현하는 방법

## UI 상태 정의

<div align="center">
<img src = "https://user-images.githubusercontent.com/48902047/150626416-d7329707-7919-4167-a602-df0f46f30a85.png" width="50%" height="50%">
</div>

위의 그림과 같이 UI는 UI Element + UI State 입니다.

그 중 UI State 는 앱에서 사용자에게 표시하는 이 정보 입니다.

이 항목은 [StatePatten 프로젝트](https://github.com/tnvnfdla1214/StatePattenSample)도 확인한 후 오시는걸 추천합니다.

State 패턴은 일반적으로 총 4가지의 형태가 있습니다.

각각의 장단점과 사용하기 좋은 프로젝트가 있지만 **구글 아키텍쳐 가이드** 에서는 State 3 를 제시해 줍니다.

ToDo에서 사용했던 State Patten을 짧게 설명을 하면

<div align="center">
<img src = "https://user-images.githubusercontent.com/48902047/150626601-414440a1-7dbc-47f6-9b71-076725cd44a4.png" width="50%" height="50%">
</div>

위의 사진은 ToDo 프로젝트의 sealed class 들 입니다.

해당 내용은 아래와 같이 해당 Ui의 상태에 따라 설정해 놓았습니다.

MVI와 가장 유사한 형태이며 의도를 파악하기 좋지만 표현하고자 하는 모든 상태를 나열해야 하기 때문에 화면이 복잡해지면 상태가 비약적으로 늘어나고 부분적인 업데이트가 불가한게 특징입니다.

 ```Kotlin
sealed class ToDoDetailState {
    object UnInitialized: ToDoDetailState()
    object Loading: ToDoDetailState()
    data class Success(
        val toDoItem: ToDoEntity
    ): ToDoDetailState()
    object Delete: ToDoDetailState()
    object Modify: ToDoDetailState()
    object Error: ToDoDetailState()
    object Write: ToDoDetailState()
}
```

가이드의 이름 규칙은 **기능 + UiState** 으로 작성합니다.

## 단방향 데이터 흐름으로 상태 관리

UI는 Data를 State에 맞춰 이벤트에 적용할 [**비지니스로직**](https://github.com/tnvnfdla1214/-Business_logic) 을 정의하고 제작해 UI 동작 로직에 적용해야 합니다. 그러기 위해서는 로직을 짜야하는데 Activity나 Fragment에 작성할 경우 UI에 부담을 주게됩니다. 또한 양향이 이루어 질 경우 테스트 등 많은 영향을 끼치게 됩니다. 그러므로 Viewmodel을 제작하여 단방향으로 흐르게 하고 Activity와 Fragment는 UI 로직만 수행하여 UI의 부담을 줄여줍니다.
(추가적으로 비지니스 로직은 Damain 레이어를 추가하면 Domain 레이어가 맡게 되고 ViewModel은 Domain 레이어와 연결하는 연결고리가 되어 더욱 커플링을 끊어내게 됩니다.)

아래 사진은 ViewModel을 추가한 Date레이어와 UI 레이어의 연결 그림입니다.

<div align="center">
<img src = "https://user-images.githubusercontent.com/48902047/150627875-dc2f6e23-e4b9-402a-aae0-50456bcf45bc.png" width="50%" height="50%">
</div>

ViewModel의 역할은 다음과 같습니다.

+ ViewModel이 UI에 사용될 상태를 보유하고 노출합니다. UI 상태는 ViewModel에 의해 변환된 애플리케이션 데이터입니다.
+ UI가 ViewModel에 사용자 이벤트를 알립니다.
+ ViewModel이 사용자 작업을 처리하고 상태를 업데이트합니다.
+ 업데이트된 상태가 렌더링할 UI에 다시 제공됩니다.
상태 변경을 야기하는 모든 이벤트에 위의 작업이 반복됩니다.

최종적으로 아래와 같은 이점을 얻게 됩니다.

+ 데이터 일관성: UI용 정보 소스가 하나입니다.
+ 테스트 가능성: 상태 소스가 분리되므로 UI와 별개로 테스트할 수 있습니다.
+ 유지 관리성: 상태 변경은 잘 정의된 패턴을 따릅니다. 즉, 변경은 사용자 이벤트 및 데이터를 가져온 소스 모두의 영향을 받습니다.

## @중요@  UI 상태 노출
우리는 ViewModel의 중요성을 알게 되었습니다. 물론 위와 같은 사항은 **Architecture Pattern** 차트에서 설명한 내용과 일치하여 이해가 쉽습니다.

중요한 부분은 이차트인데 정보가 부족해 정말 애를 많이 먹었습니다.

앞선 내용들을 토대로 ViewModel은 Activity에게 정보를 주기 위해서는 [LiveData](https://github.com/tnvnfdla1214/LiveData) (+ MutableLiveData)를 이용합니다.
