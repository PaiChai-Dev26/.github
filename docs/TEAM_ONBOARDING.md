# PaiChai Dev 26 첫날 안내서

대상: GitHub와 팀 개발이 처음인 팀원
예상 시간: 60~90분
목표: 저장소를 내려받고, 실행하고, 작은 변경을 PR로 제출한다.

## 1. 먼저 알아둘 말

- 저장소(Repository): 코드와 문서를 함께 보관하는 프로젝트 공간
- Issue: 해야 할 일과 문제를 기록하는 작업 카드
- Branch: `main`을 건드리지 않고 각자 작업하는 복사된 작업선
- Commit: 변경사항을 설명과 함께 저장한 한 단위
- Push: 내 컴퓨터의 커밋을 GitHub에 올리는 작업
- Pull Request(PR): 내 브랜치 변경을 팀원에게 검토·병합 요청하는 화면
- Review: 다른 팀원의 변경을 읽고 질문하거나 승인하는 과정

처음부터 모든 명령을 외울 필요는 없다. 아래 순서대로 실행하고, 오류가 나면 오류 메시지를 지우지
말고 팀 Issue에 남긴다.

## 2. 절대 하지 않을 것

1. `main` 브랜치에 직접 작업하거나 Push하지 않는다.
2. `.env`, 비밀번호, 토큰, 실제 학번·전화번호를 GitHub에 올리지 않는다.
3. 이해하지 못한 명령을 복사해 실행하지 않는다.
4. 다른 사람의 변경을 강제로 되돌리지 않는다.
5. 실제 고가품과 개인정보로 테스트하지 않는다.

`git reset --hard`, 강제 Push(`--force`), 대량 파일 삭제가 필요해 보이면 먼저 팀장에게 질문한다.

## 3. 내 운영체제에 맞게 최초 준비

팀장 Mac과 팀원 Windows에서 화면은 달라도 Git 명령은 같다. 이 문서에서:

- Mac은 기본 `터미널`을 사용한다.
- Windows 10/11은 **Git Bash**를 기본 터미널로 사용한다.
- 명령 앞의 `$` 또는 `PS>` 표시는 설명용이므로 직접 입력하지 않는다.

### 3.1 Mac

터미널에서 다음을 한 줄씩 실행한다.

```bash
xcode-select -p || xcode-select --install
brew install git gh uv
gh auth login
gh auth status
```

### 3.2 Windows 10/11

1. 시작 메뉴에서 `PowerShell`을 검색해 연다.
2. 다음 명령을 한 줄씩 실행한다. 설치 확인 창이 나오면 내용을 읽고 동의한다.

```powershell
winget install --id Git.Git -e
winget install --id GitHub.cli -e
winget install --id astral-sh.uv -e
```

3. 설치가 끝나면 PowerShell을 닫는다.
4. 시작 메뉴에서 `Git Bash`를 열고 다음을 실행한다.

```bash
git --version
gh --version
uv --version
gh auth login
gh auth status
```

`gh auth login`에서는 `GitHub.com` → `HTTPS` → 웹브라우저 로그인을 선택한다. 로그인 코드가 나오면
브라우저에 입력하고, 완료 후 Git Bash로 돌아온다.

