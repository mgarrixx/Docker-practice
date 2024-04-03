## **1\. Docker 네트워크 구성**

Docker 컨테이너는 격리된 환경에서 개별적으로 동작하기에, 초기에는 컨테이너간의 통신이 막혀있습니다. 컨테이너 간의 연결을 위해 주로 제공되는 네트워크 형태는 다음과 같습니다.

-   `Bridge`: 여러 컨테이너를 서로 연결시키 위한 **가상 private 네트워크**
-   `Overlay`: **서로 다른 호스트** 및 도커 데몬에 속하는 컨테이너끼리 연결하는 네트워크 (주로 Swarm에서 쓰임)
-   `Host`: **호스트 컴퓨터와 동일한 환경**에서 동작하는 네트워크
-   `None`: 네트워킹이 **일체 불가능**한 상태

## **2\. Bridge 네트워크**

이 중에서 **Bridge 네트워크**는 Docker의 기본 네트워크 옵션입니다. 네트워크 생성 명령어를 입력할 때 특별한 타입을 지정하지 않으면 Bridge 네트워크가 만들어진다고 보시면 됩니다. Docker 시스템은 default bridge인 '`docker0`'를 갖고 있으며, 일반적으로 컨테이너들은 `docker0`로 연결된 상태로 생성되어 상호간의 통신이 가능합니다. 컨테이너 간의 네트워크 환경을 분리하고자 할 경우엔 사용자 지정 네트워크를 생성하여 원하는 컨테이너들만으로 네트워크를 구축할 수 있습니다.

<p align="center">
<img src="https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FcaQkC6%2FbtrZjeSOl1c%2FFhDBhas939ybz6OCEtoKm0%2Fimg.png" height=384px>
</p>

### **Host 네트워크와의 차이점**

`Host` 네트워크에 속하는 컨테이너들은 호스트PC와 동일한 Namespace를 가지게 되며, 별도의 IP를 할당받지 않습니다. 따라서, 컨테이너에 특정 포트(예를 들어 80번)을 바인딩시켰다면 `[호스트IP]:80` 으로 네트워킹이 가능합니다.

이에 반해, `Bridge` 네트워크는 엔진이 각 컨테이너에게 내부 IP를 순차적으로 지정해줍니다. 컨테이너는 언제든 변경되고 사라질 수 있기 때문에 앞서 할당받는 IP는 고정이 아닙니다. 그렇다보니 내부 IP 주소만으로 컨테이너 간 통신을 하려하면 일이 복잡해질 수 있고, 이를 보완하는 차원에서 도커 호스트에 생성된 DNS 서비스가 각 애플리케이션의 이름 관리 및 네트워킹 조율을 맡고 있습니다.

### **Bridge 네트워크 연결**

```
# network 인터페이스 조회
$ docker network ls

# 컨테이너 생성 및 네트워크 연결 (Default bridge)
$ docker run -itd --name=web nginx

# network 인터페이스 정보 세부적으로 확인 
$ docker network inspect [네트워크명]
```

<p align="center">
<img src="https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FvjYJj%2FbtrZBrZYrRs%2FwK3bnflfdcWbEFYpmMakM1%2Fimg.png" height=384px>
<img src="https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FcvJ2hV%2FbtrZGWSc8Wx%2FMcnOp94qMjB5dik0nQKFWK%2Fimg.png" height=384px>
</p>


사실 위의 컨테이너를 생성하는 명령어에서 network 관련 옵션을 따로 지정하지 않았습니다. 이런 경우 컨테이너는 자동으로 Default Bridge에 붙게 됩니다. 현재 환경에서의 Default Bridge의 이름은 '`bridge`' (Network ID : `85bb6f0ba4fd`)로 표기되어 있네요.

### **Default Bridge vs User-defined Bridge**

Docker 환경에서는 앞서 잠깐 나온 `docker0`을 **Default Bridge**라고 따로 종류를 구분합니다. Default Bridge의 특징을 정리해보자면 다음과 같습니다.

-   **Docker 설치 단계부터 존재**
-   **Gateway로 `172.17.0.1`이 할당**
-   **서브넷의 길이는 16 (`172.17.0.0/16`)**
-   **DNS 서비스 이용 불가 (IP로만 통신해야함)**

<p align="center">
<img src="https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FcoCPXM%2FbtrZB25uxlk%2FEMnZ411Dq84xufR3tdbj70%2Fimg.png" height=384px>
</p>


