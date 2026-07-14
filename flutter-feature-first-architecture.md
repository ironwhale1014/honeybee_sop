# Flutter Feature-first 아키텍처 가이드

## 1. 목적

이 문서는 Flutter 프로젝트를 기능 중심의 Feature-first 구조로 설계하거나, 기존 Layer-first 구조를 점진적으로 전환할 때 사용하는 공통 기준이다.

구조의 기본 뼈대는 Flutter 프로젝트에서 검증한 `app / core / features / shared` 구분과 feature 내부 계층 원칙을 사용한다. 여기에 feature 간 의존성 방향, 공개 범위, 외부 진입점 관리 원칙을 결합한다.

목표는 다음과 같다.

- 한 기능을 변경할 때 관련 코드를 한 feature 안에서 찾을 수 있게 한다.
- 기능 간 내부 구현 침범과 순환 의존성을 방지한다.
- 공통 코드가 무분별하게 `shared`에 쌓이는 것을 막는다.
- 모든 기능에 불필요한 계층과 클래스를 강제하지 않는다.
- 기존 앱을 중단하지 않고 작은 단위로 전환할 수 있게 한다.

이 문서는 엄격한 Clean Architecture 구현 자체를 목표로 하지 않는다. 의존성 방향과 책임 분리는 지키되, 프로젝트 규모와 기능 복잡도에 맞는 최소한의 계층만 사용한다.

---

## 2. 핵심 원칙

### 2.1 기능을 최상위 분류 기준으로 사용한다

다음과 같은 Layer-first 구조는 기능 하나를 수정할 때 여러 최상위 폴더를 오가게 만든다.

```text
lib/
├── models/
├── repositories/
├── screens/
├── providers/
└── widgets/
```

Feature-first 구조에서는 사용자 기능과 도메인을 기준으로 코드를 모은다.

```text
lib/
├── app/
├── core/
├── features/
│   ├── auth/
│   ├── home/
│   ├── workout_journal/
│   └── settings/
└── shared/
```

한 기능의 화면, 상태, 데이터 접근, 모델은 가능한 한 해당 feature 폴더 안에 둔다.

### 2.2 공개된 경계만 사용한다

다른 feature가 특정 feature를 사용해야 한다면 공개 API 또는 명시적으로 허용된 진입점을 통해 사용한다.

다음과 같은 내부 경로 직접 참조는 기본적으로 금지한다.

```dart
import 'package:app/features/auth/data/firebase_auth_repository.dart';
import 'package:app/features/auth/presentation/widgets/login_form.dart';
```

필요하면 feature 루트에 공개 API 파일을 둔다.

```text
features/auth/
├── auth.dart
├── domain/
├── data/
├── application/
└── presentation/
```

```dart
// features/auth/auth.dart
export 'domain/app_user.dart';
export 'presentation/login_screen.dart';
```

공개 API 파일은 필수가 아니다. 외부에서 실제로 사용하는 안정된 대상이 있을 때만 만든다.

### 2.3 필요한 계층만 사용한다

모든 feature에 DTO, Mapper, UseCase, DataSource를 기계적으로 만들지 않는다.

단순한 화면 feature라면 다음과 같이 시작할 수 있다.

```text
features/settings/
└── presentation/
    └── settings_screen.dart
```

서버 연동과 상태 변경이 있는 feature라면 필요한 계층을 추가한다.

```text
features/auth/
├── domain/
├── data/
├── application/
└── presentation/
```

복잡성이 생기기 전에 추상화를 만들지 않고, 책임 분리가 필요해지는 시점에 계층을 추가한다.

### 2.4 공통화는 실제 재사용 이후에 한다

같은 코드가 두 개 이상의 feature에서 실제로 사용되고, 특정 feature의 용어와 규칙을 알지 않아도 동작할 때만 `shared` 후보가 된다.

다음 조건을 모두 확인한다.

- 둘 이상의 feature에서 실제로 사용되는가
- 특정 도메인 이름이나 정책에 의존하지 않는가
- API가 충분히 안정되어 있는가
- 공통화로 인해 사용처가 더 복잡해지지 않는가

