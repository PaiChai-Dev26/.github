# PaiChai Dev 26 GitHub 팀 개발 매뉴얼

대상: Git과 GitHub를 처음 사용하는 팀원

이 문서는 첫 PR 이후 실제 프로젝트를 진행할 때 참고하는 운영 매뉴얼이다. 처음 참여하는 팀원은 먼저
`TEAM_ONBOARDING.md`의 첫날 실습을 완료한다.

## 1. Git과 GitHub의 차이

- Git: 내 컴퓨터에서 파일 변경 이력을 관리하는 도구
- GitHub: Git 저장소를 온라인에서 공유하고 Issue·PR·Review·CI로 협업하는 서비스
- 로컬: 내 컴퓨터(Mac 또는 Windows)의 저장소
- 원격(`origin`): GitHub의 `PaiChai-Dev26/findproof` 저장소

```text
내 컴퓨터에서 수정
    ↓ git add
Commit 준비 영역
    ↓ git commit
내 브랜치의 로컬 이력
    ↓ git push
GitHub 원격 브랜치
    ↓ Pull Request + Review + CI
main에 병합
```

## 2. 이 프로젝트의 기본 규칙

1. 작업은 Issue에서 시작한다.
2. 최신 `main`에서 작업 브랜치를 만든다.
3. 한 브랜치는 한 가지 목적만 가진다.
4. `main`에 직접 Push하지 않는다.
5. PR에는 변경 이유와 확인 방법을 적는다.
6. CI가 실패한 PR은 병합하지 않는다.
7. 다른 팀원 한 명이 읽고 난 뒤 병합하는 것을 원칙으로 한다.
8. `.env`, 토큰, 실제 개인정보, 로컬 DB를 Commit하지 않는다.

## 3. 작업 전체 흐름

### 3.1 Issue 선택

Issue에서 다음 내용을 확인한다.

- 해결해야 하는 문제
- 완료 조건
- 담당자
- 난이도
- 관련 화면·API·문서

완료 조건이 이해되지 않으면 코드를 작성하기 전에 질문한다.

### 3.2 최신 main 받기

작업 파일을 수정하기 전에 실행한다.

```bash
git status
git switch main
git pull --ff-only origin main
```

`git status`에 수정 파일이 있다면 무시하고 브랜치를 바꾸지 않는다. 먼저 작업 내용을 Commit할지,
임시로 보관할지 팀장과 결정한다.

### 3.3 브랜치 만들기

```bash
git switch -c 종류/짧은-작업명
```

접두사:

| 접두사 | 사용 상황 | 예시 |
|---|---|---|
| `feat/` | 사용자 기능 | `feat/claim-form` |
| `fix/` | 오류·보안 수정 | `fix/token-reuse` |
| `test/` | 테스트 보강 | `test/matching-edge-cases` |
| `docs/` | 문서 | `docs/interview-result` |
| `research/` | 조사·실험 | `research/ocr-spike` |
| `firmware/` | ESP32 펌웨어 | `firmware/door-sensor` |

브랜치 이름은 영문 소문자, 숫자, 하이픈을 사용한다.

### 3.4 변경 확인

작업 중 자주 실행한다.

```bash
git status
git diff
```

- `git status`: 어떤 파일이 바뀌었는지 확인
- `git diff`: 파일 내용이 어떻게 바뀌었는지 확인

### 3.5 테스트

Mac:

```bash
make check
```

Windows Git Bash:

```bash
uv run ruff check .
uv run mypy src
uv run pytest --cov=findproof --cov-report=term-missing
```

Windows에는 `make`가 기본으로 없으므로 별도로 설치하지 않고 위 세 명령을 사용한다. 세 명령이 모두
통과해야 `make check`와 같은 결과다.

실패했다면 PR 설명에 숨기지 말고 원인과 남은 문제를 적는다.

### 3.6 Commit

```bash
git add 정확한/파일/경로
git diff --cached
git commit -m "종류: 변경 내용"
```

Commit 예시:

```text
feat: add claim request form
fix: hide private verification details
test: cover expired return token
docs: record dormitory interview
firmware: report door sensor state
```

Commit에는 완성된 작은 목적 하나만 넣는다. 기능 코드와 관계없는 대량 포맷 변경을 섞지 않는다.

### 3.7 Push와 PR

첫 Push:

```bash
git push -u origin 현재브랜치
```

PR 생성:

```bash
gh pr create --base main --web
```

PR 설명:

```markdown
## 변경 이유

## 변경 내용

## 확인 방법
- [ ] 운영체제에 맞는 품질 검사
- [ ] 직접 확인한 화면 또는 API

## 남은 위험

Closes #이슈번호
```

## 4. Commit, Push, Merge의 차이

- Commit: 내 브랜치에 변경 이력을 저장
- Push: 로컬 Commit을 GitHub 브랜치로 전송
- Merge: PR의 변경을 `main`에 합침

