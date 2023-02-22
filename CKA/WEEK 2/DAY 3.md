# Section 5: Application Lifecycle Management

<br>

# Configure Applications

애플리케이션 구성은 다음 개념을 이해함으로써 이루어질 수 있다.

-   Configuring Command and Arguments on applications
-   Configuring Environment Variables
-   Configuring Secrets

<br><br>

# 🚩 Commands

CKA 자격증에 필수적인 내용은 아니지만 중요한 개념이므로 파드 구성 파일 commands 및 arguments에 대해 설명할 것이다.

<br>

파드에 대한 내용을 공부하기 전, 먼저 도커에서의 커맨드와 컨테이너 개념을 복기해보자.

<br>

우분투 이미지로 docker 컨테이너를 실행시키려고 한다.

<br>

```
docker run ubuntu

docker ps
```

<br>

`docker ps` 명령으로 실행되고 있는 컨테이너 목록을 출력해보면 실행시킨 우분투 컨테이너는 존재하지 않는 것을 확인할 수 있다.

<br>

그 이유는 무엇일까?

<br>

가상머신과 달리 컨테이너는 운영체제를 호스팅하는 목적이 아닌 특정 작업 또는 프로세스를 실행하는 목적으로 사용되기 때문이다.

따라서 작업이 중지되거나 끝나면 컨테이너가 종료되며, 컨테이너는 그 안에 있는 프로세스가 실행되고 있는 동안에만 살아있다.

<br>

컨테이너에서 실행되는 프로세스는 `Dockerfile`의 `CMD`라는 항목에서 정의된다.

<br><br>

## Dockerfile의 `CMD`

<br>

-   ![](https://velog.velcdn.com/images/ramu/post/f2faf0e5-8cd6-41af-99ff-6c02fd7f21da/image.png)

<br>

위 `Dockerfile`은 우분투 이미지의 기본 `Dockefile`이다. `Dockerfile`을 보면 `CMD`에 `["bash"]`를 사용하고 있다.

<br>

`bash`는 웹 서버나 데이터베이스 서버와 같은 프로세스가 아닌 쉘(Shell)이다.

쉘은 터미널에서 입력을 받기 때문에 터미널을 찾지 못하면 해당 이미지로 만들어진 컨테이너는 종료될 것이다.

<br><br>

### 컨테이너가 종료된 이유는 무엇일까?

<br>

앞에서 `docker run` 명령어로 우분투 컨테이너를 생성할 때, 컨테이너를 생성한 후 bash를 실행하기 위해 터미널을 찾았을 것이다.

<br>

그러나 docker는 컨테이너를 실행한 후 기본적으로 터미널과 연결시키지 않기 때문에 bash 프로세스는 터미널을 찾지 못하고 종료된다.

<br>

컨테이너에서 실행한 프로세스가 종료되었기 때문에 컨테이너도 함께 종료된다.

<br><br>

## 컨테이너 시작 시 실행될 커맨드 설정 방법

<br>

### 1) `docker run` 명령에 커맨드 추가

<br>

```
docker run ubuntu sleep 5
```

<br>

이렇게 `docker run` 명령에 커맨드를 추가하면 이미지에 지정된 default CMD를 재정의 할 수 있다.
`sleep 5` 프로세스를 실행하면 5초 뒤에 컨테이너가 종료될 것이다.

<br>

만약 이 커맨드를 영구적으로 만들고 싶다면 어떻게 해야할까?

<br>

### 2) `Dockerfile`에서 커맨드 설정

<br>

```
FROM Ubuntu

CMD sleep 5
```

<br>

또는

<br>

```
FROM Ubuntu

CMD ["sleep", "5"]
```

<br>

정의한 이미지 Dockerfile을 `build` 명령으로 빌드한다.

<br>

```
docker build -t ubuntu-sleeper
```

<br>

우리가 정의한 이미지의 이름을 `ubuntu-sleeper`로 지정하고, `docker run` 커맨드를 실행하면 커맨드가 영구적으로 반영된 이미지로 컨테이너를 생성할 수 있다.

<br>

```
docker run ubuntu-sleeper
```

<br><br>

## 5초를 10초로 늘리려면 어떻게 해야할까?

<br>

`ubuntu-sleeper`의 CMD에는 `sleep 5`라는 명령이 하드코딩 되어있다.

<br>

### 1) `docker run` 명령

<br>

```
docker run ubuntu-sleeper sleep 10
```

<br>

위 명령으로 5초를 10초로 늘릴 수 있지만 커맨드를 다시 지정하지 않아도 숫자만 추가하면 알아서 변경되길 원한다.

이 경우에는 다른 방법을 사용해야 한다.

<br>

### 2) `ENTRYPOINT`

<br>

```
FROM Ubuntu

CMD sleep 5
```

<br>

기존 `ubuntu-sleeper`의 `Dockerfile` 내용을 변경해보자.

```
FROM Ubuntu

ENTRYPOINT ["sleep"]
```

<br>

그리고 `docker run` 으로 컨테이너를 실행한다.

```
docker run ubuntu-sleeper 10
```

<br><br>

## CMD와 ENTRYPOINT의 차이점

<br>

### CMD

<br>

`CMD`는 위에서 배웠듯이 컨테이너가 실행될 때 기본적으로 수행되는 커맨드이다.

따라서 `docker run`에 추가적인 옵션이 없다면 생성된 컨테이너는 기본적으로 `CMD`에 설정된 커맨드를 실행하게 된다.

만약 `docker run`에 새로운 커맨드를 넣어 실행한다면 `CMD` 명령은 모두 새로운 커맨드로 대체된다.

<br>

### ENTRYPOINT

<br>

그러나 `ENTRYPOINT`는 새로운 커맨드의 추가 여부에 관계없이 무조건 실행되는 커맨드이다.

따라서 `docker run`에 커맨드 옵션이 추가된다면 `ENTRYPOINT`에 설정된 커맨드가 실행된 후에 추가된 커맨드가 실행된다.

<br>

우리는 `CMD`와 `ENTRYPOINT`를 적절히 사용하여 원하는 프로세스를 실행시킬 수 있다.

<br>

```
FROM Ubuntu

ENTRYPOINT ["sleep"]

CMD ["5"]
```

<br>

이렇게 설정하면 `sleep` 명령에 `CMD 5` 를 실행할 수 있다.

<br><br>
