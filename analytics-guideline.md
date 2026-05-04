# Analytics 설계 가이드

이 문서는 Analytics 이벤트를 추가하거나 수정할 때 적용하는 공통 기준이다.
프로젝트별 이벤트 목록은 별도 카탈로그 문서에서 관리한다.

## 목적

Analytics는 사용자가 어떤 행동을 했고, 그 행동이 어떤 결과로 이어졌는지 측정하기 위해 사용한다.

주요 목적:

- 사용자가 실제로 가치를 느끼는 기능 확인
- 가입, 작성, 결제, 광고, 백업 같은 핵심 퍼널 추적
- 신규 사용자가 활성화 지점에 도달하는 비율 측정
- 실험 또는 UI 변경의 결과 비교
- 실패 지점과 기술적 병목 확인

## 원칙

1. PII를 수집하지 않는다.
2. 사용자 의도와 결과를 분리한다.
3. 이벤트명과 파라미터는 상수로 관리한다.
4. 화면 조회는 분석 도구의 기본 screen view 기능을 우선한다.
5. 데이터 로드, 내부 처리, bootstrap 완료 같은 기술 이벤트는 `system_*`로 기록한다.
6. 실패 이벤트는 가능한 한 `fail_*`로 기록하고, 오류 분류 파라미터를 함께 보낸다.
7. 이벤트를 추가하거나 수정하면 코드와 문서를 같은 PR에서 함께 갱신한다.

## 명명 규칙

이벤트명은 lowercase snake_case를 사용한다.

```txt
<prefix>_<domain>_<detail>
```

예:

```txt
submit_exam_save
success_exam_create
fail_search_result
system_migration_start
```

| Prefix | 의미 | 예 |
|---|---|---|
| `view_*` | 화면, 모달, 상세 UI 진입 의도 | `view_photo_full` |
| `click_*` | 버튼 또는 클릭성 UI 탭 | `click_search_open` |
| `submit_*` | 폼 제출 또는 요청 시도 | `submit_backup_upload` |
| `success_*` | 실제 결과 성공 | `success_exam_create` |
| `fail_*` | 실제 결과 실패 | `fail_exam_delete` |
| `show_*` | 자동 노출 | `show_reward_entry` |
| `change_*` | 설정, 필터, 상태 변경 | `change_order_mode` |
| `system_*` | 시스템, 기술 처리, 내부 상태 | `system_migration_start` |
| `activation_*` | 최초 활성화 지점, 보통 1회 발화 | `activation_first_item_create` |

## 책임 위치

| 위치 | 측정 대상 | Prefix |
|---|---|---|
| UI | 사용자 의도 | `view_*`, `click_*`, `submit_*`, `change_*` |
| Service | 실제 결과 | `success_*`, `fail_*`, `system_*` |
| Repository | Analytics 기록하지 않음 | 없음 |

같은 행동을 UI와 Service 양쪽에 중복 기록하지 않는다.
UI는 사용자가 시도했다는 사실을 기록하고, Service는 실제 저장, 삭제, 네트워크 요청, 광고 로드 같은 결과를 기록한다.

예:

```txt
UI: submit_item_save
Service 성공: success_item_create
Service 실패: fail_item_create
```

Repository는 DB나 API 접근 책임만 가진다.
Repository에서 Analytics를 직접 남기면 데이터 접근 계층이 제품 분석 정책에 묶이므로 피한다.

## Screen View 정책

화면 보고서는 Firebase Analytics, GA, Amplitude 같은 도구가 제공하는 기본 screen view 또는 page view 기능을 우선한다.

`view_*` 커스텀 이벤트는 아래처럼 사용자가 특정 UI에 진입했다는 의도를 따로 보고 싶을 때만 사용한다.

- 모달 열림
- 이미지 전체 보기
- 특정 기능 상세 진입
- 라우팅 화면은 아니지만 분석상 독립 의미가 있는 UI 노출

앱 시작 bootstrap, provider 준비, 데이터 preload 같은 기술 화면은 일반적으로 screen view에 포함하지 않는다.
사용자가 의미 있게 도달한 첫 화면을 첫 screen view로 기록한다.

## 파라미터 기준

파라미터는 분석에 필요한 최소한만 보낸다.

전송 가능한 값:

- 개수
- 길이
- boolean 또는 enum
- 범위 bucket
- 오류 코드
- 예외 클래스명
- 기능 진입점

전송하지 않는 값:

- 사용자 입력 원문
- 이메일, 이름, 전화번호
- 검색어 원문
- 파일 경로와 파일 내용
- 인증 토큰
- 계정 식별 정보

실패 이벤트에는 가능한 한 오류 분류를 포함한다.

```txt
fail_item_delete
params: error_type
```

광고나 외부 SDK처럼 코드와 메시지가 제공되는 경우에는 `error_code`, `error_message`를 사용할 수 있다.
단, 메시지에 PII가 들어갈 가능성이 있으면 저장하지 않는다.

## 이벤트 추가 절차

Analytics 이벤트를 추가하거나 수정할 때는 아래 순서를 따른다.

1. 이벤트 목적을 정한다.
2. 사용자 의도인지 결과인지 판단한다.
3. prefix와 domain을 정한다.
4. 필요한 파라미터를 정하고 PII 여부를 확인한다.
5. 이벤트명과 파라미터명을 상수에 추가한다.
6. 책임 위치에 맞는 코드에 호출을 추가한다.
7. 프로젝트별 이벤트 카탈로그 문서를 갱신한다.
8. PR 테스트 항목에 DebugView 확인 필요 여부를 적는다.

## PR 체크리스트

Analytics 변경이 있는 PR은 아래 내용을 확인한다.

- [ ] 이벤트명이 lowercase snake_case인가
- [ ] prefix가 의미에 맞는가
- [ ] UI와 Service 책임이 분리되어 있는가
- [ ] 문자열 리터럴이 아니라 상수를 사용하는가
- [ ] 파라미터에 PII가 없는가
- [ ] 실패 이벤트에 오류 분류가 있는가
- [ ] 프로젝트별 Analytics 문서가 갱신되었는가
- [ ] DebugView 확인 필요 여부가 PR 테스트 항목에 적혀 있는가

## 프로젝트별 문서 분리

이 가이드는 공통 원칙만 다룬다.
프로젝트별 문서에는 아래 항목을 별도로 둔다.

- 이벤트 카탈로그
- User Property 목록
- Remote Config 키
- Activation 이벤트
- 광고, 결제, 백업 같은 도메인별 퍼널
- DebugView 확인 방법
- 변경 이력
