---
title: 버전 관리 시스템
weight: 11
---
## 버전 관리 시스템
- 파일의 변화를 시간에 따라 기록한 후 과거 특정 시점의 버전을 다시 불러올 수 있는 시스템
- 컴퓨터의 모든 파일이 버전 관리 대상
- 대부분의 버전 관리 시스템은 개별 파일 혹은 프로젝트 전체를 이전 상태로 되돌리거나 시간에 따른 변경 사항을 검토할 수 있고 문제가 되는 부분을 누가 마지막으로 수정했는지 누가 이슈를 만들어 냈는지 등을 알 수 있는 기능을 제공

### 종류
- 로컬 버전 관리 시스템
  - 가장 기본적인 버전 관리는 파일을 다른 디렉터리에 복사하는 것
  - 직접 복사를 하게 되면 실수가 발생하기 쉽기 때문에 간단한 데이터베이스에 파일의 변경 사항을 기록하는 로컬 버전 관리 시스템을 사용
  - Mac OS X에서 X-code를 설치하면 RCS라는 프로그램이 자동적으로 설치되는데 개발 도중 발생하는 변경 사항을 특별한 형식의 파일에 저장하고 특정 시점의 파일 내용을 보고 싶을 때 해당 시점까지의 패치들을 모두 더해서 파일을 만들어나가는 방식을 사용
- 중앙 집중식 버전 관리 시스템(클라이언트 서버 모델)
  - 외부에 있는 개발자와의 협업 문제를 해결하기 위해서 개발되었는데 CVS, Subversion, Perforce와 같은 시스템이 여기에 속하는데 모든 파일을 저장하는 하나의 서버 와 이 중앙 서버에서 파일들을 가져오는 다수의 클라이언트가 존재하는 방식
  - 단일 장애점을 갖는 시스템
- 분산 버전 관리 시스템
  - Git이나 Mercurial이 대표적인데 클라이언트가 파일들의 마지막 스냅샷을 가져오지 않고 저장소를 통째로 복제해서 사용하는 방식으로 서버에 문제가 생겨도 클라이언트의 내용을 이용해서 서버의 데이터를 복구 할 수 있는 방식
  - 다수의 원격 저장소를 갖는 것도 가능

### 실례
- CVS: 클라이언트-서버 방식의 버전 관리 시스템
- Subversion: 분산 버전 관리 시스템
- Mercurial
  - 구글에서 관리를 지원하고 있는 분산 모델의 버전 관리 시스템
  - Git보다 많은 기능을 통합해서 제공
- Git
  - 분산 버전 관리 시스템
  - 리눅스 커널 개발을 다른 개발자와 공유하기 위해서 토발즈가 개발
  - 장점
    - 사용자가 많음: Tutorial이 많음
    - git을 이용해서 사용할 수 있는 원격 저장소가 많음

## Git 사용 환경 설정
### Git 설치
- linux: git 패키지를 다운로드 받아서 설치
- Mac이나 Linux: https://git-scm.com/downloads

### git을 GUI 환경에서 사용하기 위한 도구
- sourcetree: https://www.sourcetreeapp.com

## 소스 트리를 이용해서 Git Hub 저장소 연결하고 내용 일치
### 원격 저장소 생성
- git hub 페이지에서 저장소를 생성
- README.md 파일을 소유한 상태로 만들면 로컬에 동기화를 시키고 시작을 해야 하고 파일을 만들지 않으면 동기화하지 않아도 로컬에서 업로드 가능
- 저장소 URL: https://github.com/계정/저장소이름.git
  - 다운로드 받을 때는 저장소 URL을 전부 기재하지만 웹사이트에서 접속할 때는 .git을 생략

### 로컬 저장소 생성
- 생성 방법
  - Clone: 원격 저장소를 기반으로 생성
  - Create: 로컬 저장소에 디렉토리를 생성
  - Add: 디렉터리를 소스트리에 연결
- 소스 트리에서 클론
  - [파일] - [복제/생성]을 클릭
  - 복제할 원격 저장소 URL을 설정하고 로컬 컴퓨터의 디렉토리를 설정하고 [클론] 버튼을 누름
- git을 사용하는 디렉토리는 .git 이라는 디렉토리가 존재
  - 이 디렉토리는 숨김 속성을 가지고 있음
  - Mac 이나 Linux에서는 이름이 .으로 시작하면 숨김 속성

### 파일의 수정 내용 기록
- 내려받은 README.md 파일을 수정하고 소스 트리 의 [파일 상태]를 확인
- 수정한 내용을 확인하고 파일을 선택한 후 [모두 스테이지에 올리기] 또는 [선택 내용 스테이지에 올리기]를 선택
- 하단에 메시지를 작성하고 [커밋] 버튼을 누르면 변경 내용이 기록
- History 탭에 커밋한 내역이 추가

