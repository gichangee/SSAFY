# Git

ssh

개인 pc에 ssh 키 만들고 github에다가 등록하면

해당 pc가 github에 접근할 수 있는 권한을 주는 것 

git reset —hard 해쉬코드

내가 브랜치 따온 시점에서 commit 까지 한 상황 근데 여기서 

누군가 master에 push를 한 상황

근데 master코드와 내 branch 코드가 겹치는 상황

이때 rebase를 사용한다

내가 만든 branch에서 git rebase master

git status를 눌러서 어디가 수정되었는지 확인

git add .

git rebase —continue

그 다음 master에서 합병하면 깔끔

git merge —no-ff —log 브랜치명