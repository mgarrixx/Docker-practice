Docker가 생성한 컨테이너, 시스템, 가상 네트워크 등은 사실 **일시적으로만 유지되는 휘발성 자원**입니다. 컨테이너 내부의 데이터는 호스트 머신 어딘가에 저장되어 있기야 하겠지만, 단일 메모리 위치에 블럭 형태로 모여 있는 것도 아닐 뿐더러, 컨테이너가 종료 또는 재실행되는 과정에서 해당 영역이 다른 데이터로 덮어 씌워질 수 있습니다.

열심히 컨테이너 안에서 작업을 하거나 데이터를 수집했는데 이런 것들이 모르는 사이에 날라가면 좋진 않겠죠. 시스템이든 애플리케이션이든 언제 어떻게 업데이트 해야되고 확장, 재배치해야 할지는 당장 알 수 없는 일이니까요.

다행히도, Docker는 컨테이너 내부의 데이터가 영구적으로 유지되고 안전하게 관리 받을 수 있게 하는 체계가 존재합니다.

## **1\. Docker의 데이터 관리 유형**

<p align="center">
<img src="https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FehJNoa%2Fbtr91QV3EAy%2F0HVLOTmSSRmRtzEs1t6V7k%2Fimg.png" width=384px>
</p>

크게는 다음 **3가지 방식**으로 컨테이너 데이터를 관리할 수 있습니다.

### **1-1. Volume**

Docker는 호스트로부터 디스크 영역을 할당 받아 Docker 서비스만을 위한 데이터 **Volume**을 구성할 수 있습니다. Docker를 Linux 환경에서 구축했다면, `/var/lib/docker/volumes/` 라는 디렉토리 내에서 컨테이너 데이터가 영구적으로 보관됩니다. **Volume** 디렉토리는 Docker 엔진에 의해 철저하게 관리(위 그림에서 '_Docker area_')되므로 일반적으로는 사용자 또는 타 프로세스가 해당 영역에 직접 접근하는 것은 불가능합니다.

Volume이 만들어지면 기존 컨테이너들과 연결 또는, 새 컨테이너를 생성할 때에도 Volume이 마운트될 경로를 지정하여 데이터 자원을 자유롭게 공유/활용할 수 있습니다.

### **1-2. bind mount**

유사하게 **bind mount** 방식 역시 호스트 머신의 File system 상에 데이터 저장소를 두지만, 해당 영역은 Docker 엔진의 액세스 제어 권한 밖에 있습니다. 즉, 사용자가 호스트 머신 내 임의의 경로를 mount 타깃으로 설정할 수 있고 경로 내의 파일이나 데이터를 Docker 프로세스와는 별개로 읽고 쓸 수 있습니다.

다만 이러한 방식은 보안적으로 치명적일 수 있는데요. **컨테이너에서 동작하는 프로세스에 의해 호스트 머신에 저장된 파일과 데이터에 영향을 줄 수 있기 때문**입니다. 그래서 가급적이면 bind mount보다는 Volume을 활용하여, 공유 자원을 호스트 머신 내 따로 격리해둔 공간에 할당하는 것이 바람직합니다.

### **1-3. tmpfs mount**

앞서 2가지 방식과는 달리, 데이터를 호스트 머신의 메모리에 일시적으로 로드하는 방법입니다. 컨테이너 수명 주기 안에서만 데이터가 유지되며, 임시로 쓰일 자원과 데이터 또는 바로바로 폐기해야하는 민감 데이터를 다루는 컨테이너에 적합한 관리 유형입니다.

## **2\. Volume 활용해보기**

#### **volume 생성**

```
# Volume 생성
$ docker volume create myvol

# Volume 조회
$ docker volume ls

# Volume 정보 확인
# 호스트 머신 내에 Volume이 마운트된 위치가 나타남 
$ docker inspect myvol
```

<p align="center">
<img src="https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FbhkMVY%2Fbtsagrz3XM8%2F5jPFkPEDHPekhQHbPEYPBK%2Fimg.png" height=384px>
</p>


#### **Volume 할당** 

