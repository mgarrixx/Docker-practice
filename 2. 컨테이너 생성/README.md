### **컨테이너 이미지 다운로드**

Docker Hub를 통해 구동시킬 앱의 이미지를 미리 내려받습니다. 

```
# 이미지 다운로드: "docker image pull [image 이름]:[version]"
# version을 명시하지 않으면 가장 최신 버전으로 내려받습니다.
$ docker pull nginx

# 사용가능한 이미지 조회
$ docker image ls
```

<p align="center">
<img src="https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FbNy8ao%2Fbtr4sVHbqM6%2FGr6l3Odt1hXe5tH4AMoSsK%2Fimg.png" height=256px>
</p>

### **컨테이너 생성**

갖고 있는 이미지를 이용하여 컨테이너를 생성하고 관리하는 명령어입니다. 본 포스트에서는 Nginx를 생성하여 4885번 포트로 서비스를 운영하는 예시를 담았습니다.

```
# 컨테이너 생성 및 실행: "docker run [options] [Image 명]"
# 옵션 종류:
#     --detach (-d): 백그라운드 실행 체크,
#     --name: 컨테이너 이름,
#     --publish (-p): 포트포워딩 룰 설정 [Host Port]:[Guest Port]
#     -it: 컨테이너 생성과 동시에 컨테이너 내부 쉘로 접속 (ssh와 비슷)
#     --rm: 컨테이너 연결이 종료됨과 동시에 컨테이너 삭제 (-it 옵션과 함께 잘 쓰임)
$ docker run -d -p 4885:80 --name myweb nginx

# 컨테이너 로그 확인
$ docker logs myweb

# 컨테이너 종료
$ docker stop myweb

# 컨테이너 삭제
$ docker rm -f myweb

# 컨테이너에서 돌아가는 프로세스 확인
$ docker top myweb

# 컨테이너의 환경 설정을 확인
$ docker inspect myweb
```

<p align="center">
<img src="https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2Fw3wn0%2Fbtr4sTJv3Pg%2FhkxdxhQV9VLeqOwrFguhkK%2Fimg.png" height=256px>
<img src="https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2Fb2zLkQ%2Fbtr4jpbRxs9%2FQi2XfaYs2KkYzhmrKDbeqK%2Fimg.png" height=256px>
</p>

### **컨테이너 조회**

```
# 동작 중인 컨테이너 확인
$ docker container ls # 또는 docker ps

# 생성된 모든 컨테이너(동작 여부 상관x) 확인
# 또는 docker ps -a
$ docker container ls -a

# 각 컨테이너의 리소스 사용 정보를 출력
$ docker stats
```

<p align="center">
<img src="https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FcUWX9R%2Fbtr4tOgAMYZ%2FjxGehNvSAd7dnWIoJyWOO0%2Fimg.png" height=256px>
<img src="https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2Fb30Rag%2Fbtr4pYxTEh6%2FzzSvxMFHbrwfxtoyiPkj6K%2Fimg.png" height=256px>
</p>

### **컨테이너 쉘 접속**

생성된 컨테이너는 초기에 ssh 접속이 열려있지 않습니다. 컨테이너 생성 당시 `docker run` 명령어에 '`\-it`' 옵션을 주어 내부 쉘로 접근하는 방법 외에, Host PC서 직접 컨테이너 쉘로 연결하는 방법은 아래와 같습니다. 

```
# 컨테이너 접속: docker exec -it [컨테이너 이름] bash
$ docker exec -it myweb bash

# 쉘 종료
$ exit
```

<p align="center">
<img src="https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FBqCzd%2Fbtr4l2HpdDt%2FYjdyhuk9rLiAiSkK6kjBeK%2Fimg.png" height=256px>
</p>