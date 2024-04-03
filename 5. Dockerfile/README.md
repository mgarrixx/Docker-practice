**Dockerfile**은 사용자가 구축하고자 하는 **Docker 컨테이너의 configuration 정보를 담는 설정 템플릿**입니다. 이 스크립트 파일 안에는 기본 뼈대가 될 컨테이너 이미지(ex., Ubuntu, NginX 등), 환경 변수, 빌드 후 실행할 코드 등을 정의할 수 있습니다.

## **1\. Dockerfile 기본 구문**

<p align="center">
<img src="https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FXLZ89%2FbtsfdaOt4ni%2FFcI54LAKauIZaCUYBskD1k%2Fimg.png" width=512px>
</p>

#### **FROM**

배포할 컨테이너 **기본 이미지**를 지정합니다.

```
FROM debian:buster-slim
```

#### **ENV**

컨테이너 동작 과정에서 사용할 **환경 변수**를 설정합니다.

```
# ENV [변수 명] [변수 값]
ENV NGINX_VERSION   1.20.1
```

#### **WORKDIR**

컨테이너를 실행했을 때, **홈 디렉토리 역할을 할 경로**를 지정합니다. 뒤에 설명할 `RUN`, `CMD`과 같은 명령어도 해당 경로를 기준으로 실행됩니다.

```
WORKDIR /usr/src/app
```

#### **COPY 또는 ADD**

**`COPY`**는 호스트의 파일 시스템에서 컨테이너 내부로 복사할 파일들을 지정하는 옵션입니다. 이와 비슷한 옵션으로 `**ADD**`라는 게 하나 더 있으며, `ADD`는 로컬 파일 경로 이외에도 URL 정보를 인자로 넣을 수 있습니다.

```
# COPY [호스트 내 원본 파일 경로] [컨테이너 내부 경로]
COPY docker-entrypoint.sh /
```

#### **EXPOSE**

컨테이너에서 지정한 포트 번호로 외부와의 통신이 가능하게끔 합니다.

```
EXPOSE 80
```

#### **RUN 또는 CMD**

`**RUN**`과 `**CMD**`는 **컨테이너에서 실행될 쉘 명령어 리스트**입니다. 둘의 차이는, `RUN`은 컨테이너 빌드 과정에서 실행되는 명령어이고 `CMD`는 컨테이너 빌드가 끝난 이후에 실행되는 명령어를 지정합니다. 예를 들어 컨테이너에서 Nginx 웹 서버를 호스팅하는 경우 `CMD nginx` 명령을 Dockerfile 내에 포함시켜 컨테이너 빌드 이후 바로 웹 서버가 돌아갈 수 있도록 합니다.

```
RUN set -x \\
&& addgroup --system --gid 101 nginx \\
&& adduser --system --disabled-login --ingroup nginx --no-create-home --home /nonexistent --gecos "nginx user" --shell /bin/false --uid 101 nginx \\
&& apt-get update
```

## **2\. 컨테이너 이미지 빌드 예제**

nginx 애플리케이션을 Dockerfile로 띄워보고자 합니다. 기본적으로 nginx 컨테이너를 생성하면 브라우저에서 좌측과 같은 페이지를 확인할 수 있었는데요. 이번에는 특별히 빌드 과정에 다른 html 파일을 삽입하여 첫 화면이 우측처럼 나타나게끔 하는 예제를 수록하였습니다. 

<p align="center">
<img src="https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FcVZsj6%2FbtsfezUvlEi%2F2Ha7amv5Pb4heduCXVveHk%2Fimg.png" width=512px>
</p>

#### **Dockerfile**

```
# Dockerfile

FROM nginx:latest

WORKDIR /usr/share/nginx/html

COPY index.html index.html

EXPOSE 80
```

컨테이너 이미지는 최신 버전 nginx로 설정하고, 아래 `index.html` 파일을 배치할 컨테이너 내부 경로(`WORKDIR`)와 `COPY` 명령어를 작성하였습니다. 위 예시에서 로컬 파일시스템에 있는 index.html 파일은 컨테이너 안 `/usr/share/nginx/html/index.html` 경로에서 확인할 수 있습니다. 마지막 줄은 nginx를 웹 브라우저에서 접근할 수 있도록 80번 포트를 `EXPOSE` 옵션으로 개방하는 내용입니다.

#### **index.html**

```
<!-- index.html -->
<!doctype html>
<html>
<head>
  <meta charset="utf-8">
  <title>My Nginx</title>
</head>

<body>
  <h1>SUCCESS!</h1>
</body>
</html>
```

#### **이미지 빌드**

Dockerfile을 기반으로 이미지를 빌드하는 명령어는 아래와 같습니다.

```
# docker image build -t [이미지 명] [Dockerfile이 있는 디렉토리]
$ docker image build -t myimg .
```

<p align="center">
<img src="https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2Fp1czZ%2FbtsfaMOehlb%2FmeuK0ou4pQKbl4wy1f0Pz0%2Fimg.png" width=512px>
</p>

이미지를 빌드하면 `docker image ls` 명령어로 방금 생성한 이미지가 Docker 엔진에 등록되었음을 확인 가능합니다. 이제 `docker run` 명령어를 이용하여 **`myimg`**를 기반으로 하는 nginx 컨테이너를 생성합니다. 브라우저를 켜고 주소창에 `localhost:80`을 입력했을 때, 아래 2번째 이미지와 같은 화면이 나타나야 정상입니다.

```
docker run --name myapp -p 80:80 --rm myimg
```

<p align="center">
<img src="https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FcLCS0t%2FbtsffuMbuUo%2Fe5eVvsXXCIcOnKFKimUS5k%2Fimg.png" width=512px>
</p>

<p align="center">
<img src="https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2Fl7Sxp%2Fbtse98quOyX%2FPR4KBDbf9rGphOIEh648W1%2Fimg.png" width=512px>
</p>