Push했다고 서비스의 `main`이 바뀌는 것은 아니다. PR이 병합되어야 팀 공용 기준에 포함된다.

## 5. 병합 커밋이란

병합 결과도 Git 이력에 하나의 Commit으로 기록된다. GitHub PR에서 병합 방법에 따라 모양이 달라진다.

### Squash and merge

PR의 여러 Commit을 하나로 합쳐 `main`에 넣는다.

```text
작업 브랜치: A → B → C
main 병합:   S
```

- 장점: `main` 이력이 간결함
- 단점: 작업 브랜치 Commit SHA와 `main` SHA가 달라짐
- FindProof 기본 권장 방식

예: PR #8의 여러 변경을 합친 `306d10a`가 `main`의 병합 결과다.

### Merge commit

작업 브랜치의 모든 Commit을 유지하고 별도 병합 Commit을 만든다.

- 장점: 작업 이력을 그대로 유지
- 단점: 작은 PR이 많은 팀에서는 그래프가 복잡해짐

### Rebase and merge

작업 Commit을 `main` 끝에 일렬로 다시 배치한다.

- 장점: 선형 이력
- 단점: Commit SHA가 바뀌고 초보자가 이해하기 어려움

팀이 익숙해지기 전에는 Squash and merge를 사용한다.

## 6. PR 리뷰 방법

리뷰는 사람을 평가하는 것이 아니라 변경 내용을 함께 확인하는 과정이다.

### 작성자가 확인할 것

- PR 제목이 실제 변경을 설명하는가?
- 관련 없는 파일이 포함되지 않았는가?
- 비밀정보와 실제 개인정보가 없는가?
- 테스트가 통과하는가?
- 화면·API 결과를 직접 확인했는가?
- 미완성 기능을 완성된 것처럼 쓰지 않았는가?

### 리뷰어가 확인할 것

1. PR 설명을 읽는다.
2. `Files changed`에서 변경 파일을 확인한다.
3. 이해되지 않는 부분은 질문한다.
4. 수정이 필요하면 구체적인 이유를 적는다.
5. CI를 확인한다.
6. 문제가 없으면 승인한다.

리뷰 문장 예시:

```text
이 조건에서 비공개 특징도 응답에 포함될 가능성이 있는지 궁금합니다.
이 함수가 실패할 때의 테스트를 한 건 추가하면 좋겠습니다.
직접 실행했고 정상 동작을 확인했습니다.
```

## 7. CI 읽는 방법

CI는 GitHub가 새 Commit을 받아 동일한 품질 검사를 자동 실행하는 과정이다.

FindProof 검사:

- Ruff lint
- Ruff format check
- mypy
- pytest

PR의 `Checks` 또는 실패한 작업의 `Details`에서 로그를 확인한다. 가장 아래의 실제 오류부터 읽는다.

CI가 실패하면 새 PR을 만들지 않고 같은 브랜치에서 수정한다.

```bash
# 수정 후
# Mac: make check
# Windows: 3.5장의 uv run 명령 3개
git add 수정한파일
git commit -m "fix: resolve CI failure"
git push
```

기존 PR이 자동으로 업데이트된다.

## 8. main 병합 후 다음 작업

```bash
git switch main
git pull --ff-only origin main
```

작업 브랜치는 기록을 확인한 뒤 삭제할 수 있다.

```bash
git branch -d 이전브랜치
git push origin --delete 이전브랜치
```

브랜치 삭제는 PR과 Commit 기록을 삭제하지 않는다. 병합되지 않은 브랜치를 삭제하려면 먼저 팀장에게
확인한다.

## 9. 자주 생기는 상황

### 수정 파일을 Commit하지 않고 브랜치를 바꾸려 할 때

무리하게 강제 전환하지 않는다.

```bash
git status
git diff
```

작업이 유효하면 현재 브랜치에 Commit한다. 버려야 할 변경이라면 팀장 확인 후 처리한다.

### Push가 거부될 때

먼저 원격 상태를 확인한다.

```bash
git status
git branch -vv
git fetch origin
```

강제 Push하지 말고 오류 전체를 공유한다.

### main이 오래되었을 때

개인 작업이 없다면:

```bash
git switch main
git pull --ff-only origin main
```

작업 브랜치가 오래되어 충돌이 예상되면 팀장과 함께 최신 `main`을 반영한다.

### 잘못된 파일을 Stage했을 때

Commit 전이라면:

```bash
git restore --staged 파일경로
```

파일 수정 내용은 유지되고 Stage에서만 빠진다.

### Commit 메시지를 잘못 썼을 때

아직 Push하지 않은 가장 최근 Commit이라면:

```bash
git commit --amend -m "올바른 메시지"
```

이미 Push했다면 혼자 이력을 바꾸지 말고 팀장에게 알린다.

### 충돌이 발생했을 때

1. `git status`로 충돌 파일을 확인한다.
2. `<<<<<<<`, `=======`, `>>>>>>>` 표시를 찾는다.
3. 어떤 내용을 유지할지 작성자와 확인한다.
4. 표시를 모두 제거하고 테스트한다.
5. 혼자 판단하기 어렵다면 중단하고 도움을 요청한다.

