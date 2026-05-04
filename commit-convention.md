# Git 커밋 메시지 규칙

커밋 메시지는 변경 이유와 범위를 나중에 추적하기 위한 기록이다.
혼자 작업하는 프로젝트라도 몇 달 뒤의 자신이 이해할 수 있게 작성한다.

## 기본 형식

```txt
<type>(<scope>): <subject> #<issue-number>
```

예:

```txt
feat(auth): 소셜 로그인 버튼 추가 #12
fix(search): 빈 검색어 처리 오류 수정 #13
docs(workflow): GitHub 작업 흐름 규칙 보강 #14
```

여러 파일을 바꾸거나 판단 이유가 필요한 경우 body를 추가한다.
Issue 번호는 한 줄 커밋이면 subject 끝에, body가 있으면 footer에 남긴다.

```txt
refactor(analytics): 이벤트 책임 위치 정리

UI는 사용자 의도 이벤트만 기록하고 Service는 성공/실패 결과만
기록하도록 책임을 분리한다.

Refs #15
```

Issue 자동 종료는 커밋이 아니라 PR 본문의 `Closes #이슈번호`에서 처리한다.

## Type 기준

| type | 언제 쓰나 | 예 |
|---|---|---|
| `feat` | 사용자 기능 추가 | 검색 필터 추가 |
| `fix` | 버그 수정 | 삭제 실패 시 에러 표시 누락 수정 |
| `refactor` | 동작 변화 없는 구조 개선 | Repository 예외 처리 흐름 정리 |
| `test` | 테스트 추가 또는 수정 | Service 실패 케이스 테스트 추가 |
| `docs` | 문서 변경 | 테스트 가이드 문서 추가 |
| `style` | 포맷, 세미콜론 등 의미 없는 변경 | 들여쓰기 정리 |
| `chore` | 설정, 스크립트, 빌드, 관리 작업 | 패키지 설정 갱신 |
| `perf` | 성능 개선 | 목록 쿼리 최적화 |

판단 기준:

```txt
사용자가 체감하는 새 기능인가? → feat
기존 동작의 오류를 고쳤는가? → fix
동작은 같고 내부 구조만 좋아졌는가? → refactor
테스트 코드나 검증 코드인가? → test
문서만 바뀌었는가? → docs
포맷처럼 동작 의미가 없는 변경인가? → style
설정, 의존성, 빌드, 관리 작업인가? → chore
성능 개선이 핵심 목적인가? → perf
```

## Scope 기준

`scope`는 변경이 영향을 주는 도메인이나 모듈 이름을 쓴다.

예:

```txt
exam
photo
category
search
analytics
workflow
manual
```

여러 모듈에 걸친 변경이면 scope를 생략할 수 있다.

```txt
refactor: 전체 에러 처리 정책 정리 #20
```

## Subject 작성 원칙

- 한국어로 작성한다.
- 명령형 명사형으로 끝낸다.
- 끝에 마침표를 찍지 않는다.
- 무엇을 바꿨는지 한 줄로 알 수 있게 쓴다.
- 너무 넓은 표현을 피한다.

좋은 예:

```txt
fix(exam): 삭제 실패 시 사용자 에러 표시 추가 #12
test(repository): 검색 실패 예외 전파 테스트 추가 #13
docs(manual): 표준 개발 매뉴얼 템플릿 추가 #14
```

피해야 할 예:

```txt
수정
버그 수정
작업 완료
업데이트
여러가지 개선
```

## 커밋 단위

하나의 커밋에는 하나의 목적만 담는다.

나쁜 예:

```txt
feat(search): 검색 필터 추가 및 문서 정리 #12
```

좋은 예:

```txt
feat(search): 검색 필터 추가 #12
docs(search): 검색 필터 동작 문서화 #12
```

리팩토링과 기능 추가가 함께 필요하다면 가능하면 커밋을 나눈다.
단, 작은 문서 보강처럼 변경 목적이 하나로 설명되는 경우에는 하나의 커밋으로 묶을 수 있다.

## 커밋 전 확인

커밋 전 반드시 staging 범위와 diff를 확인한다.

```bash
git status --short
git diff --staged
```

확인할 것:

- 이번 Issue와 관련 있는 파일만 staging 되었는가
- 사용자 또는 다른 작업자의 변경을 포함하지 않았는가
- 커밋 메시지에 Issue 번호가 들어갔는가
- 테스트나 문서 갱신이 필요한 변경을 빠뜨리지 않았는가

## Amend 기준

push 전 커밋 메시지에 Issue 번호를 빠뜨렸거나 오타가 있으면 amend로 고친다.

```bash
git commit --amend -m "fix(scope): 작업 내용 #12"
```

이미 push한 커밋을 수정해야 하면 force push가 필요할 수 있다.
공유 브랜치라면 임의로 force push하지 말고 먼저 확인한다.