미래에 재사용할 것이라는 예상만으로 `shared`에 넣지 않는다.

---

## 3. 표준 폴더 구조

```text
lib/
├── main.dart
├── bootstrap.dart
│
├── app/
│   ├── app.dart
│   ├── router/
│   │   ├── app_router.dart
│   │   ├── route_names.dart
│   │   └── route_paths.dart
│   └── theme/
│       ├── app_colors.dart
│       ├── app_spacing.dart
│       └── app_theme.dart
│
├── core/
│   ├── config/
│   ├── error/
│   ├── logging/
│   ├── network/
│   ├── storage/
│   └── platform/
│
├── features/
│   ├── auth/
│   │   ├── auth.dart
│   │   ├── domain/
│   │   ├── data/
│   │   ├── application/
│   │   └── presentation/
│   │       └── widgets/
│   ├── home/
│   └── settings/
│
└── shared/
    ├── extensions/
    ├── models/
    ├── utils/
    └── widgets/
```

프로젝트에 필요하지 않은 폴더는 만들지 않는다. 빈 폴더를 구조 완성 목적으로 미리 추가하지 않는다.

---

## 4. 최상위 영역별 책임

### 4.1 `app`

애플리케이션 전체를 조립하는 영역이다.

포함할 수 있는 항목:

- `MaterialApp` 또는 `MaterialApp.router`
- 전역 Router 구성
- 앱 테마와 locale 조립
- 전역 ProviderScope override
- feature 라우트 등록
- 앱 수준 navigation shell

`app`은 feature를 조립할 수 있지만 특정 feature의 API 호출, 데이터 변환, 세부 비즈니스 규칙을 직접 구현하지 않는다.

### 4.2 `core`

특정 사용자 기능에 속하지 않는 기술 기반 코드다.

예:

- 환경설정과 `dart-define` 처리
- HTTP client와 interceptor
- 로깅과 Crash reporting 기반
- 공통 예외 타입
- 로컬 저장소 기반
- 플랫폼 adapter
- Firebase, Supabase 같은 SDK 초기화

`core`는 `app`이나 특정 feature를 import하지 않는다.

### 4.3 `features`

사용자에게 제공하는 기능과 도메인 단위 코드를 둔다.

예:

- `auth`
- `workout_journal`
- `study_session`
- `settings`
- `remote_update`

feature 이름은 기술이 아니라 사용자의 기능이나 업무 도메인을 나타내야 한다.

좋지 않은 예:

```text
features/api/
features/provider/
features/screens/
```

좋은 예:

```text
features/auth/
features/workout_journal/
features/settings/
```

### 4.4 `shared`

특정 feature를 알지 않는 재사용 코드를 둔다.

예:

- 공통 버튼과 입력 필드
- 여러 feature에서 사용하는 loading, empty, error UI
- 범용 formatter와 extension
- 도메인과 무관한 공통 값 객체

`shared`는 `features`와 `app`을 import하지 않는다. 공통 위젯은 특정 feature의 Provider나 Repository를 직접 읽지 않고 필요한 값과 callback을 인자로 받는다.

---

## 5. Feature 내부 계층

### 5.1 `domain`

기능의 핵심 개념과 계약을 둔다.

포함할 수 있는 항목:

- Entity와 Value Object
- Repository interface
- 도메인 규칙
- 도메인 전용 예외

`domain`은 Flutter UI, SDK 응답 Map, Dio, Supabase, Firebase 같은 구현 세부사항을 알지 않아야 한다.

```text
features/auth/domain/
├── app_user.dart
└── auth_repository.dart
```

### 5.2 `data`

외부 데이터와 domain 사이의 변환과 구현을 담당한다.

포함할 수 있는 항목:

- API DTO와 database row model
- Mapper
- Remote/Local DataSource
- Repository 구현체
- Retrofit, Dio, Supabase SDK 호출

presentation에 SDK 응답이나 `Map<String, dynamic>`을 직접 전달하지 않는다.