처음 두 프로젝트에서는 복잡한 충돌을 팀장과 화면을 공유하며 해결한다.

## 10. 하면 안 되는 위험 명령

팀장 확인 없이 실행하지 않는다.

```text
git push --force
git reset --hard
git clean -fd
git branch -D
```

인터넷에서 본 명령에 위 옵션이 포함되어 있으면 실행 전에 질문한다.

## 11. 비밀정보와 개인정보

Commit 금지:

- `.env`
- GitHub 토큰·API 키·비밀번호
- 실제 학생 학번·전화번호·호실
- 실제 학생증·신분증 이미지
- 운영 DB 파일
- 모델·데이터의 사용 권한이 불명확한 파일

실수로 비밀정보를 Push했다면 파일만 삭제하고 끝내지 않는다. 즉시 팀장에게 알리고 해당 토큰을
폐기·재발급한다. Git 이력에는 이전 내용이 남을 수 있다.

## 12. GitHub Issue 작성법

좋은 Issue는 다른 사람이 완료 여부를 판단할 수 있다.

```markdown
## 문제

## 할 일
- [ ]
- [ ]

## 완료 조건
- [ ] 테스트
- [ ] 화면/API 확인
- [ ] 문서 기록

## 참고

## 난이도
L1 / L2 / L3 / Spike
```

작업이 예상보다 커지면 Issue를 나눈다. 완료 조건이 바뀌면 댓글로 이유를 남긴다.

## 13. 우리 팀의 난이도 기준

- L1: 문서·데이터·HTML 수정, 약 2시간
- L2: 기존 코드를 참고한 함수·API·테스트, 약 하루
- L3: 새로운 상태 전이·인증·장치 통합, 2일 이상
- Spike: 결과를 보장하지 않는 조사·실험

처음 두 주에는 L1·L2 작업을 맡고, L3는 두 명 이상이 함께 진행한다.

## 14. 자주 쓰는 명령

```bash
# 상태와 변경 확인
git status
git diff
git diff --cached

# 원격 최신 정보 받기
git fetch origin

# 브랜치 확인·생성·전환
git branch -vv
git switch main
git switch -c feat/example

# Commit과 Push
git add 파일경로
git commit -m "feat: explain change"
git push -u origin feat/example

# PR
gh pr create --base main --web
gh pr status

# 프로젝트 검사
make check
```

위 명령은 Mac 기준이다. Windows는 3.5장의 `uv run` 세 명령을 사용한다. 그 외 Git·GitHub 명령은
Windows Git Bash에서도 동일하다.

## 15. Windows에서 자주 생기는 상황

### 명령을 찾을 수 없을 때

Git·GitHub CLI·uv 설치 직후 열려 있던 터미널에는 새 경로가 반영되지 않을 수 있다. Git Bash를 완전히
닫고 다시 연 뒤 버전을 확인한다.

```bash
git --version
gh --version
uv --version
```

### PowerShell과 Git Bash 명령이 섞였을 때

설치는 PowerShell의 `winget`으로 하고, 프로젝트 작업은 Git Bash로 통일한다. `cp`, `mkdir -p` 같은
명령은 Git Bash에서 실행한다.

### 경로 또는 동기화 문제가 생길 때

한글·공백이 많은 경로나 OneDrive 동기화 폴더에서는 도구별 경로 문제가 생길 수 있다. 저장소를
`C:\dev\findproof`처럼 짧은 영문 경로에 다시 Clone한다. 기존 폴더를 바로 삭제하지 말고 Commit하지
않은 변경이 없는지 먼저 확인한다.

### 줄바꿈이 파일 전체 변경으로 보일 때

코드를 수정하지 않았는데 파일 전체가 바뀐 것으로 보이면 Commit하지 않는다. 아래 결과를 팀장에게
보낸다.

```bash
git status
git diff --stat
git diff
```

에디터의 줄바꿈 설정을 혼자 일괄 변경하지 않는다.

## 16. 도움 요청 체크리스트

질문할 때 공유한다.

```text
현재 브랜치:
하려던 작업:
실행한 명령:
오류 전문:
시도한 방법:
git status 결과:
```

토큰·비밀번호·개인정보는 가리고 공유한다. 오류를 숨기는 것보다 빠르게 공유하는 것이 좋은 팀 개발이다.

## 17. 팀원이 반드시 직접 해볼 것

- Issue 하나 읽고 질문하기
- 브랜치 직접 만들기
- 작은 Commit 만들기
- Push와 PR 생성하기
- 다른 팀원의 PR 리뷰하기
- CI 실패 로그 한 번 읽기
- 병합 후 최신 `main` 받기

명령을 외우는 것이 목표가 아니다. 변경을 작게 만들고, 확인하고, 다른 사람이 이해할 수 있게 기록하는
습관을 만드는 것이 목표다.
