본 포스트는 Docker Desktop 없이 WSL(Windows Subsystem for Linux)에 Docker를 설치하는 과정을 요약하였습니다.

## **1\. WSL 2 활성화**

```
wsl --set-default-version 2
```
<p align="center">
<img src="https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FDbxKk%2FbtrZAgb5IAY%2FalzhstHXHy2k39RWEdEVCK%2Fimg.png" height=256px>
</p>



## **2\. WSL용 Ubuntu 설치**

Docker 엔진이 설치될 Ubuntu 배포판을 설치합니다. 

``Microsoft Store`` 앱을 키고 Ubuntu를 검색하여 다운로드 받은 뒤, WSL 전용 터미널을 실행시킵니다. 

<p align="center">
<img src="https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FbuMEUo%2FbtrZAgJVShi%2FPdvdSlQcFxRHwAnpdJ40Zk%2Fimg.png" height=256px>
<img src="https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FoH9AP%2FbtrZAGax8vY%2FgsiRENDmJhaWjq68dhl1vK%2Fimg.png" height=256px>
</p>


## **3\. **Docker 설치****

### **패키지 업데이트**

```
sudo apt update && sudo apt upgrade
```

### **의존성 추가**

```
$ sudo apt install --no-install-recommends apt-transport-https ca-certificates curl gnupg2

$ . /etc/os-release

# ${ID}는 /etc/os-release 결과에 따라 ubuntu 또는 debian
$ curl -fsSL https://download.docker.com/linux/${ID}/gpg | sudo tee /etc/apt/trusted.gpg.d/docker.asc

$ echo "deb [arch=amd64] https://download.docker.com/linux/${ID} ${VERSION_CODENAME} stable" | sudo tee /etc/apt/sources.list.d/docker.list

$ sudo apt update
```

<p align="center">
<img src="https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FbETw9U%2FbtrZAt3prb2%2FFba1B7ZjZBsLRoP6hGodzk%2Fimg.png" height=256px>
</p>


### **본 설치**

```
$ sudo apt install docker-ce docker-ce-cli containerd.io
```

<p align="center">
<img src="https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2Ft40OB%2FbtrZAhoxiXQ%2FHxYfkAFa4PT4fcFX2VcZLK%2Fimg.png" height=256px>
</p>


## **4\. Docker group에 사용자 추가**

이 과정을 수행해야 `docker` 명령어를 수행할 때, 매번 '`sudo`'를 붙이지 않아도 됩니다.

```
$ sudo usermod -aG docker $USER
```

이후 Ubuntu를 재부팅시켜주세요 (Windows는 ×)

## **5\. Docker 테스트**

### **설치 확인**

```
$ docker version
```

<p align="center">
<img src="https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FZyXVB%2FbtrZzHVwOPC%2FWHFMmX55N0cNLAo2k4sMmK%2Fimg.png" height=256px>
</p>

### **컨테이너 띄우기**

Docker를 이용하여 nginx 서비스를 생성하고 포트 `8080`번으로 접속 가능하게 하였습니다. 브라우저 상단의 주소 창에 `localhost:8080`을 입력하면 아래와 같이 애플리케이션이 성공적으로 동작하는 화면을 확인할 수 있습니다.

```
$ sudo service docker start

$ docker run -p 8080:80 -d nginx
```

<p align="center">
<img src="https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FbtJBxT%2FbtrZzGvAGyN%2F6TkeGIvyrOrfVTqsk1HGK1%2Fimg.png" height=256px>
<img src="https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FL2kVF%2FbtrZBtveJWZ%2FiUgGE5mP7mzgeSxz5DkCpk%2Fimg.png" height=256px>
</p>