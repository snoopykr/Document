# 원격 저장소에 올라간 커밋 되돌리기

로컬에 저장된 커밋인 경우에는 git reset 명령을 이룡해서 쉽게 커밋을 되돌릴 수 있지만 원격 저장소에 올라간 경우라면 이야기가 달라진다.

- reset : 기존 commit을 제거한다.
- revert : 기존 commit에 새로운 commit을 추가한다.

## reset 사용
```bash
// 3 단계 롤백...
$ git reset --hard HEAD~3

// 에러 발생...
$ git push origin master

// -f 또는 --force 옵션 추가...
$ git push -f origin master
```

## revert 사용
```bash
// revert는 사용할때 마다 커밋이 발생하게 되기 때문에 --no-commit 옵션을 사용해서 커밋이 발생되지 않게 해야 한다.

// 3 단계 롤백...
$ git revert --no-commit HEAD~3

// 롤백 내용 commit
$ git commit -m 'Revert "Commit C, B, A"'

// 원격 저장소에 저장.
$ git push origin master
```