```text
features/auth/data/
├── auth_user_dto.dart
├── auth_user_mapper.dart
└── supabase_auth_repository.dart
```

### 5.3 `application`

사용자 행동에 따른 작업 흐름과 화면에서 사용하는 상태를 관리한다.

포함할 수 있는 항목:

- Riverpod Provider와 Controller
- UseCase 또는 Service
- 화면 상태 모델
- 여러 repository 호출의 순서와 조합

비즈니스 작업은 가능한 한 Repository interface를 대상으로 작성한다. Repository 구현체를 Provider에 연결하기 위한 dependency binding은 `application` 또는 feature 루트의 명시적인 provider 파일에 둘 수 있다.

```text
features/auth/application/
├── auth_repository_provider.dart
├── auth_session_provider.dart
├── login_controller.dart
└── login_state.dart
```

### 5.4 `presentation`

Flutter UI와 사용자 입력 처리를 담당한다.

포함할 수 있는 항목:

- Screen과 Page
- feature 전용 Widget
- Dialog, BottomSheet
- UI 전용 formatter와 view model

presentation은 API SDK를 직접 호출하지 않는다. 저장, 조회, 삭제 같은 작업은 application의 Controller나 Provider를 통해 요청한다.

```text
features/auth/presentation/
├── login_screen.dart
└── widgets/
    └── login_form.dart
```

---

## 6. 의존성 방향

권장 최상위 의존성 방향은 다음과 같다.

```text
app ───────────────▶ features
 │                    │
 │                    ├────────▶ shared
 │                    └────────▶ core
 ├──────────────────▶ shared
 └──────────────────▶ core

shared ─────────────▶ core  (필요할 때만)
core ───────────────▶ 외부 패키지와 Dart/Flutter SDK
```

금지 방향:

```text
core     ─X─▶ features
shared   ─X─▶ features
shared   ─X─▶ app
feature A 내부 ─X─▶ feature B 내부
```

feature 내부의 기본 방향은 다음과 같다.

```text
presentation ─▶ application ─▶ domain
      │                │
      ├───────────────▶ domain
      └───────────────▶ shared

data ─────────▶ domain
data ─────────▶ core
application ──▶ core  (필요할 때만)
```

의존성 주입을 위한 provider binding에서만 `application`이 `data`의 구현체를 참조할 수 있다. 이 예외는 파일 역할이 드러나도록 분리하고, 비즈니스 로직이 구현체에 직접 묶이지 않게 한다.

---

## 7. Feature 간 협력 규칙

### 7.1 기본 규칙

feature A가 feature B의 내부 폴더를 직접 import하지 않는다.

```dart
// 금지
import 'package:app/features/auth/data/auth_token_store.dart';
import 'package:app/features/auth/presentation/widgets/user_avatar.dart';
```

다음 순서로 해결한다.

1. `app`에서 두 feature를 조립할 수 있는지 확인한다.
2. feature의 공개 API를 통해 필요한 기능만 노출한다.
3. 두 feature가 공유하는 범용 개념이면 `shared` 또는 `core`로 이동한다.
4. 실제로 하나의 도메인이라면 feature 분리가 잘못된 것은 아닌지 검토한다.

### 7.2 공개 API 파일

Flutter에서는 React의 `index.ts`와 같은 공개 진입점이 반드시 필요하지 않다. 다음 조건일 때 `<feature>.dart` 파일을 사용할 수 있다.

- 외부 feature나 `app`에서 여러 파일을 안정적으로 사용한다.
- 내부 폴더 구조 변경의 영향을 외부에 숨길 필요가 있다.
- 공개 대상과 내부 대상을 명확히 구분할 가치가 있다.

공개 가능한 대상:

- 앱 라우터가 사용하는 Screen 또는 Route builder
- 다른 feature가 소비하는 안정된 domain type
- 외부 조립에 필요한 Provider
- feature가 제공하는 명시적인 facade

공개하지 않을 대상:

- DTO와 Mapper
- Repository 구현체
- feature 전용 local widget
- 내부 Controller 구현 세부사항
- SDK adapter