`winget`을 찾을 수 없거나 학교 PC라 설치 권한이 없다면 임의의 설치 파일을 받지 말고 팀장에게 화면을
보낸다. 공식 설치 안내는 [Git for Windows](https://git-scm.com/download/win),
[GitHub CLI](https://cli.github.com/), [uv](https://docs.astral.sh/uv/getting-started/installation/)에서
확인한다.

Windows 작업 폴더는 OneDrive 동기화 폴더나 바탕화면보다 `C:\dev`처럼 짧은 영문 경로를 권장한다.
Git Bash에서는 해당 경로가 `/c/dev`로 보인다.

```bash
mkdir -p /c/dev
cd /c/dev
```

### 3.3 공통 Git 사용자 설정

GitHub 사용자 이름과 이메일도 확인한다.

```bash
git config --global user.name
git config --global user.email
```

값이 비어 있으면 본인의 GitHub 이름과 GitHub 비공개 이메일을 설정한다. 개인 이메일 공개 여부를 먼저
확인하고 설정한다.

## 4. 저장소 내려받기(Mac·Windows 공통)

팀장이 알려준 작업 폴더에서 실행한다.

```bash
gh repo clone PaiChai-Dev26/findproof
cd findproof
git status
```

`git status`에서 현재 브랜치와 변경 파일을 확인하는 습관을 들인다.

## 5. 프로젝트 실행하기

```bash
cp .env.example .env
uv sync --all-groups
uv run python scripts/seed.py
uv run fastapi dev
```

브라우저에서 다음 주소를 확인한다.

- 홈: http://127.0.0.1:8000
- API 문서: http://127.0.0.1:8000/docs
- 상태 확인: http://127.0.0.1:8000/health

서버를 종료할 때는 실행 중인 터미널에서 `Control + C`를 누른다.

## 6. 테스트 실행하기

Mac:

```bash
make check
```

Windows Git Bash에는 `make`가 기본 설치되지 않으므로 아래 세 명령을 사용한다.

```bash
uv run ruff check .
uv run mypy src
uv run pytest --cov=findproof --cov-report=term-missing
```

처음에는 테스트 내용을 모두 이해하지 못해도 된다. 명령이 통과하는지 확인하고, 실패하면 마지막 오류
부분을 복사해 Issue에 남긴다. Windows에서 `make: command not found`가 나오는 것은 프로젝트 오류가
아니므로 위의 Windows 검사 명령을 사용한다.

## 7. 첫 브랜치 만들기

항상 최신 `main`에서 시작한다.

```bash
git switch main
git pull --ff-only origin main
git switch -c docs/내깃허브아이디-first-contribution
```

예시:

```bash
git switch -c docs/seojin103-first-contribution
```

브랜치 이름에는 공백과 한글을 사용하지 않는다.

## 8. 첫 번째 연습 작업

`docs/evidence/` 아래에 본인의 첫 실행 기록을 만든다.

파일명 예시:

```text
docs/evidence/2026-07-22-seojin103-first-run.md
```

내용:

```markdown
# 첫 실행 기록

- GitHub ID:
- 실행한 날짜:
- `/health` 결과:
- `make check` 결과:
- 가장 헷갈렸던 점:
- 다음에 해볼 작업:
```

실제 토큰, 이메일, 로컬 전체 경로, 개인정보는 기록하지 않는다.

## 9. Commit과 Push

```bash
git status
git diff
git add docs/evidence/내파일명.md
git diff --cached
git commit -m "docs: add first run evidence"
git push -u origin 현재브랜치이름
```

`git add .`보다 올릴 파일을 직접 지정한다. Commit 전에는 `git diff --cached`로 GitHub에 올라갈 내용을
반드시 확인한다.

## 10. Pull Request 만들기

```bash
gh pr create --base main --web
```

PR 설명에는 다음을 적는다.

```text
변경한 이유:
변경한 내용:
확인한 방법:
막혔던 점 또는 남은 문제:
```

첫 PR은 기능의 크기가 아니라 전체 흐름을 한 번 경험하는 것이 목표다. 팀원 한 명이 내용을 읽은 후
병합한다.

## 11. 매일 작업 시작과 종료

시작할 때:

```bash
git status
git switch main
git pull --ff-only origin main
git switch 내작업브랜치
```

종료할 때:

```bash
git status
git diff
```

그다음 운영체제에 맞는 품질 검사 명령을 실행한다. Mac은 `make check`, Windows는 6장의 세 명령을
사용한다.

작업 중인 파일이 남아 있다면 무리하게 정리하지 말고 현재 상태와 다음 할 일을 Issue에 기록한다.

## 12. 질문하는 방법

막히면 아래 형식으로 공유한다.

```text
하려던 것:
실행한 명령:
발생한 오류:
시도한 방법:
현재 예상 원인:
```

오류 화면만 보내지 말고, 오류가 나오기 직전에 무엇을 했는지 함께 적는다. 비밀번호와 토큰은 가린다.

## 13. 첫 주 완료 기준

- 저장소 Clone 성공
- 로컬 서버 실행 성공
- 운영체제에 맞는 품질 검사 실행
- 개인 브랜치 생성
- 첫 실행 기록 Commit
- 첫 PR 생성
- 다른 팀원 PR에 질문 또는 승인 1회

첫 주에는 어려운 AI나 하드웨어 구현보다 이 과정을 세 명 모두 직접 완주하는 것이 우선이다.

첫 PR을 완료한 뒤에는 [`GITHUB_MANUAL.md`](GITHUB_MANUAL.md)에서 실제 팀 개발 흐름과 실수 복구 방법을
확인한다.