```
# 'shared' 디렉토리를 공유하는 컨테이너 생성 
$ docker run -it --name myapp --rm -v myvol://shared ubuntu

$ docker run -it --name myapp2 --rm -v myvol://shared alpine
```

<p align="center">
<img src="https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FlbLoo%2Fbtr96Pv2nqF%2F3bVGSzImsbd8fAiVNqAL30%2Fimg.png" height=384px>
</p>


앞서 만든 Volume **`myvol`**을 두 컨테이너 생성과 동시에 연결하였습니다. `myvol` 안에 `shared`라는 디렉토리를 공유폴더로 사용하게끔 설정하였습니다. 각 컨테이너마다 root 경로에 `shared`라는 디렉토리가 공통으로 존재하는 것을 확인해 볼 수 있을 겁니다. 

한 컨테이너가 `shared` 디렉토리 안에 새로운 파일("`hello.txt`")을 만들었다면 반대쪽 컨테이너의 `shared` 디렉토리에서도 동일한 파일을 조회해볼 수 있습니다.

<p align="center">
<img src="https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FbiSQyY%2FbtsafquTYxS%2FGpQKATxIo8WGWDJqQDTGh1%2Fimg.png" height=384px>
</p>

이 `hello.txt` 파일은 이를 생성했던 기존 컨테이너(`myapp`)가 설령 종료되거나 제거되더라도 여전히 `myvol://shared` 디렉토리 안에 존재합니다.

<p align="center">
<img src="https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2Fc9Ab7U%2FbtsaeuLwnGq%2Fwu2wkXdXcGnKBkTHKMkc1k%2Fimg.png" height=384px>
</p>

## **3\. bind mount 활용해보기**

호스트 머신의 현재 경로를 컨테이너에 마운트시켜 보았습니다. 현재 제 호스트 머신(`@pyro`)에는 웹에 표시될 HTML 파일(아래 2번째 이미지)이 있으며, 주어진 명령어를 수행하였을 때, 브라우저에서 `localhost:80`으로 접속하면 3번째 이미지처럼 "_Original_"이라는 텍스트가 나타날 것입니다.

```
<!doctype html>
<html>
<head>
    <meta charset="utf-8">
    <title>My Nginx</title>
</head>

<body>
    <h1>Original</h1>
</body>
</html>
```

```
# 현재 디렉토리 경로
$ echo $(pwd)

# 현재 디렉토리를 공유 디렉토리로 삼는 컨테이너 생성
$ docker run -d --name myweb -p 80:80 -v /$(pwd):/usr/share/nginx/html nginx
```

<p align="center">
<img src="https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FedRS7z%2FbtsrB50WhGZ%2FmZSUADbrUoaQc2rHvyEEH0%2Fimg.png" width=512px>
</p>

<p align="center">
<img src="https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FbltEil%2FbtsrB7xG401%2FwvZIYYtx3QAqd9y9V3fWtk%2Fimg.png" width=512px>
</p>

<p align="center">
<img src="https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FDpt3h%2FbtsaeTxG7hm%2FQJ7mKja1WKIba8T79iMj4k%2Fimg.png" height=384px>
</p>


여기서 Docker 컨테이너에 직접 들어가지 않고 호스트에 있는 `index.html` 파일을 아래와 같이 수정하면, 브라우저 페이지를 새로고침했을 때 그 결과물 역시 새롭게 반영되었음을 확인 가능합니다.

<p align="center">
<img src="https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FdaurZ7%2FbtsrxOFHdNr%2FiI6jLKDuaO5of30fTuToD0%2Fimg.png" width=512px>
</p>

<p align="center">
<img src="https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FmODL4%2FbtsafQ76maL%2FxaLOLlsE35uYKxQcoO9PnK%2Fimg.png" height=384px>
</p>

또한, bind mount를 설명할 때도 언급했듯이, 컨테이너 내에서 공유 디렉토리를 손 대게되면(아래와 같이 파일을 생성한다던가...) 그 결과물 역시 호스트 머신에 고스란히 반영됩니다.

<p align="center">
<img src="https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FcgpNE7%2FbtsaeuLA2mX%2FNEL8WPeKO5e9ddikHIERD1%2Fimg.png" height=384px>
</p>