barrel file을 과도하게 만들면 순환 의존성과 숨겨진 import가 생길 수 있으므로 feature 외부 경계에만 제한적으로 사용한다.

---

## 8. Riverpod과 Flutter UI 적용 원칙

- Provider와 Controller는 feature의 `application`에 둔다.
- feature 전용 Provider를 `core`나 전역 `providers/` 폴더에 모으지 않는다.
- 화면은 Controller가 제공하는 작업과 상태를 사용하고 Repository를 직접 호출하지 않는다.
- `BuildContext`가 필요한 UI 책임과 저장/API 책임을 분리한다.
- `TextEditingController`, `FocusNode`, animation처럼 생명주기가 있는 UI 객체는 presentation에서 관리한다.
- 공통 Widget은 특정 feature Provider를 직접 읽지 않고 값과 callback을 전달받는다.
- code generation을 사용하는 프로젝트는 생성 파일을 원본 파일과 같은 feature 안에 둔다.

---

## 9. 테스트 구조

테스트도 실제 feature 구조를 가능한 한 따라간다.

```text
test/
├── features/
│   └── auth/
│       ├── domain/
│       ├── data/
│       ├── application/
│       └── presentation/
├── shared/
└── app/
```

권장 검증 범위:

- `domain`: 순수 규칙과 값 검증
- `data`: DTO 변환, Mapper, Repository 구현
- `application`: Controller 상태 전이와 오류 처리
- `presentation`: 핵심 사용자 흐름과 위젯 상태
- `app`: 라우팅과 feature 조립

파일 이동만 하는 PR에서도 import 오류를 확인하기 위해 최소한 `flutter analyze`를 실행한다.

---

## 10. 기존 프로젝트 전환 절차

전체 `lib/`를 한 번에 이동하지 않는다. 사용자 동작을 유지하면서 feature 하나씩 전환한다.

### 단계 1. 현재 구조 조사

- 주요 화면과 사용자 흐름 목록
- Provider, Repository, Model의 사용처
- feature 간 직접 import 관계
- 실제로 여러 기능에서 사용하는 공통 코드
- 테스트와 Analytics 책임 위치

### 단계 2. 목표 feature 경계 결정

기술 파일 종류가 아니라 사용자의 기능을 기준으로 feature 목록을 만든다.

```text
인증 → auth
운동일지 → workout_journal
루틴 → routine
설정 → settings
```

화면 하나를 무조건 feature 하나로 만들지 않는다. 같은 데이터와 규칙을 공유하는 화면은 하나의 feature 안에 둘 수 있다.

### 단계 3. 최상위 골격 생성

현재 프로젝트에 필요한 폴더만 만든다.

```text
lib/app/
lib/core/
lib/features/
lib/shared/
```

기존 코드를 즉시 삭제하지 않는다.

### 단계 4. 수직 기능 하나 전환

의존성이 적고 테스트하기 쉬운 feature부터 다음 순서로 옮긴다.

1. domain model과 repository 계약
2. data 구현과 mapper
3. application provider/controller
4. presentation screen/widget
5. router 연결
6. 관련 테스트

파일 위치만 바꾸는 작업과 동작을 변경하는 리팩터링은 가능한 한 별도 commit 또는 PR로 나눈다.

### 단계 5. Feature 경계 정리

- 다른 feature가 내부 파일을 직접 참조하는지 확인한다.
- 필요한 공개 API만 노출한다.
- 중복 코드가 실제로 확인되면 `shared`로 이동한다.
- feature 용어가 남아 있는 공통 코드는 다시 해당 feature로 돌린다.

### 단계 6. 기존 Layer-first 폴더 제거

모든 사용처와 테스트가 새 경로로 전환된 후 비어 있는 기존 폴더를 제거한다.

```text
models/
repositories/
screens/
providers/
```

### 단계 7. 통합 검증

```bash
flutter analyze
flutter test
```

프로젝트에 build runner를 사용한다면 다음도 실행한다.

```bash
dart run build_runner build --delete-conflicting-outputs
```

