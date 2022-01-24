# git branch와 merge의 기초

# 브랜치(Branch)란?

개발을 진행하다보면 현재 개발하는 코드를 복사하여 독립적인 개발(추가, 변경)을 진행해야하는 경우가 있지!

이때 사용하는 것이 브랜치야!!

독립적인 개발을 지원하는 것이 브랜치라고 생각하면 좋을 것 같애!

<br/><br/>

## 예시

<br/>

### 시나리오

1. git main 브랜치에서 작업을 진행 중 (file1.txt)
2. main 브랜치와는 독립적으로 다른 작업을 하고 싶어(file2.txt) 추가
3. main 브랜치에서는 위의 file2.txt에 대한 내용은 포함하지 않고 file3.txt 작업을 진행하고싶어
4. 즉 main branch : file1.txt → file3.txt 작업, 또 다른 브랜치(second_branch) : file1.txt → file2.txt

<br/>

### 과정
<img width="371" alt="스크린샷 2022-01-25 오전 1 16 14" src="https://user-images.githubusercontent.com/31160622/150827928-033b646f-9a5d-44f2-8b89-59da3aa0eac9.png">

먼저 main branch에서 file1.txt에 대한 작업을 진행 후 add 와 commit을 한다. (변경 사항을 stage area에 올린 후 해당 사항들에 대한 스냅샷을 뜬다!)

<img width="410" alt="스크린샷 2022-01-25 오전 1 21 11" src="https://user-images.githubusercontent.com/31160622/150828022-42f8cd77-f46a-43e8-a71b-22a973eec02a.png">

second_branch를 생성 후 file2.txt에 대한 작업을 진행 후 add, commit

<img width="357" alt="스크린샷 2022-01-25 오전 1 25 20" src="https://user-images.githubusercontent.com/31160622/150828132-2e00e818-e00b-4431-923f-d595f19d7f9b.png">

main 브랜치로 돌아와서 file3.txt에 대한 작업을 진행 후 add, commit

<img width="222" alt="스크린샷 2022-01-25 오전 1 26 09" src="https://user-images.githubusercontent.com/31160622/150828159-89fc773b-1657-4598-98c1-d575a119cf89.png">

위 그림처럼 main branch와 second_branch는 독자적으로 개발되는 모습을 볼 수 있어.  commit 1 (file1.txt)이라는 공통 부모를 가지지만 해당 커밋에서 독립적으로 뻗어나와 각자 개발을 진행하는 모습을 볼 수 있찌!!

main branch로 이동하면 working directory애는 file1.txt와 file3.txt가 존재할 것이고 second_branch로 이동하면 file1.txt와 file2.txt가 존재하겠지!!

이러한 **독립적인 개발을 가능하게 해주는 것이 브랜치**야!!

<br/><br/><br/>

# 병합(Merge)란?

두 브랜치의 내용을 합치는 것을 merge 라고 해!! merge에는 두가지 종류가 존재해.

먼저 시나리오를 가정해보자.

<br/><br/>

## 시나리오

1. 현재 개발을 진행중인 master 브랜치가 존재해. (현재는 main으로 default branch 명이 변경되었으나 master로 가정)
    
    ![](https://git-scm.com/book/en/v2/images/basic-branching-1.png)
    
2. 추가하고 싶은 기능이 생겨 iss53 브랜치를  새로 생성하여 작업을 진행해
    
    ![](https://git-scm.com/book/en/v2/images/basic-branching-3.png)
    
3. 이때 급하게 해결해야할 문제가 생겨 master branch에서 hotfix 브랜치를 새로 만들어서 문제를 해결해 (문제를 해결하는 과정에서 2번의 이슈와 섞이는 것을 방지하고자 master에서 hotfix 브랜치 생성)

    
    ![](https://git-scm.com/book/en/v2/images/basic-branching-4.png)
    
<br/><br/>

## Fast-forward Merge

위의 시나리오에서 3번이 현 상태라고 가정을 해볼게.

급하게 해결해야하는 문제를 hotfix 브랜치를 생성하여 해결하였어!!

그럼 다음으로 할 것은 이 해결한 내용을 master 브랜치와 합쳐주어야겠지. 그래야 반영이 되지.

master 브랜치에 hotfix 브랜치 내용을 합치기 위해서는 아래와 같은 명령어를 이용하겠지.

```bash
$ git checkout master
$ git merge hotfix
Updating f42c576..3a0874c
Fast-forward
```

그럼 위처럼 Fast-forward 라는 내용을 볼 수 있을거야.

이게 무엇인가!? 위의 3번을 보면 hotfix가 가리키는 c4 커밋이 c2 커밋에 기반하고 있찌. c4 커밋의 parent가 c2 커밋이자나. 이 경우에 master branch가 가리키고있는 c2 커밋의 내용과 hotfix가 가리키고있는 c4 커밋의 내용을 합치기 위해서는 특별한 과정 없이 master 브랜치가 c4 내용을 가리키게 변경해주면 끝나는거야!!

why? c4가 c2 기반이니까!! 

![](https://git-scm.com/book/en/v2/images/basic-branching-5.png)

위의 결과를 그림으로 표현하면 위처럼 되겠지!

이제 hotfix 브랜치는 필요없으니 지웠다고 생각하고 다음 과정을 보자

<br/><br/>

## 3-way Merge

![](https://git-scm.com/book/en/v2/images/basic-merging-1.png)

이러한 상황에서 추가적으로 진행하고 싶었던 작업이 끝나 반영하고 싶어.

즉,  iss53 branch의 커밋 내용과 master branch의 커밋을 합치고 싶다는거지. (정확히는 master branch에 iss53 내용을 합병)

이러한 경우는 위의 Fast-forward merge에서 봤던 내용과는 다르지!! iss53의 c5 커밋의 부모는 c3이고 c3의 부모 커밋은 현재 master 브랜치가 가리키고 있는 c4가 아닌 c2야!!

한마디로 master branch가 가리키는 commit 만 바꾼다고해서 해결할 수가 없는 상황이야. (master branch가 가리키는 c4 커밋에 대한 내용은 반영될 수가 없기 때문이지)

이때는 위의 c2, c4, c5 커밋 3개를 merge하여 (3-way merge) 그 결과를 별도의 커밋으로 만들고 나서 해당 브랜치(여기서는 master)가 만든 커밋을 가리키도록 이동시켜!! 이렇게 새롭게 만들어진 commit은 부모 커밋이 여러개가 되겠지. 이를 merge 커밋이라고도 해.

![](https://git-scm.com/book/en/v2/images/basic-merging-2.png)

결과적으로 위 그림처럼 되겠찌

<br/><br/><br/><br/><br/><br/><br/>

> [https://git-scm.com/book/en/v2](https://git-scm.com/book/en/v2)
> 

> [https://www.youtube.com/watch?v=PmWPdYkAMg4&list=PLuHgQVnccGMA8iwZwrGyNXCGy2LAAsTXk&index=20](https://www.youtube.com/watch?v=PmWPdYkAMg4&list=PLuHgQVnccGMA8iwZwrGyNXCGy2LAAsTXk&index=20)
>