### 새로운 파일을 추가하고 기록
- 디렉터리에 새로운 파일을 추가하고 작성: test.py
```python
# test.py

def test():
    pass

if __name__=='__main__':
    test()
```
- 새로운 파일을 추가하게 되면 파일 상태에서 확인하면 '스테이지에 올라가지 않은 파일' 이라고 보이게 되고 파일 이름 앞에 ? 가 보이는데 이는 git 이 관리하지 않는 파일이라는 의미: add
- 변경 내용을 기록하는 방법은 이전과 동일: commit

### 로컬 저장소의 내용을 원격 저장소로 업로드: push
- push 아이콘을 누르면 원격 저장소에 업로드가 됨

### 로컬 저장소의 변경 내용을 로컬 저장소로 다운로드: pull
- 원격 저장소에서 파일을 생성하고 작성: remote_test.py
- 소스 트리에서 pull을 누르게 되면 변경 내용을 다운로드 받음

## Git 의 동작 개념
### 작업 영역
- Working Directory
  - 이력 관리 대상 파일들이 위치하는 영역
  - 저장소 디렉토리에서 .git 디렉토리를 제외한 공간
  - 작업 중인 파일이나 코드가 저장되는 공간
- Stage Area
  - 이력을 기록하는 공간
  - commit을 진행할 대상 파일들이 위치하는 영역
  - .git 디렉토리 하위에 파일 형태로 존재(index)
- Repository
  - commit 된 파일들이 위치하는 영역
  - .git 디렉토리에 이력 관리를 위한 모든 정보를 저장

### 작업
- 코드를 수정하는 것은 working directory의 내용을 수정
- 수정한 내역을 staging 영역에 전달하는 명령이 git add
- staging 영역의 내용을 repository로 전달하는 명령이 git commit

### git 이 관리하는 파일 상태
- modified: 변경은 감지되었지만 커밋이 되지 않은 상태

- staged: 감지된 파일의 수정 내용이 staging area로 이동한 상태

- commited: 파일의 수정 사항에 대한 이력 저장이 완료된 상태

- git 이 관리하는 파일들을 tracked 파일이라고 하고 그렇지 않은 파일들을 untracked 라고 합니다.

### Staging Area가 필요한 이유
- 일부 파일만 커밋하기 위해서
- 충돌을 수정하기 위해서
- 커밋을 수정하기 위해서

## Git 기본 명령어
### 저장소 연결: git init
- 현재 디렉토리가 git 의 관리 대상이 됨
- .git 디렉토리가 생성됨
- 모든 git에 대한 정보는 .git 디렉토리에 저장되므로 .git 디렉토리를 삭제하면 git 의 관리대상에서 해제
- .git 디렉토리를 다른 디렉토리로 복사하면 동일한 이력을 가지고 있는 저장소의 기능을 수행하게 됨

### 사용자 정보 설정
- 명령 형식
```
git config user.name 이름
git config user.email 이메일
```
- 내용 확인
```
git config --list

git config user.name
git config user.email
```
- 환경 설정 내용 삭제
```
git config --unset 속성
```
- 환경 설정을 할 때 --global을 추가하면 모든 git  디렉토리에 적용

### add 와 commit
- 파이썬에서 GUI 프로그램을 만들기 위한 패키지 설치
```
pip install PyQt5
```
- 윈도우 화면을 만드는 코드를 작성: main.py
- 파일을 생성하고 코드를 작성: main.py

- 현재 git의 상태 확인: git status

- 파일을 스테이징 영역으로 옮기기: git add main.py

- 이력 저장하기: git commit

- main.py 내용을 다시 수정하고 변경 내용 반영
```
git add main.py

git commit
```
- git commit 명령을 내릴 때 `-m` 옵션을 이용해서 메시지를 작성해서 바로 commit 할 수 있음, 메세지를 작성할 때 여러 줄이면 “ ”로 묶어서 작성
- add 와 commit을 한 번에 하고자 하는 경우에는 `-a` 옵션을 이용하면 되는데 이 경우 untracked 파일은 안됨
```git
git commit -am "한번에 커밋"
```
- 커밋 메시지 컨벤션
  - 커밋 메시지 규칙

