# GitHub 작업 흐름 표준

이 문서는 Issue, Branch, Pull Request를 사용해 작업을 관리하는 표준 규칙이다.
목표는 작업 이유와 검증 결과를 GitHub 기록으로 남기고, 변경 범위를 Issue 단위로 작게 유지하는 것이다.

## Type 규칙

Issue 제목, Branch 이름, Commit 메시지에서 사용하는 type은 아래 기준을 따른다.

| Type | Issue 제목 | Branch | Commit | 의미 |
|---|---|---|---|---|
| FEAT / feat | O | O | O | 사용자 기능 추가 |
| FIX / fix | O | O | O | 버그 수정 |
| REFACTOR / refactor | O | O | O | 동작 변화 없는 구조 개선 |
| TEST / test | O | O | O | 테스트 추가 또는 수정 |
| DOCS / docs | O | O | O | 문서 작업 |
| STYLE / style | O | O | O | 포맷, 세미콜론 등 의미 없는 변경 |
| CHORE / chore | O | O | O | 설정, 스크립트, 빌드, 관리 작업 |
| PERF / perf | O | O | O | 성능 개선 |
| EXPERIMENT | O | X | X | 지표 검증 목적의 실험 |

`EXPERIMENT`는 Issue와 PR 제목에서만 사용한다. Branch와 Commit에서는 실제 변경 성격에 맞는 기존 type을 사용한다.

## 전체 흐름

모든 작업은 아래 순서를 지킨다.
작은 문서 수정이나 단순 버그 수정도 예외 없이 Issue에서 시작한다.

```txt
Issue 생성
  ↓
Issue 번호 기반 Branch 생성
  ↓
작업 전 현재 Branch 확인
  ↓
개발
  ↓
테스트/검증
  ↓
Commit
  ↓
원격 Branch Push
  ↓
PR 생성
  ↓
PR 내용 확인
  ↓
Merge
  ↓
Issue 자동 종료
```

단, 사용자가 명시적으로 Issue/PR 없이 처리하라고 지시한 경우에는 그 지시를 우선한다.
이 경우에도 작업 전 현재 Branch와 변경 범위를 확인하고, 관련 없는 변경은 포함하지 않는다.
작업 후에는 어떤 규칙을 예외 처리했는지 최종 응답에 남긴다.

표준 명령 흐름:

```bash
gh issue create --title "[FIX] 작업 제목" --body "..."
git checkout -b fix/issue-12 origin/main
git status --short --branch

# 개발 및 검증

git add <changed-files>
git status --short
git diff --staged
git commit -m "fix(scope): 작업 내용 #12"
git push -u origin fix/issue-12
gh pr create --title "[FIX] 작업 제목 (#12)" --body "..."
```

## Issue 규칙

Issue는 작업의 시작점이다.

제목 형식:

```txt
[TYPE] 대상 + 목표
```

좋은 예:

```txt
[FEAT] 작성 완료 후 리워드 진입점 노출
[FIX] 사진 삭제 표시 후 저장 시 상태 꼬임 수정
[REFACTOR] Analytics success 이벤트 호출 위치 정리
[DOCS] GitHub 이슈 브랜치 PR 작업 흐름 정리
[EXPERIMENT] 정렬 바텀시트 전환율 확인
```

Issue는 가능하면 한국어로 작성한다.
기존 Issue의 문체와 섹션 구조를 따른다.
본문에는 배경, 문제 또는 정리 대상, 목표, 작업 내용, 완료 조건을 남긴다.

권장 양식은 함께 복사한 `templates/issue-template.md`를 사용한다.

Issue 작성 원칙:

- 단순 TODO가 아니라 작업 이유를 남긴다.
- 이번 작업에서 제외하는 범위가 있으면 `메모`에 적는다.
- 후속 Issue와 연결되는 작업이면 Issue 번호나 제목을 적는다.
- Analytics 변경이 있으면 이벤트명, 파라미터, 책임 위치, 문서 갱신 여부를 완료 조건에 포함한다.

## Branch 규칙

기본 원칙:

```txt
Issue 1개 = Branch 1개
```

Branch는 `origin/main` 기준으로 만든다.
다른 작업 Branch 위에서 새 작업 Branch를 만들지 않는다.

Branch 이름:

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

Branch 생성 전 확인:

```bash
git status --short --branch
```

현재 Branch가 `main`이 아니거나 다른 Issue Branch라면, 새 Branch는 반드시 `origin/main`에서 만든다.

```bash
git checkout -b docs/issue-15 origin/main
```

미커밋 변경이 이미 있다면 다음 기준을 적용한다.

