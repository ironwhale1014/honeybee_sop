# AGENTS.md

## 역할

이 저장소에서 작업하는 AI 코딩 에이전트는 프로젝트의 기존 규칙을 우선한다.
코드 변경은 작게, 명확하게, Issue 단위로 진행한다.

## 필수 작업 흐름

모든 작업은 아래 순서를 따른다.

1. GitHub Issue 생성
2. Issue 번호 기반 Branch 생성
3. 작업 전 현재 Branch 확인
4. 개발
5. 테스트/검증
6. Commit
7. 원격 Branch Push
8. PR 생성
9. PR 내용 확인
10. Merge

작은 문서 수정이나 단순 버그 수정도 예외 없이 Issue에서 시작한다.
단, 사용자가 명시적으로 Issue/PR 없이 처리하라고 지시한 경우에는 사용자 지시를 우선한다.

## 금지 사항

- Issue 없이 작업을 시작하지 않는다.
- `main`에서 직접 개발하거나 commit하지 않는다.
- 다른 Issue Branch 위에서 새 작업 Branch를 만들지 않는다.
- 관련 없는 미커밋 변경을 staging/commit/PR에 포함하지 않는다.
- 사용자가 만든 변경을 임의로 되돌리지 않는다.
- `git reset --hard`, `git checkout --` 같은 파괴적 명령은 명시 요청 없이는 사용하지 않는다.
- PR 없이 `main`에 merge하지 않는다.

## Branch 규칙

Branch는 항상 `origin/main` 기준으로 만든다.

```bash
git checkout -b fix/issue-12 origin/main
```

Branch 이름 형식:

```txt
<type>/issue-<number>
```

예:

```txt
feat/issue-12
fix/issue-13
refactor/issue-14
docs/issue-15
style/issue-16
chore/issue-17
```

작업 전 항상 확인한다.

```bash
git status --short --branch
```

## Commit 규칙

커밋 메시지는 Conventional Commits 형식을 따른다.

```txt
<type>(<scope>): <subject> #<issue-number>
```

예:

```txt
fix(auth): 로그인 실패 메시지 표시 오류 수정 #12
docs(workflow): GitHub 작업 흐름 규칙 보강 #13
```

커밋 전 확인한다.

```bash
git status --short
git diff --staged
```

확인할 것:

- staging된 파일이 이번 Issue 범위와 일치하는가
- 관련 없는 미커밋 변경이 섞이지 않았는가
- 커밋 메시지에 Issue 번호가 들어갔는가

## PR 규칙

PR 제목 형식:

```txt
[TYPE] 작업 제목 (#이슈번호)
```

PR 본문에는 반드시 연결 이슈를 포함한다.

```md
## 연결 이슈

Closes #이슈번호
```

PR 생성 전 확인한다.

```bash
git status --short --branch
git log --oneline origin/main..HEAD
```

확인할 것:

- 현재 Branch가 `main`이 아닌 Issue Branch인가
- PR에 들어갈 commit이 이번 Issue 작업만 포함하는가
- Files changed에 관련 없는 파일이 없는가

## 검증 규칙

프로젝트 종류에 맞는 검증 명령을 실행한다.

예:

```bash
# Flutter
flutter analyze
flutter test

# React
npm run build
npm test
```

전체 검증이 기존 문제로 실패하면, 이번 변경 파일 또는 변경 범위의 검증 결과를 별도로 기록한다.

## Analytics 규칙

Analytics를 사용하는 프로젝트라면 별도 Analytics 문서를 기준으로 한다.

- 화면 조회는 기본 screen view 기능을 우선한다.
- `view_*`는 실제 사용자 UI 진입 의도에만 사용한다.
- 데이터 로드, 내부 처리, provider 준비 완료 이벤트는 `system_*`를 사용한다.
- 이벤트명, 파라미터, 책임 위치를 문서와 함께 갱신한다.
- DebugView 확인이 필요한 경우 PR 테스트 항목에 적는다.

Analytics를 사용하지 않는 프로젝트라면 이 섹션을 제거하거나 "사용하지 않음"으로 명시한다.

## Dirty Worktree 규칙

작업 중 기존 미커밋 변경이 있을 수 있다.

- 내 작업과 무관하면 건드리지 않는다.
- staging에도 포함하지 않는다.
- 같은 파일에 변경이 섞여 있으면 먼저 diff를 읽고 사용자 변경을 보존한다.
- 충돌하거나 판단이 어려우면 작업을 멈추고 사용자에게 확인한다.

## 실수 복구

`main`에 직접 commit했다면 push하지 않는다.

1. 새 Issue를 만든다.
2. 현재 commit에서 Issue Branch를 만든다.
3. commit 메시지의 Issue 번호를 새 Issue 번호로 고친다.
4. `main` 포인터를 `origin/main`으로 되돌린다.
5. Issue Branch를 push하고 PR을 만든다.

예:

```bash
git checkout -b fix/issue-8
git commit --amend -m "fix(common): 작업 내용 #8"
git branch -f main origin/main
git push -u origin fix/issue-8
```

## 참고 문서

- Workflow: `<복사한 경로>/development-workflow.md`
- Commit convention: `<복사한 경로>/commit-convention.md`
- Analytics guideline: `<복사한 경로>/analytics-guideline.md`
