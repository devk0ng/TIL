# git pull, fetch, tag의 원리

# git remote repository (원격저장소)

<br/><br/>

## 원격저장소란?

원격저장소는 현재 로컬 레파지토리에서 연결할 대상이라고 볼 수 있어.

연결을 했다는 것은 git 명령어(push, pull, fetch)를 통해 연결된 외부 레파지토리에 현재 작업한 로컬 레파지토리의 내용을 반영하거나 혹은 외부 레파지토리의 내용을 가져올 수 있게 되는 것을 뜻해.

이 원격저장소에 대해 전문적으로 서비스 해주는 녀석이 바로 Git hub야.

자신이 직접 저장소를 만들어서 관리할 수도 있어!! 

```bash
git init —bare
```

위 명령어를 사용하면 돼!! 위 명령어는 working directory 즉 작업공간을 만드는 것이 아닌 저장소만을 만들어줘!!

<br/><br/>

## 원격 저장소의 원리

local repository 와 remote repository를 연결하는 아래 명령어를 수행한다면

```bash
git remote add origin address
```

.git/config 파일에 아래와 같은 내용이 추가되어져!!

```
[remote "origin"]
	url = https://testurl.com
	fetch = +refs/heads/*:refs/remotes/origin/*
```

origin은 관례적으로 외부 repository에 붙이는 alias라고 생각하면 돼.

<br/><br/><br/><br/>

# git pull vs git fetch

<br/><br/>

## git pull

git pull 명령어를 실행하면 원격저장소의 커밋을 가져와.

그리고 그렇게 받아온 최신 커밋에 대해 refs/heads/current_branch 가 가리키게 하는거야!!

그와 동시에 refs/remotes/origin/current_branch 가 가리키게 해.

즉, refs/heads/current_branch 와 refs/remotes/origin/current_branch 가 가리키는 커밋 내용이 동일하다는 것이야. 이 내용은 현재 자신의 로컬 브랜치의 내용이 원격 브랜치의 내용과 합쳐졌다고 볼 수 있어.

<br/><br/>

## git fetch

git fetch의 경우 pull과 유사하지만 조금 달라. fetch의 경우에는 refs/heads/current_branch의 내용은 변화시키지 않고 refs/remotes/origin/current_branch의 내용을 변화시켜.

즉 pull과 fetch의 차이는 원격저장소의 내용을 가져와서 병합을 하는지 안하는지의 차이야!! 

병합을 하지 않는다면 가져와서 차이점들을 볼 수 있는 이점을 가질 수 있겠지.

<br/><br/><br/><br/>

# git tag

<br/><br/>

## git tag란?

tag는 특정 커밋을 고정적으로 가리키는 놈이라고 생각하면 될 것 같애!

이런 성질을 이용해서 어떤 릴리즈한 버전들에 대해 구분할 때 사용하면 좋겠지! 물론 실제 그렇게 사용하고도 있는거고.

```bash
git tag tag_name
```

위의 명령어를 통해 간단히 태그를 생성할 수 있어.

여기서 뭔가 추가적인 정보를 태그에 주고싶다한다면 annotated tag를 사용하면 돼

```bash
git tag -a tag_name -m "message"
```

위 처럼 사용하면 돼!! tag에 대한 간단한 메시지와 그 외에도 생성한 사람에 대한 정보(author)라던가 기타 정보가 함께 만들어져.

<br/><br/>

## git tag의 원리

git tag 명령어를 실행하게 되면 refs/tags/tag_name 파일이 생성이 돼!

<img width="312" alt="스크린샷 2022-02-13 오후 11 09 23" src="https://user-images.githubusercontent.com/31160622/153757435-771524ef-2859-4128-a1c6-2dee99334596.png">


이렇게 말이야!!

내용을 봐서 알겠지만 현재 브랜치가 가리키고 있던 commit 내용에 대한 정보를 가지고 있어!!

즉 저 object id가 결국 특정 커밋에 대한 버전을 지정하려는 의도로 tag 명령어를 사용하였을텐데 그 대상이 되는 커밋 정보라는 거지!!

annotated tag는 생성되는 정보가 조금 달라.

이 친구는 objects/ 하위에 특정 오브젝트가 생겨. 그리고 그 오브젝트는 각종 설명(태그 정보, 태그 내용, 누가 만들었는지)와 함께 특정 object id를 내용으로 가지고 있어.

그리고 저기서 Object id가 버전을 붙이고자하는 커밋에 대한 정보를 가지고 있는거지.

그 이후에는 위와 동일한데 refs/tags/tag_name 파일이 object/ 하위에 생겼다던 그 오브젝트를 가리키고 있어.