### status, log, show
- 커밋의 내용을 확인하는 명령
- status는 상태 정보를 확인하는 명령
  - 옵션으로 `-s`가 있어서 간단하게 보기가 가능<br>
  파일 앞에 4가지 기호가 같이 출력되는데 `??`는 untracked 이고 `M`은 수정이 발생한 경우고 `MM`은 add 된 후 다시 수정이 발생한 경우고 `A`는 경로가 Staging 된 후 다시 파일이 생성된 경우
- git log
  - 저장소에 기록된 커밋 히스토리를 확인
  - 옵션으로 숫자를 설정할 수 있는데 숫자를 설정하면 숫자만큼의 히스토리만 출력
  - `p`옵션을 이용하면 상세히 보기 `git log -p -1`
  - `pretty=oneline` 옵션을 이용하면 커밋을 한 줄로 출력하는데 커밋이 많을 때 사용
  - oneline은 7개만 출력
  - graph 옵션을 이용하면 그래프 형태로 출력 `git log --graph`
- `git show`
  - 상세보기 명령으로 commit의 hash 값을 가지고 조회

### diff
- 파일의 수정 또는 변경 내용을 비교하는 명령
- 커밋 간 파일 내용 비교에 특화
- 비교할 대상 커밋들을 지정할 수 있음
- 옵션 없이 사용하면 현재 상태와 최신 커밋의 파일을 비교
- 코드를 수정 후 사용
  - main.py 파일을 수정
  - 내용을 수정하고 `git diff` 를 수행하면 수정한 내용을 확인할 수 있음
- `add`를 한 후 사용
  - `git add .`(.을 설정하면 현재 디렉터리의 모든 내용을 stage area에 추가)
  - `git diff`
  - 변경된 내용을 전부 staging area에 전송을 한 후 `git diff`를 하면 아무것도 출력되지 않는데 이유는 매개변수 없이 수행하면 unstaged 된 파일들의 수정 사항을 출력
- Staging Area의 파일과 최신 커밋의 비교는 `--stage`라는 옵션을 이용
  - `git diff --staged`
- commit 간의 비교
  - git diff 변경 전 커밋해시 변경후커밋해시

### reset
- Unstaging에 사용하는 명령
- `git add`를 취소하는 명령
  - 파일 2개를 추가(test1.txt, test2.txt)
  - staging area로 2개의 파일 전송
```
git add test1.txt test2.txt
git add .
git reset
옵션없이 사용하면 마지막에 staging 한 내용이 취소됩니다.
```
- 일부만 unstaging
```
git add .
git reset test2.txt
```

### amend
- 최근에 작성한 커밋 수정
- `git commit --amend` 형태로 사용
- 파일 1개만 commit 하고 amend를 이용해서 추가
```
git add test1.txt
git commit -m "add files"
git add test2.txt
git commit --amend
```
- 이렇게 가장 최근과 현재 내용이 합쳐져서 최근 commit을 삭제하고 새로운 commit을 생성

### checkout
- 해당 커밋 시점으로 되돌리는 명령
- `git checkout 커밋해시`
- 이전 상태로는 당연히 돌아갈 수 있고 이전 상태로 돌아갔다가 다시 최신 상태로 돌아올 수 있는데 이 경우는 커밋해시를 기억하거나 master 라는 이름을 이용하면 됨.<br>
첫번째 커밋을 HEAD라고 하고 마지막 커밋을 master 또는 main 이라고도 함
- 커밋 해시 대신에 파일명이나 .을 이용하면 파일이나 디렉토리의 전체를 이전 상태로 되돌릴 수 있음

### est1.txt 와 test2.txt 파일을 업로드해서 커밋을 한 상황에서 2개의 파일을 삭제해야 하는 상황
- 해결 방법
  - 저장소에서 파일을 제거하고 커밋을 추가: 로그가 남게됩니다.
  - 두 파일을 추가했던 커밋을 취소
- git reset의 옵션
```
        Working Directory       Staging Area        Repository
--hard  변경                    변경                변경
--mix
default 유지                    유지                유지
--soft  유지                    유지                변경
```
- reset 은 이전 커밋을 삭제하는 것이 아니고 마지막 커밋을 그 위치로 설정하는 명령

### reflog: HEAD의 참조 이력 확인
- 이력을 취소했는데 이전에 있던 이력으로 돌아갈려고 하는데 해시 값을 알지 못하는 경우 사용 가능
- `git reflog`
  - HEAD가 참조했던 역순으로 출력

