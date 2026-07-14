# 개발 매뉴얼

이 저장소는 새 프로젝트를 시작할 때 복사해서 사용할 수 있는 표준 개발 매뉴얼 원본이다.

새 프로젝트에 적용할 때는 이 문서 묶음을 프로젝트의 `docs/` 아래로 복사해 사용한다.
현재 프로젝트의 실제 규칙은 복사 후 프로젝트 루트에 둔 `AGENTS.md`가 우선한다.

## 문서 구성

| 문서 | 목적 |
|---|---|
| `AGENTS.md` | 새 프로젝트 루트에 복사해서 사용할 AI 에이전트 작업 규칙 템플릿 |
| `development-workflow.md` | Issue, Branch, PR, Merge 흐름 표준 |
| `commit-convention.md` | Conventional Commits 기반 커밋 메시지 규칙 |
| `analytics-guideline.md` | Analytics 이벤트 설계와 문서화 공통 기준 |
| `flutter-feature-first-architecture.md` | Flutter Feature-first 구조, 의존성 경계, 점진적 전환 기준 |
| `templates/issue-template.md` | Issue 작성용 상세 템플릿 |
| `templates/pr-template.md` | PR 작성용 상세 템플릿 |

## 새 프로젝트 적용 순서

1. `AGENTS.md`를 새 프로젝트 루트에 복사한다.
2. 프로젝트 이름, 검증 명령, 참고 문서 경로를 새 프로젝트에 맞게 수정한다.
3. `development-workflow.md`와 `commit-convention.md`를 새 프로젝트의 `docs/`에 복사한다.
4. Flutter 프로젝트라면 `flutter-feature-first-architecture.md`를 복사하고, 실제 feature 목록과 허용할 의존성 예외를 프로젝트 문서에 기록한다.
5. Analytics를 사용하는 프로젝트라면 `analytics-guideline.md`를 복사하고, 프로젝트별 이벤트 카탈로그 문서를 별도로 만든다.
6. GitHub Issue/PR 템플릿이 필요하면 `templates/`의 파일을 `.github/` 하위 템플릿으로 옮겨 사용한다.

## 문서 역할 기준

| 위치 | 역할 |
|---|---|
| `AGENTS.md` | 현재 프로젝트에서 반드시 따라야 하는 짧은 필수 규칙 |
| `docs/` | 현재 프로젝트의 상세 설계, 구현 기록, 도메인별 운영 문서 |

`AGENTS.md`에는 에이전트가 매 작업마다 지켜야 하는 핵심 규칙만 둔다. 상세 설명, 예시, 판단 기준은 `docs/`로 분리한다.

## 프로젝트별로 바꿔야 하는 항목

새 프로젝트에 복사한 뒤 아래 항목은 반드시 확인한다.

| 항목 | 예 |
|---|---|
| 프로젝트 이름 | `my_exam_note`, `exam-note-landing-page` |
| 검증 명령 | `flutter analyze`, `flutter test`, `npm run build` |
| 문서 경로 | `docs/analytics_overview.md`, `docs/github-workflow.md` |
| Flutter 아키텍처 | feature 목록, 공개 API 사용 여부, `core`/`shared` 책임, 허용 의존성 예외 |
| Analytics 사용 여부 | Firebase Analytics 사용, 미사용, 다른 분석 도구 사용 |
| 배포/호스팅 방식 | Play Store, Vercel, GitHub Pages, 자체 서버 |
| Issue/PR 예외 정책 | 사용자가 명시한 경우만 예외 허용 |

## 운영 원칙

- 작업은 작게 나누고 Issue 단위로 진행한다.
- Branch는 항상 `origin/main` 기준으로 만든다.
- 커밋 메시지에는 Issue 번호를 포함한다.
- PR 본문에는 연결 Issue와 실제 검증 결과를 남긴다.
- Flutter 구조를 추가하거나 변경하면 feature 경계와 의존성 방향을 함께 검토한다.
- Analytics 이벤트를 추가하거나 수정하면 코드와 문서를 함께 갱신한다.
- 사용자가 명시적으로 다른 지시를 내리면 그 지시를 우선하되, 어떤 규칙을 예외 처리했는지 남긴다.