`docker0` 말고도 사용자는 직접 Bridge 네트워크를 만들수 있고, 이를 **User-defined Bridge**라고 부릅니다. User-defined Bridge는 여러 네트워크 설정(MTU, iptables 룰)이 가능하고, Default Bridge에서는 구동시키지 못했던 DNS 서비스를 이용할 수 있습니다. 여러 방면에서 실제 프로덕션 단계에는 User-defined Bridge를 사용하는게 바람직합니다.

### **User-defined Bridge 활용해보기**

#### **네트워크 생성**

```
$ docker network create my_bridge
```

<p align="center">
<img src="https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FN8PfH%2FbtrZESoywcH%2FiBkEt6F137ZXTycpm0ib2K%2Fimg.png" height=384px>
</p>


#### **컨테이너 생성**

```
// 컨테이너 생성
$ docker run -d --net=my_bridge --name=db redis

// 연결된 네트워크 확인
$ docker inspect --format='{{json .NetworkSettings.Networks}}' db
```

<p align="center">
<img src="https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FbJuWYr%2FbtrZBtb6TAv%2FvYh0ITKbs5zKJwmcf7MsYk%2Fimg.png" height=256px>
</p>

## **3\. 컨테이너를 여러 Bridge와 연결하기**

앞의 단계를 모두 마무리하셨다면 도커 내부는 아래 이미지처럼 구성되어 있을 겁니다. 두 Bridge `docker0`과 `my_bridge`는 같은 호스트 내부에 있긴해도, 그 위에서 동작하는 `web`과 `db`는 각각 별개의 가상 네트워크 인터페이스로 분리되어 있기에 서로 통신이 불가능합니다.

<p align="center">
<img src="https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FzzUGz%2FbtrZD17ri9J%2FA6KDISyHwnyoCRX7bnuwwk%2Fimg.png" height=256px>
</p>


`web` 컨테이너에 할당된 IP는 `172.17.0.2`이고, `db` 컨테이너에서 해당 주소로 ping을 날려보면 모든 패킷이 드랍됨을 확인할 수 있습니다.

<p align="center">
<img src="https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FcmHLvR%2FbtrZIcfUeOr%2FX4qx6tMjBjroP3fPmTMRE0%2Fimg.png" height=192px>
<img src="https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FkQQiW%2FbtrZImWUNiC%2F69SWieoOXtH29HKfng2oPK%2Fimg.png" height=192px>
</p>


Docker는 컨테이너를 여러 네트워크에 자유자재로 붙일수 있게 허용합니다. `web` 컨테이너를 `my_bridge`에 연결하면 아래와 같이 `my_bridge`를 조회해볼 시 `web` 컨테이너 정보가 추가되었음을 확인 가능합니다.

```
$ docker network connect my_bridge web
```

<p align="center">
<img src="https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FbpIEjI%2FbtrZD1TWFOA%2FfiDb1cWuJIz3w9NQw6RtsK%2Fimg.png" height=384px>
</p>


나아가, `web` 컨테이너를 조회하면 기존에는 `172.17.0.2`라는 IP만 할당되었던 것과는 다르게 새로운 IP `172.19.0.3`을 추가되었음을 인지하셨을 겁니다. 이는 기존 `my_bridge`의 IP 대역 `172.19.0.0/16`에 속하는 주소로, `my_bridge`와 원래부터 연결되어 있던 `db`가 새로 연결된 `web`과 통신을 할때 사용하게 됩니다.

```
$ docker inspect --format='{{range .NetworkSettings.Networks}}{{split .IPAddress ","}}{{end}}' web
```

<p align="center">
<img src="https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2Fcf5WtW%2FbtrZBGbHYBu%2FZ2lM2CsaK1y5LlgQauidPk%2Fimg.png" height=192px>
</p>


```
$ docker container exec -it db bash

# db 컨테이너 쉘에서 컨테이너 명('web')으로 직접 통신이 가능합니다.
$ ping web
```

<p align="center">
<img src="https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FU5hIg%2FbtrZInhlNw8%2FmFnzn1SFcTK55n2W9eDcQ0%2Fimg.png" height=192px>
</p>


#### **생성된 컨테이너의 연결/연결 끊기**

```
# 네트워크 간 연결/연결 끊기
$ docker network connect/disconnect [인터페이스 ID/이름] [컨테이너 ID/이름]
```