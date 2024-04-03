# 1. Compose 파일 만들기

`Docker-compose`를 이용하기 위해선 먼저 **YAML 파일(.yml)**을 생성하여 동작시키고자하는 애플리케이션 스택에 대한 모든 정보를 집어 넣어야 합니다.

```bash
$ vi docker-compose.yml
```

```yaml
# docker-compose.yml
version:
services:
volumes:
networks:
```

파일명을 `docker-compose.yml` 로 짓는 것은 나중에 Docker 엔진이 기본 설정파일을 자동으로 인식하게하기 위함입니다. 

Compose 파일 안에 들어가야하는 핵심 요소는 4가지가 있는데요. 각 구성요소를 살펴보면, 

### version

Compose 파일의 Schema 버전입니다.  `1.x`, `2.x`, `3.x` 버전대가 존재하며 `3.x` 버전대가 가장 풍부한 기능을 지원하므로 가급적이면 해당 버전으로 사용합니다.

```yaml
# docker-compose.yml
version: "3.7"
```

### services

앱 컨테이너들을 정의하는 영역입니다. 각 컨테이너 별로 사용하게 될 **이미지/포트/볼륨/환경 변수**를 명시합니다. 아래 예시에는 2개의 애플리케이션 (`app`, `mysql`)을 동시에 서비스로 구성하는 내용을 담고 있습니다. 

```yaml
# docker-compose.yml
services:
  app:
    image: node:20-alpine # 빌드할 컨테이너 이미지 정보
    command: sh -c "yarn install && yarn run dev" # 빌드 후 명령어
    ports: # 외부 네트워크 연결 정보 (개방 포트)
      - 3000:3000
    working_dir: /app # 초기 디렉토리
    volumes: # 사용할 볼륨 공간 (뒤에서 부가 설명)
      - ./app:/app
    networks: # 사용할 네트워크 (뒤에서 부가 설명)
      - my-net
    environment: # 환경 변수
      MYSQL_HOST: mysql
      MYSQL_USER: root
      MYSQL_PASSWORD: secret
      MYSQL_DB: todos

  mysql:
    image: mysql:5.7
    volumes:
      - my-storage:/var/lib/mysql
    networks:
      - my-net
    environment: 
      MYSQL_ROOT_PASSWORD: secret
      MYSQL_DATABASE: todos
```

### volumes

컨테이너 동작 시에 사용할 공유 볼륨입니다. Compose 파일 안에 정의한 볼륨을 사용하고자 할 경우, `services` 항목에 속한 앱에 `volumes` 필드를 추가하시면 됩니다. 

```yaml
# docker-compose.yml
volumes:
  my-storage: # 사용자 정의 볼륨
```

<p align="center">
<img src="https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FcQ13YJ%2FbtsqrMXgLr5%2FKKRxNGG4RfXPnJJa5VOfY0%2Fimg.png" width=512px>
</p>

### networks

구축 환경에 포함시킬 네트워크 정보로, 해당 필드가 비어있을 경우 컨테이너들은 도커에서 제공하는 기본 브릿지 네트워크를 사용하게 됩니다. 

```yaml
# docker-compose.yml
networks:
  my-net: # 사용자 정의 네트워크
    driver: bridge # 네트워킹 방식
```

<p align="center">
<img src="https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FcfJDhf%2Fbtsqh3E2oug%2FDcLrKKpzekZxXddnegs7D1%2Fimg.png" width=512px>
</p>

필요에 따라 컨테이너에 여러 네트워크를 연결할 수도 있습니다. 도커에서 제공하는 기본 브릿지 네트워크를 사용하고자 한다면 아래 이미지의 중앙에서처럼 `default` 옵션을 주거나, 또는 우측 코드와 같이 `networks` 항목을 따로 두지 않아도 알아서 기본 브릿지에 연결됩니다.

<p align="center">
<img src="https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2Fs9JDg%2FbtsqrFKiIBQ%2Fd1gGIlBhIapMK850cUON3k%2Fimg.png" width=512px>
</p>

