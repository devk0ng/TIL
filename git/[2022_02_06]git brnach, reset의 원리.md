# git branch, reset의 원리

# git branch의 원리

<br/>

## HEAD 파일

처음 git init 명령어를 실행할 경우 HEAD 파일이 생겨.

.git 디렉토리 내부에 생성된 것을 확인할 수 있어.

<img width="292" alt="스크린샷 2022-02-06 오후 11 40 27" src="https://user-images.githubusercontent.com/31160622/152688895-2a3ca8c0-6112-429d-983f-8c4272e45446.png">

해당 파일의 내용을 열어보면

<img width="290" alt="스크린샷 2022-02-06 오후 11 42 13" src="https://user-images.githubusercontent.com/31160622/152688910-e71111e5-33e1-4ce0-9ae1-aff5d77c17bb.png">

이렇게 ref:refs/heads/main 이라는 내용을 확인할 수 있어.

지금부터는 main branch에서 작업을 한다 생각하고 이후 내용을 써내려가보자.

<br/><br/>

## refs/head/main object

한번이라도 커밋을 할 경우 refs/heads/main 이라는 오브젝트가 생겨.

그리고 이 오브젝트 내용을 살펴보면 방금 커밋한 내용을 담고있는 개체에 대한 참조하는 값(id)가 들어있어!!

<img width="325" alt="스크린샷 2022-02-06 오후 11 48 01" src="https://user-images.githubusercontent.com/31160622/152688925-b31d8596-ce26-4c0c-9d10-9abf73fdfc70.png">

위의 e32~~~ 는 테스트를 위해 방금 커밋한 내용을 가리키는 id야!!

<img width="524" alt="스크린샷 2022-02-06 오후 11 50 14" src="https://user-images.githubusercontent.com/31160622/152688937-19d429e0-b594-481a-8024-34937a352cad.png">

해당 내용을 읽어보면 방금 커밋으로 인해 생성된 object를 가리키고 있음을 알 수 있지!!

<br/><br/>

## 정리

1. 결국 브랜치란 것은 refs 폴더 하위에 존재하는 파일이야!!
2. 브랜치를 만들 때 마다 refs/heads/ 하위에 해당 브렌치에 대한 파일이 생기게 되는거야
3. 그리고 HEAD 파일이 가리키는 (해당 파일에 기재된 브랜치 내용) refs/heads/ 의 특정 파일(브랜치)가 현재 작업 브랜치인거지!!
4. 결과적으로 refs/heads/ 안에 각각의 브렌치 파일들이 존재하고 그 브렌치 파일들은 단순히 특정 커밋(각 브랜치에서 최신)의 id를 가지고 있는 텍스트 파일인거야!! 
5. 브렌치를 옮길 때 마다(switch, checkout) HEAD 파일의 refs 대상을 변경할 뿐인 것이고 그 대상이란 것도 refs/heads/ 하위에 존재하는 브랜치 오브젝트일 뿐인거지.
6. 이렇게 함으로써 각각의 브렌치는 자신들이 가리키는 최신 commit 내용을 독립적으로 유지할 수 있는 것이고 최종적으로 독립적인 개발을 가능하게 해주는거야

<br/><br/><br/>

# git reset의 원리

<br/>

## git reset이란

```bash
git reset --hard commitId
```

위의 명령어는 해당 commitId 상태로 이동하게 해달라는 명령어이며 그 사이에 있던 커밋은 날려버리게 돼!

즉 reset을 한다는 것은 현재 브랜치가 가리키는 최신 커밋 상태를 대상 커밋(commitId에 해당하는)으로 바꾸는거야!!

위의 브랜치 내용과 연관지어본다면 refs/heads/branch_file 의 내용(해당 브랜치에서 가리키는 최신 커밋)이 수정되는거겠지!!

<br/><br/>

## git reset 되돌리기

그럼 git reset 명령어를 통해 날라간 커밋 내용들은 완전히 사라진 것일까?? nope!!! 🤗

git은 왠만하면 생성된 내용들을 지우거나 하지 않아!! 물론 gc가 돌아갈 상황이 생긴다하면 그럴 수 있겠지만 거의 남아있다고 생각하면 돼!! 즉 복구가 가능하단거야

어떻게 할 수 있을까??

1. logs/refs/heads/branch_file의 내용을 확인하면 해당 branch의 history 즉 사건들에 대한 기록을 볼 수 있어.
    <img width="1325" alt="스크린샷 2022-02-07 오전 12 22 46" src="https://user-images.githubusercontent.com/31160622/152688982-2c8369fa-355e-442b-8304-f6239e9a0572.png">

    위의 내용을 바탕으로 복구하고자하는 commit object의 id를 이용해 reset 명령어를 사용할 경우 복구할 수 있는거지
    
2. ORIG_HEAD 의 파일 내용을 바탕으로 복구할 수 이써!! 저기에는 뭐가 들어있냐??
    
    reset같이 뭔가 날릴 수 있는 조금은 위험하다 생각할 수 있는 명령어들에 대해서는 해당 명령어를 실행전에 현재 HEAD가 가리키는 커밋에 대해 백업작업을 진행 후 명령어를 실행해.
    
    그리고 그 백업 내용이 ORIG_HEAD에 들어있는거지
    
    결국 아래 명령어를 통해 복구 가능하단거야!!
    
    ```bash
    git reset --hard ORIG_HEAD
    ```
    
3. git reflog 명령어를 통해 현재까지 진행한 명령어들의 히스토리를 볼 수 있는데 이를 활용할 수도 있어.

<br/><br/>

## git reset vs git checkout

```bash
git checkout commitId
```

위처럼 사용할 수도 있어.

저렇게 할 경우 HEAD가 해당 커밋(commitId)를 가리키게 되는거지!!

하지만 이렇게 할 경우 일반적으로 HEAD파일은 위에서도 이야기했듯이 refs/heads/branch_file 즉 특정 브랜치를 가리키는 내용을 담고있었는데 위의 명령어를 진행할 경우 브랜치가 아닌 커밋 자체를 가르키게 되버려!!

즉 브랜치는 아니란거지!!

이는 커밋을 직접 가리키는 상태이고 detached되어있는 상태라고 이야기해!!

<br/><br/>

## reset의 option 들

| working directory | staging area | repository |
| --- | --- | --- |
|  |  | git reset —soft |
|  | git reset —mixed | git reset —mixed |
| git reset —hard | git reset —hard | git reset —hard |

위의 표처럼 —hard의 경우 reset 대상이 되는 커밋 내용들에 대해 working directory, staging area, local repository 모두에 반영이 되어져!!

—soft의 경우에는 repository의 내용에 대해서만 reset 대상이 되는 커밋으로 반영해주는 거구

mixed는 —soft에서 staging area까지 추가반영!!

보통 복구라는게 working directory, staging area, repository 모두에 대해 진행할 것 같아서 아마 —hard 옵션을 가장 많이 사용하지 않을까 생각하는데 혹시 다른 옵션들 사용하게 될 일이 있으면 한번 더 공부해보고 슥 해보자!