- 이번 Issue와 관련 있는 변경만 새 Branch에서 함께 작업한다.
- 이번 Issue와 무관한 변경은 staging/commit에 포함하지 않는다.
- checkout이 막히면 임의로 되돌리지 말고, 어떤 파일이 충돌하는지 확인한 뒤 사용자와 정리한다.

## Commit 규칙

커밋 메시지는 함께 복사한 `commit-convention.md`를 따른다.

기본 형식:

```txt
<type>(<scope>): <subject> #<issue-number>
```

예:

```txt
feat(exam): 정렬 바텀시트 추가 #12
fix(photo): 삭제 표시 상태 갱신 오류 수정 #13
docs(workflow): GitHub 작업 흐름 규칙 보강 #15
```

커밋 전 확인:

```bash
git status --short
git diff --staged
```

확인할 것:

- staging된 파일이 이번 Issue 범위와 일치하는가
- 미커밋 변경 중 이번 Issue와 무관한 파일이 섞여 있지 않은가
- 커밋 메시지에 Issue 번호가 들어갔는가

## PR 규칙

PR은 작업 결과 보고서다.

PR 제목 형식:

```txt
[TYPE] 작업 제목 (#이슈번호)
```

PR 본문에는 반드시 연결 이슈를 포함한다.

```md
## 연결 이슈

Closes #이슈번호
```

권장 양식은 함께 복사한 `templates/pr-template.md`를 사용한다.

PR 본문 작성 원칙:

- 한국어로 작성하고 기존 PR 문체를 따른다.
- "무엇을 바꿨는지"뿐 아니라 "왜 바꿨는지"를 남긴다.
- 변경 전/후를 분리해 리뷰어가 diff를 보기 전에 의도를 이해할 수 있게 한다.
- 이번 PR에서 일부러 하지 않은 작업을 `변경하지 않은 것`에 적어 범위 오해를 줄인다.
- 테스트는 체크박스만 두지 말고 실제 실행한 명령과 결과를 기록한다.
- Analytics 변경이 있으면 이벤트명, 파라미터, 책임 위치, 문서 갱신 여부를 적는다.

PR 생성 전 확인:

```bash
git status --short --branch
git log --oneline origin/main..HEAD
```

확인할 것:

- 현재 Branch가 `main`이 아닌 Issue Branch인가
- PR에 들어갈 commit이 이번 Issue 작업만 포함하는가
- 관련 없는 미커밋 변경이 staging되어 있지 않은가

PR 생성 후 확인:

- PR 제목이 `[TYPE] 작업 제목 (#이슈번호)` 형식인가
- PR 본문에 `## 연결 이슈`와 `Closes #이슈번호`가 있는가
- 테스트 체크리스트가 실제 수행 결과와 일치하는가
- Files changed에 이번 Issue와 무관한 파일이 없는가
- CI 또는 로컬 검증 결과가 실패하면 merge하지 않는다.

## 실수 복구 규칙

### `main`에 직접 커밋한 경우

`main`에 직접 커밋했다면 push하지 않는다.
이미 작업한 커밋은 버리지 말고 Issue Branch로 옮긴다.

1. 새 Issue를 만든다.
2. 잘못 만든 현재 커밋 위치에서 Issue Branch를 만든다.
3. 커밋 메시지의 Issue 번호가 새 Issue와 다르면 고친다.
4. 실수한 로컬 `main`에만 문제가 있고 미커밋 변경이 없다는 것을 확인한 뒤, `main` 포인터를 원격 기준으로 되돌린다.
5. Issue Branch를 push하고 PR을 만든다.

예:

```bash
git checkout -b fix/issue-8
git commit --amend -m "fix(common): 작업 내용 #8"
git branch -f main origin/main
git push -u origin fix/issue-8
```

주의:

- 실수 복구 중 `git reset --hard`를 사용하지 않는다.
- `git branch -f main origin/main`은 현재 Branch가 새 Issue Branch인 상태에서만 실행한다.
- 사용자가 만든 미커밋 변경을 임의로 되돌리지 않는다.
- 관련 없는 미커밋 파일은 `git add` 대상에서 제외한다.

### 다른 Issue Branch에 잘못 커밋한 경우

다른 Issue Branch에 잘못 커밋했다면 새 Issue Branch를 `origin/main` 기준으로 만들고, 필요한 commit만 옮긴다.

```bash
git checkout -b fix/issue-새번호 origin/main
git cherry-pick <잘못-올린-commit>
```

그 뒤 기존 Branch에는 해당 commit이 남아 있으면 PR에 섞이지 않도록 정리한다.
정리가 애매하면 임의로 force push하지 말고, 현재 상태와 필요한 목표 상태를 먼저 공유한 뒤 진행한다.