# 2. 애플리케이션 스택 실행

보다 간단한 Compose 파일 예시를 이용하여 전체 서비스를 띄워보겠습니다.

```yaml
# docker-compose.yml
version: "3.7"

services:
  myapp:
    image: nginx:latest
    ports:
      - '8080:80'
```

`docker-compose.yml` 파일이 있는 디렉토리로 들어가 아래 명령어를 입력하면 Docker 엔진이 알아서 설정 파일을 찾아 컨테이너를 생성하고 실행시킵니다.

```yaml
# 애플리케이션 스택 실행
docker-compose up -d
```

`docker ps` 명령어 또는, 브라우저에 `localhost:8080`을 입력하여 Nginx가 정상 동작하는지 확인합니다.

```bash
$ docker ps
```

<p align="center">
<img src="https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FuU6jq%2FbtsqtuIxSZ7%2Fj2pt4MknTLK6VEeMyLATek%2Fimg.png" width=512px>
</p>

<p align="center">
<img src="https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2Fb3o0c3%2FbtsqwBzVBJD%2FgPQsFAGVZDseqxY82qTMT1%2Fimg.png" width=512px>
</p>

만약 `docker-compose.yml` 파일이 아닌 다름 이름의 설정 파일을 활용하고자 한다면 `docker-compose` 명령어 뒷부분에 `-f` 옵션을 주어야 합니다. 그 외에 명령어들은 아래와 같습니다.

```bash
docker compose up -f [compose 파일 명]
```

생성한 앱 스택 전체를 종료하는 명령어는 `docker-compose down` 입니다. **주의할 건, `docker-compose down`을 수행하게되면 생성했던 컨테이너들이 모두 삭제된다는 것입니다.**

```bash
# 앱 스택 조회
docker-compose ps

# 앱 스택 종료
docker-compose down
```

<p align="center">
<img src="https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FdzKm8S%2FbtsqwBmoLb0%2F3sVYfFW2VaQUKKRQ3mt2ok%2Fimg.png" width=512px>
</p>

`docker-compose down` 이후 실행했던 컨테이너 리스트가 비게 됩니다. 

# 3. Dockerfile을 이용하여 커스텀 이미지 기반의 앱 스택 생성

이전 섹션에서는 Docker Hub에 등록된 이미지 `nginx:latest`를 이용했습니다. 이번에는 나만의 커스텀 이미지를 기반으로 앱을 구성해보겠습니다.

 커스텀 이미지를 빌드하기 위해선 Dockerfile이 필요합니다. Dockerfile을 작성하고 해당 파일이 있는 디렉토리 경로를 `services → [앱 이름] → build` 항목에 작성합니다.

```yaml
# docker-compose2.yml
version: "3.7"

services:
  myapp:
    image: custom-nginx
    build: src/ # Dockerfile이 있는 디렉토리
    ports:
      - '8080:80'
```

```docker
# src/Dockerfile
FROM nginx:latest

WORKDIR /usr/share/nginx/html

COPY index.html index.html
```

위 예시는 Nginx 초기화 이후 `src` 디렉토리에 있는 index.html을 곧바로 적용하여 “***3. 애플리케이션 스택 실행“***섹션에서 본 Welcome 화면이 아닌 새로운 웹 페이지가 나오도록 하는 튜토리얼 코드입니다.

```html
<!-- src/index.html -->

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

모든 파일들을 작성한 후에 아래 명령어를 입력하고 브라우저에 `localhost:8080`을 검색하면 아래와 같이 ***SUCCESS!***가 나타날 것입니다.

```bash
$ docker-compose up -f docker-compose2.yml -d
```

<p align="center">
<img src="https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2Fc6deuC%2Fbtsqtvgqhmc%2FmK18OqKqgn5ijlguOM9Bok%2Fimg.png" width=512px>
</p>

<p align="center">
<img src="https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FyyQsP%2FbtsqrCz6tgH%2Ful4aM6kJVVHPVsmg4AApf1%2Fimg.png" width=512px>
</p>