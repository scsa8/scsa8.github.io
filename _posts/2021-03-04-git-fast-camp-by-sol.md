---
layout: post
title: "깃(Git) 쪽집게 정리"
description: "현업에서 쓰는 깃 관련 개념과 커맨드만 속성으로 알아본다."
categories: 
  - Dev Tool
tags: ["Dev. Sol", "GIT", "형상관리"]
author: "Dev. Sol"
---

## 깃(Git)이란

컴퓨터 파일의 변경사항을 추적하고 여러 명의 사용자들 간에 해당 파일들의 작업을 조율하기 위한 분산 버전 관리 시스템. 그냥 형상관리 툴 중 하나이며 가장 많이 쓰이는 툴이다.

간혹 입문자들이 github과 git을 동일시하기도 하는데 둘은 엄연히 다르다. github은 원격 저장소를 제공해주며 Web UI를 통해 편리하게 깃의 다양한 기능을 이용하도록 도와주는 웹이다.

이번 포스트는 깃 입문자들을 위하여 현업에서 쓰이는 기능들을 통해 속성으로 개념과 커맨드를 한번에 정리해보려한다. 물론 소스트리, 톨토이즈와 같은 UI 기반 깃 클라이언트나 IDE에서 제공하는 Git 플러그인을 사용하면 더 쉬울 수 있지만... 개발자에게 간지란 커맨드라고 생각한다.ㅎㅎ 그리고 이것만 알면 웬만한 깃을 통한 협업은 다 할 수 있다고 확신한다.

## 기본 개념
![그림1-1](/assets/images/sol/20210304/git_diagram.jpg "그림 1-1")

위 도식과 함께 용어 및 개념을 정리해보자.
- 로컬(Local) : 작업 중인 폴더
- 원격 저장소(Remote Repository) : 네트워크 상의 다른 위치에 존재하는 깃 저장소 ex) Github, Gitlab
- Staging Area : 어떤 변경사항이 저장소에 커밋되기 전에, 반드시 거쳐야만 하는 중간단계. 즉 커밋할 목록들을 저장해 놓은 곳. 변경된 사항이라도 Staging Area에 존재하지 않으면 커밋되지 않는다. 여기서부터 깃으로 관리된다고 생각하면 됨.
- Local Repository : Staging Area에서 커밋된 변경사항들이 저장되는 곳. 즉, 로컬에서의 모든 변경 작업이 완료된 상태.
- git clone : 원격 저장소로부터 모든 파일들을 형상 그대로 로컬 저장소로 다운로드 하는 명령어
- git add : 변경된 사항을 Staging Area로 이동시키는 명령어
- git commit : Staging Area의 내용들을 Local Repository로 이동시키는 명령어
- git pull : 원격 저장소의 변경된 내용들을 Local Repository로 다운로드하는 명령어
- git push : Local Repository의 변경된 내용들을 원격 저장소로 업로드하는 명령어
- git fetch : pull과는 다르게 원격 저장소의 변경된 내용들을 Local Repository로 다운로드하지 않고 변경 로그(log)들을 최신화하는 명령어
- 브랜치(branch) : 브랜치란 독립적으로 어떤 작업을 진행하기 위한 개념. 필요에 의해 만들어지는 각각의 브랜치는 다른 브랜치의 영향을 받지 않기 때문에, 여러 작업을 동시에 진행할 수 있음.

위 개념들만 숙지한다면 깃을 통한 협업을 시작할 수 있다.


## 깃 활용
개념들을 숙지했다면 실습을 통해 부딪치며 이해해보자.

### 깃 설치
깃 <a href="https://git-scm.com/download">공식 사이트</a>에서 환경에 맞게 깃을 설치한다.

### 클론
 **git clone \<원격 저장소 url\>** 커맨드를 통해 원격 Repository의 소스를 다운로드한다. <br>
실습용으로 https://github.com/scsa8/GitTestRepository.git 을 이용해보자. <br>
원하는 디렉토리로 가서 아래 커맨드를 실행한다. <br>
Git Bash를 이용해도 되고 CMD를 이용해도 된다.
```sh
$ git clone https://github.com/scsa8/GitTestRepository.git
```
소스가 내려받아진다면 성공

### 브랜치 생성
자, main 브랜치를 다이렉트로 변경하게 되면 위험할 수 있으니 나만의 작업브랜치를 새로 생성해보자.<br>
 **git checkout -b \<브랜치 이름\>** 커맨드를 사용하면 브랜치를 생성함과 동시에 해당 브랜치로 checkout(브랜치 이동) 할 수 있다.
```sh
$ git checkout -b your-branch
```

### 변경사항 Staging
파일을 새로 생성하거나 test.txt를 변경한다.
현재 깃의 상태(현재 브랜치, 변경사항, Staging된 항목, commit 등)을 알아보려면 아래 커맨드를 실행한다.
```sh
$ git status

On branch your-branch

Changes not staged for commit:
  (use "git add <file>..." to update what will be committed)
  (use "git checkout -- <file>..." to discard changes in working directory)

        modified:   test.txt

no changes added to commit (use "git add" and/or "git commit -a")

```

위와 같이 test.txt가 stage되지 않았다고 뜬다.<br>
해당 변경 사항을 커밋하기 위해 먼저 **git add <파일명>**을 이용해 stage 해보자.
```sh
$ git add text.txt
```

상태 확인 커맨드를 다시 실행해보면 아래와 같이 나온다.
```sh
$ git status

On branch your-branch
Changes to be committed:
  (use "git reset HEAD <file>..." to unstage)

        modified:   test.txt

```

