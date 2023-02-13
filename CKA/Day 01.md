# Section 1: Introduction

## Kubernetes

오늘날 클라우드 컴퓨팅에서 가장 유행하는 기술 중 하나이다.
클라우드 플랫폼에 상관없이 지원되며, 광활하고 복잡한 기술을 만드는 다양한 종류의 아키텍처의 향상된 호스팅과 복잡한 응용 프로그램도 지원한다.

<br>

## CKA 과정

고가용성 클러스터 배포, 모니터링 관리, 보안, 저장소 문제 해결 등에 대한 제품 사용 사례를 다룬다.

<br><br>

# Section 2: Core Concepts

-   Cluster Architecture
-   Pod, ReplicaSet, Deploy, Service, ...

<br>

## Cluster Architecture

**Node Set**

-   Worker Node : 컨테이너를 로딩할 수 있음

-   Master Node
    <br>쿠버네티스 클러스터 관리, 서로 다른 노드에 대한 정보 저장, 노드와 컨테이너 모니터링 등

<br>

**ETCD**

키 값 형식으로 정보를 저장하는 DB

<br>

**Kube-scheduler**

컨테이너를 설치하기 위해 컨테이너 리소스 요구 사항, 워커 노드 용량, taint와 toleration 등과 같은 정책이나 제약 조건을 확인하여 적절한 노드를 식별

<br>

**Controller-Manager**

-   node-controller
    <br>
    클러스터에 노드를 온보딩하고, 노드를 사용할 수 없거나 파괴될 경우 처리

-   replica-controller
    <br>
    설정된 컨테이너 수를 ReplicaSet을 통해 항상 유지되도록 보장

<br>

**Kube API**

외부 사용자가 클러스터에서 관리 작업을 수행할 수 있는 api 서버

<br>

**Container Runtime Engine**

컨테이너를 실행할 수 있는 소프트웨어

예. Docker, ContainerD

<br>

**Kubelet**

클러스터의 각 노드에서 실행되는 에이전트

-   Kube API 서버의 지시에 따라 노드에 컨테이너를 배포하거나 제거
-   Kube API에게 노드와 컨테이너 상태 제공

<br>

**Kube-proxy**

클러스터 내부에 존재하는 서비스 간 통신을 가능하게 함

<br><br>

## ETCD

분산되어 있고, 신뢰성 있는 key-value 저장소로, 간단하고, 안전하며 신속함!

### key-value 저장소

<br>

