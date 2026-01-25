---
layout: post
title: "[Android] 일회성 이벤트 처리를 위한 SharedFlow 전략"
category: dev
tags: [Android, Kotlin, Coroutines, Flow]
---

### 1. 시작하며: 상태(State)와 이벤트(Event)는 다르다

안드로이드 개발을 하다 보면 데이터를 UI로 전달하는 수많은 상황을 마주합니다. 보통 `StateFlow`를 사용하여 '현재 상태'를 유지하지만, 때로는 **"특정 시점에 단 한 번 발생하는 이벤트"**를 처리해야 할 때가 있습니다.

예를 들어, 토스트 메시지 출력, 화면 이동, 혹은 특정 데이터 수신 알림 등이 그렇습니다. 이러한 **일회성 이벤트**를 가장 효율적으로 다루기 위해 제가 선택한 `SharedFlow` 전략을 공유합니다.

---
### 2. MutableSharedFlow의 전략적 설계

프로젝트에서 사용한 이벤트 전달 파이프라인의 핵심 코드입니다.
```kotlin
 private val _eventFlow = MutableSharedFlow<String>( replay = 1, // 최신 이벤트 1개는 저장했다가 새로운 구독자에게 바로 전달 onBufferOverflow = BufferOverflow.DROP_OLDEST // 버퍼가 차면 오래된 것부터 버림 (성능 최적화) ) val eventFlow = _eventFlow.asSharedFlow()
 ```

이 짧은 선언에는 세 가지 중요한 설계 의도가 담겨 있습니다.

---

### 3. 왜 SharedFlow인가? (핵심 파헤치기)

#### (1) 왜 StateFlow가 아닌가?
안드로이드에서 가장 많이 쓰이는 두 Flow의 차이점은 명확합니다.

*   **StateFlow**: 항상 현재 상태(State)를 가지고 있어야 합니다. 초기값이 필수이며, UI는 언제나 최신 상태를 반영합니다.
*   **SharedFlow**: **특정 이벤트(Event)**를 전달할 때 사용합니다. 초기값이 필요 없으며, 발생한 사건을 구독자들에게 전파하는 데 집중합니다.

> [!NOTE]
"현재 로딩 중인가?"는 상태(`StateFlow`)이지만, "데이터 수신이 완료됨!"은 이벤트(`SharedFlow`)입니다. 이벤트가 없을 때 억지로 "상태 없음"이라는 초기값을 유지할 필요가 없기 때문에 `SharedFlow`가 이벤트 처리에 더 적합합니다.

#### (2) `replay = 1`의 마법: 최신 데이터 보존
일반적인 이벤트는 발행되는 순간 구독자가 없으면 소멸합니다. 하지만 `replay = 1`은 보험과 같습니다.

*   **의미**: "가장 최근에 발생한 이벤트 1개를 기억하고 있다가, 새로운 구독자가 나타나면 즉시 전달해 줘"라는 뜻입니다.
*   **실제 활용**: 사용자가 잠시 화면을 전환하거나 configuration change(화면 회전 등)가 일어나는 찰나에 이벤트가 발생하더라도, 다시 구독을 시작할 때 해당 이벤트를 놓치지 않고 처리할 수 있게 해줍니다.

#### (3) `onBufferOverflow = BufferOverflow.DROP_OLDEST`
시스템 리소스는 유한합니다. 이벤트가 처리 속도보다 빠르게 대량으로 발생할 때를 대비해야 합니다.

*   **의미**: 버퍼가 꽉 차면 **가장 오래된 낡은 메시지를 버리고** 최신 데이터를 담습니다.
*   **전략적 선택**: 사용자에게 가장 중요한 것은 '지금 당장 벌어진 최신 일'입니다. 성능을 유지하면서도 사용자에게 가장 유효한 정보를 보장하기 위한 설정입니다.

---

### 4. Compose에서의 활용 (Sample)

Compose 환경에서는 다음과 같이 이 이벤트를 안전하게 수집할 수 있습니다.
```kotlin
@Composable 
fun EventScreen(viewModel: ViewModel) { 
    val lifecycleOwner = LocalLifecycleOwner.current
    LaunchedEffect(viewModel.eventFlow) {
    viewModel.eventFlow.collect { message ->
        // 토스트를 띄우거나 화면을 이동하는 등의 이벤트 처리
        println("Received event: $message")
    }
}}
```
---

### 5. 마치며

단순히 `Flow`를 사용하는 것을 넘어, `replay`와 `BufferOverflow` 정책을 고민하는 과정에서 **안드로이드의 비동기 스트림**이 얼마나 정교한지 배울 수 있었습니다.

이벤트의 성격에 따라 이 파라미터들을 조절하는 것만으로도, 훨씬 견고하고 반응성 좋은 앱을 만들 수 있습니다. 오늘 공유한 `SharedFlow` 전략이 여러분의 프로젝트에도 도움이 되길 바랍니다.

---
**참고 자료**
- [Kotlin 공식 문서 - SharedFlow](https://kotlinlang.org/api/kotlinx-coroutines/kotlinx-coroutines-core/kotlinx.coroutines.flow/-shared-flow/)
- [Android Developers - Flow Guide](https://developer.android.com/kotlin/flow)