---

## 11. 전환 PR 운영 기준

- feature 하나 또는 명확한 하위 흐름 하나를 PR 단위로 잡는다.
- 파일 이동과 기능 변경을 한 PR에 과도하게 섞지 않는다.
- 기존 사용자 동작과 API 계약을 유지한다.
- 관련 없는 공통화와 이름 변경을 함께 진행하지 않는다.
- PR 본문에 이전 경로와 새 경로를 기록한다.
- 실행한 분석과 테스트 결과를 남긴다.
- 남아 있는 직접 import와 후속 전환 대상을 기록한다.

권장 PR 설명 항목:

```md
## 전환 대상

- 대상 feature:
- 이전 경로:
- 새 경로:

## 경계 변경

- 새 공개 API:
- 제거한 내부 직접 참조:

## 검증

- [ ] flutter analyze
- [ ] flutter test
```

---

## 12. 피해야 할 구조

### 12.1 거대한 `shared`

```text
shared/
├── auth_service.dart
├── workout_provider.dart
├── study_session_mapper.dart
└── settings_screen.dart
```

특정 feature 이름과 규칙이 들어간 코드는 공통 코드가 아니다.

### 12.2 모든 기능에 동일한 계층 강제

```text
simple_feature/
├── domain/
├── data/
├── application/
├── presentation/
├── usecases/
├── datasources/
├── dto/
└── mappers/
```

내용이 없는 계층과 전달만 하는 클래스는 유지보수 비용만 늘린다.

### 12.3 화면에서 SDK 직접 호출

```dart
onPressed: () async {
  await Supabase.instance.client.from('records').insert(...);
}
```

presentation과 데이터 접근 책임이 섞이므로 application과 repository를 거치게 한다.

### 12.4 Feature 내부 깊은 경로 참조

```dart
import '../../features/auth/data/internal/token_mapper.dart';
```

다른 feature의 내부 구현에 결합되므로 공개 API, app 조립 또는 공통 계약으로 대체한다.

### 12.5 기술 이름으로 Feature 구성

```text
features/api/
features/models/
features/riverpod/
```

이는 이름만 `features`인 Layer-first 구조다.

---

## 13. 프로젝트별 결정 항목

이 문서를 프로젝트에 복사한 뒤 다음 항목을 `AGENTS.md` 또는 프로젝트 아키텍처 결정 문서에 명시한다.

- 실제 feature 목록과 경계
- `core`와 `shared`의 프로젝트별 책임
- feature 공개 API 파일 사용 여부
- Repository binding 위치
- Riverpod code generation 사용 여부
- API DTO와 domain model 분리 기준
- feature 간 허용된 의존성 예외
- legacy 코드 보존 또는 제거 전략
- 필수 분석, 테스트, codegen 명령

프로젝트의 기존 구조와 이 문서가 충돌하면 무조건 대규모로 맞추지 않는다. 프로젝트별 결정 문서에 예외와 전환 계획을 기록하고 단계적으로 적용한다.

---

## 14. 완료 체크리스트

### 신규 기능

- [ ] 사용자 기능 또는 업무 도메인 기준으로 feature 이름을 정했다.
- [ ] 필요한 계층만 만들었다.
- [ ] presentation에서 SDK와 Repository 구현체를 직접 호출하지 않는다.
- [ ] 다른 feature의 내부 경로를 직접 import하지 않는다.
- [ ] 공통 코드는 특정 feature를 알지 않는다.
- [ ] 외부 공개 대상만 명시적으로 노출한다.
- [ ] 관련 테스트와 분석을 실행했다.

### 기존 구조 전환

- [ ] 현재 import 관계와 사용자 흐름을 조사했다.
- [ ] feature 하나씩 수직으로 전환한다.
- [ ] 파일 이동과 동작 변경을 가능한 한 분리했다.
- [ ] 기존 API 계약과 사용자 동작을 유지했다.
- [ ] 남은 legacy 경로와 후속 작업을 기록했다.
- [ ] `flutter analyze`와 `flutter test` 결과를 남겼다.