파일을 일일히 입력하는게 귀찮으면 **git add --all** 명령어를 입력한다. 모든 변경된 사항들(.gitignore에 명시된 파일들 제외, 이건 따로 알아보라)을 stage해준다.

### 커밋
이제 원격 저장소에 업로드 전 최종적으로 Local Repository에 저장할 것이다.<br>
**git commit -m \<커밋 메시지\>** 커맨드를 실행하여 커밋을 한다. 되도록이면 커밋 메시지는 명확하게 적어주는 것이 좋다.
```sh
$ git commit -m "text.txt 파일 편집"
```

### 푸시
다됐다. 이제 내 작업을 원격 저장소로 업로드하는 일만 남았다.
업로드를 위해 **git push**를 날려본다.
```sh
$ git push
fatal: The current branch your-branch has no upstream branch.
To push the current branch and set the remote as upstream, use

    git push --set-upstream origin your-branch

```
뭔가 문제가 있어보인다. 이유는 원격 저장소에 해당 브랜치가 없기 때문이다. 다행히 친절하게 git push --set-upstream origin your-branch를 사용하라고 나온다. <br>
여기서 알아둬야 할 점은 브랜치 앞에 origin이 붙으면 원격 저장소의 브랜치를 뜻한다. <br>
그리고 --set-upstream은 현재 로컬브랜치 your-branch가 따르는 원격 브랜치를 해당 브랜치로 설정하겠다는 의미이다.<br>
즉 처음에 한번만 해당 커맨드를 실행하면 그 다음부터는 **git push** 커맨드만 입력해도 알아서 origin your-branch로 푸시하겠다는 의미이다. 해당 옵션이 빠지면 매번 뒤에 브랜치명을 붙여줘야한다.
물론, --set-upstream을 쓰기 귀찮으니 더 간단하게 **git push -u origin \<브랜치명\>**을 사용하면 된다.<br>
***참고로 원격 브랜치명은 로컬 브랜치명과 같아야한다.***
아래 커맨드를 실행해보자.
```sh
$ git push -u origin your-branch
```
아마도 다른 문제가 없다면 성공적으로 성공했다는 문구가 뜰 것이다.

### main 브랜치와 머지
내 작업 브랜치의 내용을 main 브랜치와 머지할 차례다. <br>
깃 커맨드로 사용할 수도 있지만 보다 질 높은 머지를 위해 github을 이용할 것이다. 차근차근 해보자.
1. github 리파지토리로 접근한다.(https://github.com/scsa8/GitTestRepository)
2. Pull Request Tab에 들어간다.
3. New Pull Request 버튼을 클릭한다.
![그림1-2](/assets/images/sol/20210304/pull-request.PNG "그림 1-2")
4. 아래 그림과 같이 compare -> base로 머지하도록 설정한다.
![그림1-2](/assets/images/sol/20210304/pull-request2.PNG "그림 1-2")
5. Create Pull Request 버튼을 누르고 이상이 없다면 다음 화면에서 한번 더 누른다.
6. 그 다음에는 리뷰어에게 리뷰를 요청할 수도 있고 최종 확인이 되었다면 Merge Pull Request -> Confirm Merge 버튼을 클릭한다.
7. 메인 브랜치로의 머지가 완료되었다.

### 내 작업 브랜치로 새로운 변경사항 pull 하기
나 외에 다른 개발자가 main 브랜치에 새로운 변경사항을 적용했다고 가정하자.<br>
현재 작업 브랜치와 메인 브랜치를 비교하면 어떤가?<br>
물론 메인 브랜치로부터 새로운 작업 브랜치를 또 생성하여 작업할 수도 있겠지만, 기존 your-branch 브랜치에서 계속 작업하고 싶을 수도 있다.<br>
이럴때 **git pull origin \<브랜치명\>**을 사용한다. 메인 브랜치의 변경사항을 당겨오고 싶기 때문에 아래와 같이 커맨드를 실행해보자.
```sh
$ git pull origin main
```
새로운 변경사항이 적용되는 것을 알 수 있다.

필자는 개인적으로 **git pull -r origin main**을 사용한다. -r(rebase)이란 옵션이 들어가는 것은 한번 공부해볼 것!

### 그 외 유용한 명령어
```sh
$ git log
```
현재 Local Repository에 저장된 모든 커밋 내역들을 보여준다.

```sh
$ git fetch
```
위에 설명했듯이 소스는 다운받지 않고 원격 저장소 정보만 최신화한다. 설명이 좀 부족한 것 같은데 쓰다보면 알게 된다.

```sh
$ git stash
```
staging된 변경사항들을 임시로 Staging Area에서 제거하는 커맨드

```sh
$ git stash pop
```
stash된 변경사항들 중 가장 최근에 stash된 변경사항을 Staging Area로 가져오는 커맨드

### 마치며
사실 터미널로 커맨드를 사용하기보다 IDE나 다른 툴(툴은 소스트리 추천)을 통해 깔끔하게 UI로 작업하는 것이 더 쉬울 수 있다.
하지만 때로는 그러한 툴을 사용하기보다 커맨드를 이용한 방법이 더 편할때가 있다. 필자는 주로 터미널을 통해 커맨드를 사용하는 것을 선호한다.
이외에 더 많은 기능들이 있지만 위에 언급된 기능들만으로도 모든 협업이 가능하다.
물론 이번 포스팅에 언급되지 않은 git 설정 관련 내용들은 반드시 더 공부가 필요하다. 시간이 난다면 해당 내용도 포스팅해보겠다. <br>
git 입문을 환영한다...<br>
질문도 언제나 환영이다.