![](https://velog.velcdn.com/images/ramu/post/b321ffc0-eb22-4732-9fb5-fc5bf8b3c872/image.png)

<br>

관계형 데이터베이스에서는 컬럼(속성)을 추가할 경우 테이블 전체와 각 행(레코드)에 영향을 미친다.

<br>

key-value 저장은 각 개체마다 파일에 자신의 정보를 저장할 수 있기 때문에 속성을 추가해도 다른 개체에 영향을 미치지 않는다.

<br>

![](https://velog.velcdn.com/images/ramu/post/9cb39035-26f9-4398-9bad-c0bac36846b5/image.png)

<br>

### 설치 방법

1. Binary 파일 다운로드
   github에서 바이너리 파일을 다운로드

2. Extract
   압축 해제

3. ETCD 실행파일 실행

<br>

### 역할

**클러스터 정보 저장**

Node, Pod, Config, Secret, Account, Role, Binding, ...

<br>

**`kubectl` 실행을 위한 정보 제공**

노드 추가, 파드 배포, 레플리카셋 추가 등 `kubectl`을 사용한 작업들의 결과가 ETCD 서버에 업데이트 되었을 때에만 작업이 완료됨

클러스터 내 모든 변경 사항이 ETCD 서버에 업데이트 된다.

<br>

### ETCD 클러스터 배포 방법

**직접 설치** - binary 파일 설치

마스터 노드에서 직접 바이너리 설치 및 etcd 구성

<span style="background-color: pink">전달 옵션</span>

-   인증서(Certification) : TLS 인증
-   advertise-client-urls : ETCD가 청취하는 주소로, 서버 IP와 2379번 포트 ▶ kube API가 etcd와 통신하고자 할 때 사용

**클러스터로 구성** - `kubeadm`을 이용한 배포
`kube-system`이라는 네임스페이스 내에 pod 형태로 etcd 마스터 등 etcd 서버 배포

-   `kube-system` 네임스페이스 내의 pod 확인

```
kubectl get pods -n kube-system
```

![](https://velog.velcdn.com/images/ramu/post/be7edf4d-e4c5-47f5-9513-c4f81cda49b0/image.png)

<br>

쿠버네티스는 디렉터리 내에 데이터를 저장한다.
루트 디렉터리는 레지스트리로 사용되며, 그 안에 다양한 쿠버네티스 구성 요소가 저장된다.

<br>

### 고가용성 환경의 ETCD

고가용성 환경에서는 마스터 노드가 여러 개이다. 또한 마스터 노드에 여러 개의 etcd 인스턴스가 존재한다. 이렇게 분산된 etcd는 서로에 대해 알고 있어야 한다.

![](https://velog.velcdn.com/images/ramu/post/47e5a00b-b4c0-4eef-89bd-1013aef2f178/image.png)

이를 위해 초기 클러스터 옵션에서 매개 변수를 설정해주어야 한다.

<br>

<div style="background-color: pink"><strong>💻 etcdctl</strong></div>

<br>

`etcdctl`은 etcd와 상호작용하기 위한 CLI 도구이다.

`etcdctl`은 etcd 서버와 통신할 수 있고, 버전 2와 버전 3이 존재한다. 기본 버전은 버전 2이며, 버전마다 다른 명령어를 가지고 있다.

<br><br>

## Kube-api server

api-server는 **쿠버네티스의 주요 관리 구성요소**이다.

<br>

### kubectl을 통해 명령 실행

<br>

![](https://velog.velcdn.com/images/ramu/post/1db35edc-c086-45fe-a481-5d778b285932/image.png)

1. `kubectl` 명령을 실행하면 kube-api server가 가장 먼저 요청을 보낸 사용자를 인증한다.

2. 이후 요청의 유효성을 검증한다.

3. 이후 etcd 클러스터로부터 데이터를 받아 요청에 대한 정보를 전달한다.

<br>

### api를 직접 호출할 경우

<br>

![](https://velog.velcdn.com/images/ramu/post/0fd9c4af-d57f-48f1-8cc0-8b47b1d8b23a/image.png)

1. 사용자가 post 요청으로 pod 생성을 요청하면 api server는 pod를 생성하고, 이 pod는 etcd 서버에 사용자에 대한 정보를 업데이트 한다.

2. 스케줄러는 api 서버를 지속적으로 모니터링하고 있다.
   새로운 pod가 생성된 것을 확인하면, 스케줄러는 새 pod를 띄울 노드를 식별하여 api 서버에 전달한다.

3. api 서버는 이에 대한 정보를 etcd에 업데이트 한다.

4. kubelet은 해당 노드에 pod를 생성하고, 컨테이너 런타임 엔진에 애플리케이션 이미지 배포를 지시한다.

5. 해당 작업이 완료되면 kubelet이 상태를 업데이트 한 후, 이를 api 서버에 전달한다.

6. api 서버는 해당 내용을 etcd에 업데이트 한다.

<br>

> **😎 정리**

-   kube-apiserver는 요청의 인증과 유효성을 확인하고, etcd 서버의 데이터 검색, 업데이트를 담당한다.
-   kube-apiserver는 etcd와 직접 상호작용하는 유일한 구성요소이다.
-   스케줄러, controller-manager, kubelet은 apiserver를 통해 각 영역의 클러스터에서 업데이트를 수행한다.

<br><br>

## Kube-Controller-Manager

쿠버네티스에는 Node Controller, Replication Controller 등 다양한 컨트롤러가 있다.

![](https://velog.velcdn.com/images/ramu/post/593d70fa-8330-4cfb-bb28-0152a2ea0689/image.png)

<br>

컨트롤 매니저는 쿠버네티스에서 **다양한 컨트롤러를 관리**하는 구성요소이다.

![](https://velog.velcdn.com/images/ramu/post/7b657575-ee6e-4fb0-adc3-7386f227b025/image.png)

지속적으로 **컴포넌트의 상태를 모니터링**하고, 전체 **시스템이 의도된 기능과 상태로 동작**하도록 한다.

<br>

### Node Controller

노드의 상태를 모니터링하고, 애플리케이션 실행을 유지하기 위해 필요한 기능을 api 서버를 통해 수행한다.

노드의 상태를 5초 마다 모니터링하다가, 노드가 멈추면 40초 뒤 노드의 상태를 unreachable로 표시한다.

```
Node Monitor Period = 5s
Node Monitor Grace Period = 40s
```

이후에도 5분 동안 다시 되돌아오기를 기다린다.

```
POD Eviction Timeout = 5m
```

5분이 지나도 상태가 회복되지 않는다면 해당 노드에 할당된 pod를 제거한 후 다른 정상 노드에 replica 수에 맞춰 재할당한다.

<br>

### Replication Controller

Replica Set의 상태를 모니터링하고, 만약 pod가 죽으면 다른 pod를 재생성하여 설정된 만큼의 pod가 항상 존재하도록 한다.

<br>

이외에도 다양한 컨트롤러들이 존재한다.
쿠버네티스의 많은 구성요소들은 컨트롤러들에 의해 구현된다.

그렇다면 컨트롤러는 어떻게 구성요소들을 컨트롤 하는 것이고, 어디에 위치하고 있을까?

> 컨트롤러들은 `kube-controller-manager` 속에 패키지화 되어있다. 컨트롤러 매니저를 설치하면 다른 컨트롤러도 함께 설치된다.

<br>

**설치**

```
wget https://storage.googleapis.com/kubernetes-release/release/v1.13.0/bin/linux/amd64/kube-controller-manager
```

<br><br>

## Kube Scheduler

node와 pod의 일정 관리를 수행한다.

> 어떤 pod가 어떤 node에 생성될지 결정하는 구성요소이다.

pod를 node에 생성하는 역할은 `kubelet`이 담당한다.

<br>

### 스케줄 방법

1. pod에 맞지 않는 node는 제외한다.

-   pod의 리소스 요구사항(cpu, memory)을 확인한다.
-   node의 크기가 pod를 충분히 수용할 수 있는지 확인한다.
-   특정 응용 프로그램만을 전용으로 담는 node가 존재할 수 있다.

2. 우선순위를 산정하여 적절한 node를 선택한다.

-   우선순위 함수 등으로 기준을 정해 노드에 점수를 매긴다.
-   우선순위가 가장 높은 node를 선택한다.

<br>

**설치**

```
wget https://storage.googleapis.com/kubernetes-release/release/v1.13.0/bin/linux/amd64/kube-scheduler
```
