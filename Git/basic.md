# git 저장소

## git 저장소 초기화 및 복제
```bash
// 초기화 ('.git' 폴더 생성)
// git init 폴더
$ git init
$ git init .

// 복제
// git clone 원격저장소URL 새폴더이름
$ git clone https://github.com/snoopykr/Hancom
```

## 워킹 디렉토리(워킹트리), 스테이지, 레파지토리

- 워킹 디렉토리 : 작업을 하는 공간
- 스테이지 : 임시로 저장하는 공간
- 레파지토리 : 실제로 저장하여 기록하는 공간

```bash
// 파일('test.go') 생성...
$ vi test.go

// tracked 상태로 전환...
$ git add test.go


// (참조) untracked 상태로 전환...
$ git rm --cached test.go
// (참조) 한번이라도 커밋을 했다면 reset
$ git reset HEAD test.go


// 현재 git의 상태를 출력... (modified 상태 확인)
$ git status
On branch master
Your branch is up to date with 'origin/master'.

Changes not staged for commit:
  (use "git add <file>..." to update what will be committed)
  (use "git checkout -- <file>..." to discard changes in working directory)

	modified:   Git/basic.md

no changes added to commit (use "git add" and/or "git commit -a")

// 스테이지에 저장되어 있는 즉 tracked된 파일 목록
$ git ls-files --stage
```

## 파일 이름 변경
```bash
// 등록된 파일 이름이 변경된 경우
// git mv 파일이름 새파일이름
$ git mv test.go testing.go

// 단계적 실행...
$ mv test.go testing.go
$ git rm test.go
$ git add testing.go
```

## git commit
```bash
// commit 도움말...
$ git commit --help
$ git commit -h

// 외부 에디터 사용을 위한 한경 설정
$ git config --global core.editor "에디터경로"

// 파일 등록과 커밋을 동시에 진행
$ git commit -a
```