### HEAD와 master
- branch
  - 저장소 내에 존재하는 독립적인 작업 관리 영역
  - `git init`으로 저장소를 생성하면 기본적으로 한 개의 브랜치가 제공되는데 이 때 기본 이름이 master
  - (HEAD -> master)은 HEAD가 master를 가리키고 있고 master가 어떤 커밋을 가리키고 있다라는 의미
  - HEAD는 master 처럼 커밋을 간접적으로 가리키기도 하고 커밋을 직접 가리키기도 함
  - HEAD는 Git에서 사용하는 공식 명칭이라서 변경이 불가능하지만 master는 사용자가 설정하는 이름이라서 변경이 가능
  - git checkout은 HEAD의 참조가 지정한 커밋으로 변경되는 것
  - HEAD와 master가 분리되는데 HEAD의 참조만 이동하기 때문으로 이런 HEAD를 Detached HEAD라고 함
  - git reset 명령은 브랜치의 참조가 변경되는 명령으로 브랜치는 최신 커밋을 참조

## Github 사용
### 1. 원격 저장소 생성
- git hub에 로그인 하고 Repositories에서 New 버튼을 누르고 옵션을 설정하면 생성

### 2. 원격 저장소 등록
- 명령: `git remote add [원격저장소이름] [url]`
  - 처음 연결하는 경우 원격 저장소이름을 origin으로 하는 것이 관계
- 확인
```
git remote

git remote -v

git remote show 원격저장소이름
```
### 3. 현재 상태에서 push를 하면 에러가 발생
```
git push --set-upstream origin master
```
- 원격 저장소에 master 브랜치가 없어서 에러가 발생
- 로컬의 브랜치를 main으로 변경하거나 원격 저장소에 master 브랜치를 만들어 주어야 함
- 로컬 저장소에 있는 브랜치가 원격 저장소에 없는 경우에는 `-u` 옵션을 이용하거나 `--set-upstream-to` 옵션을 이용
- `-u` 옵션 사용
  ```
  git push -u [원격저장소이름] [브랜치이름]
  git push -u origin master
  ```
  - 메시지를 확인하면 master 브랜치를 원격 저장소에 만들고 이 브랜치는 이제 원격 저장소의 master 브랜치를 추적한다고 함
  - 연결 확인
  ```
  git remote show origin
  ```
  - log를 확인하면 로컬 저장소 와 원격 저장소가 동기화 되었는지 확인하는 것이 가능<br>
  이 때 HEAD와 로컬 브랜치 그리고 원격 브랜치가 일치하면 동기화 된 상태
  ```
  git log --oneline
  ```
  - 한번 연결되면 이제부터 git push 만 해도 업로드됨
  - 커밋만 되고 push 되지 않아 동기화 되지 않은 상태, head와 origin/main이 다름
  ```bash
  $ git log --oneline
  4a1230e (HEAD -> main) test
  79cae72 (origin/main) add files
  cbe90d8 content update
  e8544f9 first
  ```
### 4. pull
- 원격 저장소의 변경 내역을 로컬 저장소에 반영하는 명령

### tag
- tag는 부가 정보
- git은 기본적으로 해시를 아이디로 사용하기 때문에 기억하기가 어려움
- 종류
  - Lightweight 태그: 태그만 설정
  - Annotated 태그: 태그 와 함께 여러 부가 정보를 같이 부여
- Lightweight 태그 붙이기
```
git tag [태그이름] [커밋해시]
git tag v1.0.0 e8544f9
```
- 태그를 이용해서 checkout 이 가능

### 푸시한 커밋 되돌리기
- 푸시한 커밋을 되돌리는 방법
  - 프로그램을 원래대로 복구하고 커밋과 푸시 추가하기
  - git revert로 푸시한 커밋 되돌리기
- git revert 사용
  - 파일 1개의 내용 수정
  - commit: `git commit -am "git revert 실습"`
  - 푸시: `git push`
- `git reset`은 브랜치의 참조를 특정 커밋으로 이동하는 명령이고 `git revert`의 경우는 특정 커밋을 취소하는 내용의 새로운 커밋을 만드는 것<br>
`git revert` 다음에는 취소할 커밋을 지정
- 가장 최근의 커밋을 revert
  - revert를 하고 log를 확인해보면 새로운 커밋이 만들어진 것을 확인할 수 있음
- git reset으로 이동한 경우는 push가 안됨
  - 로컬 저장소는 관계없는데 원격 저장소는 과거로 돌아가지 못하도록 되어 있음
  - `git push --force`로 강제로 푸시는 가능

## git hub 협업
### clone
- 원격 저장소의 내용을 가져오는 명령
  - `git clone 원격저장소URL [디렉터리이름]`
  - 디렉터리 이름을 생략하면 프로젝트 이름으로 생성
- 여러 개발자가 하나의 github 레포지토리를 공유하고자 하는 경우
  - git hub의 repository에서 Setting 를 클릭
  - collaborators 클릭