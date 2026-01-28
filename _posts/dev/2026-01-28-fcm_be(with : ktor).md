---
layout: post
title: "Android + Ktor로 배우는 FCM 푸시(2)"
category: dev
---

# Ktor 3(+) + Koin으로 FCM 알림 서버 깔끔하게 구축하기

Ktor를 사용해 FCM 알림 서버를 구축했다. 
Ktor 3버전으로 마이그레이션하면서 겪은 라이브러리 충돌 이슈와 해결 과정을 공유한다. 
불필요한 삽질을 줄이고 싶은 분들에게 도움이 되길 바란다.

---

## 1. 개발 환경
- **Framework**: Ktor 3.3.3
- **Language**: Kotlin 2.2.21
- **DI**: Koin 4.1.0Beta8
- **FCM**: Firebase Admin SDK 9.4.1

---

## 2. 핵심 설정: 의존성 관리 (libs.versions.toml)

Ktor 3 버전이상을 사용한다면 Koin 설정이 핵심이다. 기존 `koin-ktor`를 사용하면 런타임 에러가 발생하므로 반드시 `koin-ktor3`를 사용해야 한다.

```toml
[versions]
ktor = "3.3.3"
koin = "4.1.0-Beta8"

[libraries]
# Ktor 3 대응을 위한 전용 라이브러리 사용
koin-ktor = { module = "io.insert-koin:koin-ktor3", version.ref = "koin" }
```

---

## 3. 주요 트러블슈팅 및 해결 방안

### 1) 중복 플러그인 설치 (DuplicatePluginException)
`CallLogging` 플러그인을 분리해서 설치하다가 발생한 에러다. Ktor는 동일한 플러그인의 중복 설치를 허용하지 않는다.

- **해결**: `install(CallLogging)` 블록 하나에 `level`, `format`, `callIdMdc` 설정을 모두 통합했다.

### 2) 클래스 참조 에러 (NoClassDefFoundError)
서버 실행 시 `RoutingKt` 클래스를 찾지 못하는 문제가 발생했다. 이는 Ktor 3와 Koin 버전 간의 호환성 문제였다.

- **해결**: `koin-ktor3`로 라이브러리를 교체하여 즉시 해결했다.

---

## 4. 구조 설계 (Clean Architecture)

유지보수와 확장성을 고려해 **Domain, Data, Presentation** 계층을 분리했다.

- **Domain**: 알림 대상(Token, Topic)과 데이터 모델 정의.
- **Data**: Firebase SDK를 활용한 실질적인 전송 로직 구현.
- **Presentation**: API 엔드포인트 구성 및 요청 처리.

계층 분리를 통해 코드가 명확해지고 테스트가 용이해졌다.

---

## 5. 전송 방식: Token vs Topic

실제 테스트를 통해 확인한 두 방식의 차이점이다.

- **Token**: 특정 기기 식별자를 이용한 1:1 메시지 전송. 개인화된 알림에 사용한다.
- **Topic**: 특정 주제를 구독한 그룹에 메시지 전송. 전체 공지나 이벤트 알림에 효율적이다.

**참고**: Topic 전송 시 클라이언트가 해당 주제를 미리 구독하고 있어야 하며, 서버와 주제 이름을 정확히 일치시켜야 한다.

---

## 6. 결론
![테스트 메시지 성공](/images/fcm_be_screenshot.png)

FCM 서버 구축 시 `messageId`가 정상적으로 반환된다면 백엔드 로직은 성공이다. 
그럼에도 기기에 알림이 오지 않는다면 앱의 권한 설정이나 토픽 구독 여부를 확인해보면 된다. 