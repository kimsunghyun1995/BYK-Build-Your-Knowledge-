# 개요

협업을 하면 가장 많이 사용할 git 명령어 중 pull, push의 속도가 현저히 느려짐을 체감했다.

아무래도, 여러 게임들이 하나의 프로젝트에 들어가기도 했고, 브랜치를 머지 후에 삭제하는 옵션을 적용안시켰기 때문에, 사용하지 않은 브랜치가 계속 쌓이고 있었다.

원격 저장소 뿐만 아니라, 사용하고 있는 Git 크라켄의 최적화에 대해서도 정리하였다.

## 1\. git gc 명령어를 이용한다.

garbage collect을 이용하여, 저장소를 효율적으로 관리한다. 자세한 설명은 아래 링크 참조

-   [https://git-scm.com/docs/git-gc](https://git-scm.com/docs/git-gc) (깃 공식 Doc)

```
## 깃 크라켄 내 터미널에서는 사용 불가 -> git bash 이용 추천

GC 하려는 폴더로 이동 

git gc 명령어 또는 git gc --aggressive

(git gc --aggressive 말 그대로 공격적으로 처리하는 작업이다. 리소스, 시간이 많이 드니, 주의해서 사용)
```

만약 아래와 같은 에러가 발생한다면, 다른 프로그램에서 git을 사용하고 있는 거라, 재부팅 or 깃을 lock 하고 있는 프로세스를 찾아 종료시키면 된다.

## 2\. 사용하지 않는 브랜치를 삭제한다.

현재 사용하고 있는 브랜치가 없으므로, develop을 제외한 나머지 브랜치를 삭제한다.

[##_Image|kage@nmn6k/btssBX8KEjS/klPPT0jod4TFuqzNlqynak/img.png|CDM|1.3|{"originWidth":1231,"originHeight":194,"style":"alignCenter"}_##]

## 보너스 - 깃 크라켄 최적화 하기

필자는 Git Tool로 깃 크라켄을 사용하는데, 로컬에서도 느려짐이 체감되어 추가로 아래와 같은 작업을 하였다.

## 3\. SOLO 기능을 사용한다.

[##_Image|kage@bhrTgn/btssveRpMnN/AiA2cybKY47FHHOjobW5i0/img.png|CDM|1.3|{"originWidth":669,"originHeight":587,"style":"alignCenter"}_##]

## 4\. Max Commits in Graph 개수를 줄인다.

[##_Image|kage@4gRKb/btsswWQeKaC/lG7dpkFHWSqH9JhlngQk40/img.png|CDM|1.3|{"originWidth":959,"originHeight":430,"style":"alignCenter"}_##]

### 로컬 브랜치 삭제가 귀찮다면 새로 git clone 하는 방법도 있다.

[##_Image|kage@bZKuai/btssv1RLq4O/hDoQr8frKoztDhMXIVzTc0/img.png|CDM|1.3|{"originWidth":198,"originHeight":178,"style":"alignCenter"}_##]

# REFERENCE

[https://git-scm.com/docs/git-status](https://git-scm.com/docs/git-status)  
[https://help.gitkraken.com/gitkraken-client/performance-issues/](https://help.gitkraken.com/gitkraken-client/performance-issues/)  
[https://stackoverflow.com/questions/45335949/git-pull-failed-unable-to-unlink-file-invalid-argument](https://stackoverflow.com/questions/45335949/git-pull-failed-unable-to-unlink-file-invalid-argument)