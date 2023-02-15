# 👩🏻‍🤝‍👩🏻 ReplicaSet

**컨트롤러**는 쿠버네티스의 두뇌를 담당하여 쿠버네티스 오브젝트를 모니터링하고 반응하는 프로세스이다.

<br>
<br>

## Replication Controller

<br>

복제 컨트롤러에 대해 이야기 해보자!

Replica란 무엇이고, Replication Controller는 왜 필요한걸까?

<br>

> 애플리케이션을 실행하는 단일 파드가 있다고 가정한다.

<br>

어떤 이유로 애플리케이션과 파드가 죽게 되면 사용자는 더이상 애플리케이션에 접근할 수 없다.

<span style="color: hotpink">**따라서 사용자의 애플리케이션 접근 권한을 보장하기 위해 동시에 실행되는 둘 이상의 파드를 동작시킬 수 있다.**</span>

<br>
<br>

### 1) 고가용성 (High Availability)

<br>
복제 컨트롤러는 쿠버네티스 클러스터에서 파드의 여러 인스턴스를 실행할 수 있도록 고가용성을 제공한다.

<br>

![](https://velog.velcdn.com/images/ramu/post/70380fe1-ded2-4e11-a6e1-12d0cb56628f/image.png)

<br>

여러 개의 파드가 아닌 하나의 파드만 유지하기 위해서도 복제 컨트롤러를 사용할 수 있다.

![](https://velog.velcdn.com/images/ramu/post/cdeacbda-9920-4e78-b2e8-a095cb0a7b44/image.png)

이경우에는 기존 파드에 문제가 발생하면 복제 컨트롤러가 자동으로 새 파드를 생성해준다.

<br>

> 😎 따라서 복제 컨트롤러는 특정 파드가 항상 실행되도록 보장해준다

<br>
<br>

### 2) 로드 밸런싱 & 스케일링 (Load Balancing & Scaling)

<br>

복제 컨트롤러는 부하를 분산하기 위해 여러 파드를 생성해준다.

![](https://velog.velcdn.com/images/ramu/post/7683847d-a32c-4b4f-a49b-28c495672184/image.png)

<br>

사용자에게 서비스를 제공하는 하나의 애플리케이션 파드가 있다.

![](https://velog.velcdn.com/images/ramu/post/e378310b-2703-47ba-a146-ff92028137de/image.png)

<br>

사용자 수가 증가하면 파드를 추가하여 로드밸런싱을 수행한다.

![](https://velog.velcdn.com/images/ramu/post/cb4843d5-48e2-4249-9bc9-7652baeb68f9/image.png)

파드의 증가로 해당 노드의 리소스가 부족해지면 클러스터의 다른 노드에 파드를 추가로 배포한다.

<br>

> 복제 컨트롤러는 클러스터 내의 노드들에 걸쳐있기 때문에 여러 노드에서 파드와 애플리케이션 <span style="color: #ff08a4">**로드 밸런싱과 오토 스케일링**</span>에 도움을 줄 수 있다.

<br>
<br>

## Replication Controller와 Replica Set

복제 컨트롤러와 레플리카셋의 용도는 같다. 그러나 차이점이 존재한다.

<br>

**복제 컨트롤러**

복제 컨트롤러 기술이 더 오래된 기술이며, 레플리카셋에 의해 대체되었다.

<br>

**레플리카 셋**

레플리카셋은 복제를 설정하는 새로운 권장 방법이다.

<br>
<br>

## ReplicaController 생성

<br>

### 1) YAML 파일 생성

```
vim rc-definition.yaml
```

<br>

### 2) 4개의 필수 요소 작성

```
apiVersion: v1
kind: ReplicationController
metadata:
  name: myapp-rc
  labels:
    app: myapp
    type: front-end
spec:
  template:
	# 무슨 내용이 들어가야 할까?
```

<br>

**spec**
<br>

`spec` 부분은 오브젝트 내부에 무엇이 있는지 정의한다.

현재 ReplicaController를 생성하고 있으므로, 생성할 복제 컨트롤러가 관리하는데 필요한 파드의 template를 정의하면 된다.

<br>

```
spec:
  template:
    metadata:
      name: myapp-pod
      labels:
        app: myapp
        type: front-end
    spec:
      containers:
      - name: nginx-container
        image: nginx
  replicas: 3
```

<br>

### 3) 파일 저장

esc를 누른 후 `wq` 커맨드로 파일을 저장한다.

-   w : 쓰기 작업 저장
-   q : 나가기

<br>

### 4) 오브젝트 생성

yaml 파일을 적용하여 ReplicaController를 생성한다.

```
kubectl create -f rc-definition.yaml
```

![](https://velog.velcdn.com/images/ramu/post/0e940217-c64e-47fe-8706-1ab5338eb5c9/image.png)

<br>

복제 컨트롤러가 생성되면서 설정한 개수(3)의 파드가 함께 생성된다.

```
kubectl get replicationcontroller

kubectl get pods
```

![](https://velog.velcdn.com/images/ramu/post/4ab77937-f6e2-44d6-abe8-94e132dcdebc/image.png)

![](https://velog.velcdn.com/images/ramu/post/d6fff613-a8e6-45e8-ac62-650786e5d199/image.png)

<br>

> **🤷‍♀️ create와 apply의 차이?**
>
> 리소스가 존재하지 않을 경우에는 차이가 없지만 리소스가 이미 존재할 경우 create 명령에서 error가 발생하고, apply 명령은 업데이트가 된다.

<br>
<br>

## ReplicaSet 생성

<br>

### 1) YAML 파일 생성

```
vim replicaset-definition.yaml
```

<br>

### 2) 4개의 필수 요소 작성

-   `apiVersion` : v1이 아닌 `app/v1`으로 작성해야 한다. v1으로 작성하면 에러가 발생한다. (레플리카 셋은 `app/v1` 버전으로 생성 가능하다.)
-   `kind` : `ReplicaSet`
-   `metadata` : 파드의 이름과 label을 설정한다.
-   `spec` : Replica Controller와 유사하게 template을 작성해주어야 한다.
    -   `template` : 복제할 pod의 템플릿이 들어가며, 템플릿 아래에는 `metadata`와 `spec`이 들어간다.
    -   `replicas` : 복제본 수
    -   ✨`selector` : ReplicaSet에는 `selector` 부분을 꼭 작성해주어야 한다.

<br>

**selector**
<br>

`selector` 섹션은 어떤 파드가 ReplicaSet 아래에 있는지 식별하는 역할을 한다.

`selector`의 하위 항목에는 `matchLabels` 속성이 있다.
`matchLabels`는 파드의 `labels`와 매칭되는 정보이다. 이 속성은 지정된 라벨과 같은 라벨을 가진 파드를 연결한다.

<br>

```
apiVersion: app/v1
kind: ReplicaSet
metadata:
  name: myapp-replicaset
  labels:
    app: myapp
    type: front-end
spec:
  template:
    metadata:
      name: myapp-pod
      labels:
        app: myapp
        type: front-end
    spec:
      containers:
      - name: nginx-continaer
        image: nginx
  replicas: 3
  selector:
    matchLabels:
      type: front-end
```

<br>

### 3) 파일 저장

esc를 누른 후 `wq` 커맨드로 파일을 저장한다.

-   w : 쓰기 작업 저장
-   q : 나가기

<br>

### 4) 오브젝트 생성

yaml 파일을 적용하여 ReplicaController를 생성한다.

```
kubectl create -f replicaset-definition.yaml
```

![](https://velog.velcdn.com/images/ramu/post/8628a580-1016-47b2-996c-e95cda304069/image.png)

<br>

레플리카 셋이 생성되면서 설정한 개수(3)의 파드가 함께 생성된다.

```
kubectl get replicaset

kubectl get pods
```

![](https://velog.velcdn.com/images/ramu/post/f7bdc72f-e5c9-4045-92cf-1dc2041576ba/image.png)

![](https://velog.velcdn.com/images/ramu/post/de6870b7-7bdb-433d-8bf8-542e19580cf3/image.png)

<br>
<br>

## Label과 Selector

`labels`와 `selector` 속성은 무엇이고, `labels`를 지정하는 이유는 무엇일까?

<br>

`ReplicaSet`의 역할은 파드를 모니터링하고, 파드가 존재하지 않거나 관리하고 있는 파드에 문제가 생기면 새 파드를 배포함으로써 복제본 수를 유지한다.

그렇다면 `ReplicaSet`은 어떤 파드를 모니터링 해야 할지 알고 있어야 한다. 바로 여기서 파드의 labels가 사용된다.

<br>

> `labels`는 `ReplicaSet`에 대한 필터 역할을 한다.

<br>

### 결론

selector의 `matchLabels`에 기입된 label과 관리할 대상 파드의 `labels`를 일치시켜, 해당 label을 가지고 있는 pod를 대상으로 모니터링하고, 관리할 수 있다.

<br>

## 복제본 수 관리 방법

### 1) YAML 변경

-   **yaml 파일 내용 변경**
    `replicas` 항목 값을 변경해준다. (3 -> 6)

```
apiVersion: app/v1
kind: ReplicaSet
metadata:
  name: myapp-replicaset
  labels:
    app: myapp
    type: front-end
spec:
  template:
    ...
  replicas: 6
  selector:
    matchLabels:
      type: front-end
```

-   **ReplicaSet 업데이트**
    아래 명령어로 `ReplicaSet`의 구성이 변경되어 업데이트 된다.

```
kubectl replace -f replicaset-definition.yaml

kubectl apply -f replicaset-definition.yaml
```

<br>

### 2) YAML 내용을 `scale` 커맨드로 변경

```
kubectl scale --replicas=6 -f replicaset-definition.yaml
```

`scale` 커맨드를 통해 구성 파일 내의 replica의 수를 업데이트 할 수 있다.

<br>

### 3) `scale` 커맨드로 직접 변경

```
kubectl scale --replicas=6 replicaset myapp-replicaset
```

구성 파일 내용은 변경되지 않지만 오브젝트 타입(ReplicaSet)과 오브젝트 이름(myapp-replicaset)을 직접 기입하여 레플리카 수를 변경할 수 있다.

<br>

![](https://velog.velcdn.com/images/ramu/post/f3cdb57e-24df-4fb5-be1a-6e3f4630ee54/image.png)

<br>
<br>

# 💻 Deployment

![](https://velog.velcdn.com/images/ramu/post/f8352e02-f474-4370-98fa-4e73c973eaa6/image.png)

-   배포해야 할 웹 서버가 있고, 웹 서버의 인스턴스가 많이 필요한 상황이다.
-   애플리케이션의 새로운 버전이 나올 때마다 인스턴스를 원활하게 업그레이드 하기를 원한다.
-   롤링 업데이트 : 인스턴스 전체를 한번에 업그레이드 하지 않고 하나씩 업그레이드 하고자 한다.
-   롤백 : 업그레이드 중 오류가 발생하면, 최근 변경사항을 원래의 상태로 되돌려야 할 수 있다.

<br>

이러한 요구사항을 쿠버네티스의 `Deployment`를 사용하여 만족할 수 있다.

![](https://velog.velcdn.com/images/ramu/post/94269245-69b8-4ead-8d85-6616c2a296f8/image.png)

`Deployment`는 원활한 업그레이드를 통해 **<span style="color: #ff08a4">롤링 업데이트 사용, 롤백, 중지, 재개하는 기능</span>**을 제공한다.

<br>

**롤아웃 상태 보기**

```
kubectl rollout status deployment/myapp-deployment

# 결과
Waiting for rollout to finish: 1 out of 3 new replicas have been updated...
```

<br>

**롤아웃 기록 확인하기**

```
kubectl rollout history deployment/myapp-deployment

# 결과
deployments "nginx-deployment"
REVISION    CHANGE-CAUSE
1           kubectl apply --filename=https://k8s.io/examples/controllers/nginx-deployment.yaml
2           kubectl set image deployment/nginx-deployment nginx=nginx:1.16.1
3           kubectl set image deployment/nginx-deployment nginx=nginx:1.161
```

<br>

**이전 단계로 롤백하기**

```
kubectl rollout undo deployment/nginx-deployment

# 결과
deployment.apps/nginx-deployment rolled back
```

<br>

**특정 단계로 롤백하기**

```
kubectl rollout undo deployment/nginx-deployment --to-revision=2

# 결과
deployment.apps/nginx-deployment rolled back
```

<br>

**레플리카 수 늘리기 (확장)**

```
kubectl scale deployment/nginx-deployment --replicas=10

# 결과
deployment.apps/nginx-deployment scaled
```

<br>

## 디플로이 생성

### 1) 구성 파일 작성

텍스트 편집기를 통해 구성 파일을 작성한다.

```
vim deployment-definition.yaml
```

<br>

`kind` 항목만 제외하면 `ReplicaSet`과 작성 방법이 똑같다.

```
apiVersion: app/v1
kind: Deployment
metadata:
  name: myapp-deployment
  labels:
    app: myapp
    type: front-end
spec:
  template:
    metadata:
      name: myapp-pod
      labels:
        app: myapp
        type: front-end
    spec:
      containers:
      - name: nginx-continaer
        image: nginx
  replicas: 3
  selector:
    matchLabels:
      type: front-end
```

<br>

`Deployment`는 `ReplicaSet`보다 더 높은 계층 구조에 위치하며, `Deployment`를 생성하면 `ReplicaSet`이 자동으로 생성된다.

<br>

### 2) 디플로이 생성

```
kubectl create -f deployment-definition.yaml
```

![](https://velog.velcdn.com/images/ramu/post/1897f03b-a84f-4928-8741-4e8a444f8d8d/image.png)

<br>

디플로이를 생성하면 `replicaset`과 `pod`가 함께 생성된다.

```
kubectl get deployments

kubectl get replicaset

kubectl get pods
```

![](https://velog.velcdn.com/images/ramu/post/1d459196-fcb1-4f57-8a63-4d88cd0d1fa5/image.png)

<br>

> **모든 오브젝트를 한번에 보는 커맨드**

```
kubectl get all
```

![](https://velog.velcdn.com/images/ramu/post/f64654aa-86ef-4571-8fd4-32706f9641bd/image.png)

<br><br>

# 💯 Certification Tip!

yaml 파일을 생성하고 편집하는 것이 다소 어려울 수 있다. 또한 시험 중 yaml 파일 복사, 붙여넣기가 불편할 수 있다.

이때 `kubectl run` 커맨드를 사용하면 yaml 파일 템플릿을 생성하는데 도움이 된다.

### Create an NGINX pod

```
kubectl run nginx --image=nginx
```

<br>

### Generate POD Manifest YAML file (-o yaml). Don't create it(--dry-run)

```
kubectl run nginx --image=nginx --dry-run=client -o yaml
```

<br>

### Crate a deployment

```
kubectl create deployment --image=nginx  nginx
```

<br>

### Generate Deployment Yaml file (-o yaml). Don't create it(--dry-run)

```
kubectl create deployment --image=nginx nginx --dry-run=client -o yaml
```

<br>

### Generate Deployment Yaml file (-o yaml). Don't create it(--dry-run) with 4 Replicas (--replicas=4)

-   방법 1

```
kubectl create deployment --image=nginx nginx --dry-run=client -o yaml > nginx-deployment.yaml
```

<br>

커맨드를 입력한 후 `ls`를 입력하면 `nginx-deployment.yaml` 파일이 생성된 것을 확인할 수 있다.

텍스트 편집기로 파일을 열어(`vim nginx-deployment.yaml`) replicas 항목 값을 4로 설정한다.

설정 후 디플로이를 생성한다

```
kubectl create -f nginx-deployment.yaml
```

<br>

-   방법 2

```
kubectl create deployment --image=nginx nginx --replicas=4 --dry-run=client -o yaml > nginx-deployment.yaml
```

k8s의 버전 중 1.19 이상의 버전에서는 `--replicas` 옵션으로 replicas 값을 설정할 수 있다.

<br><br>

# 🔊 Services

쿠버네티스의 `Services`는 내부 구성요소 간, 외부에 있는 애플리케이션과 통신할 수 있게 해주는 역할을 한다.

예를 들어, 애플리케이션이 프론트엔드 그룹, 백엔드 그룹, 데이터 그룹으로 구성되어 있다고 생각해보자!

이러한 파드 그룹을 연결해주는 것이 `Service`이다.
`Service`를 통해 사용자가 애플리케이션의 프론트엔드에 접근할 수 있고, 백엔드와 프론트엔드 파드가 통신할 수 있으며, 외부 데이터 베이스와 연결할 수 있다.

<br>

## External Communication

웹 애플리케이션 파드를 배포할 경우, 외부에서 사용자가 웹페이지에 접속하는 방법은 무엇일까?

기본 설정을 살펴보자!

-   파드가 속해있는 worker node의 IP 주소는 `192.168.1.2`이다.
-   현재 사용자의 노트북 또한 같은 네트워크 상에 있으며, IP 주소는 `192.168.1.10`이다.
-   노드의 내부에 생성되는 파드 IP의 네트워크 주소는 `10.244.0.0`이다.
-   접속하려는 파드의 IP는 `10.244.0.2`이다.

현재 사용자의 IP 주소와 파드의 IP는 다른 네트워크 주소를 가지므로, ping을 보내거나 접속할 수 없다.

이 경우, 쿠버네티스 노드와 SSH 연결을 통해 curl이나 노드 내에서 웹페이지 내용을 확인할 수 있다.

![](https://velog.velcdn.com/images/ramu/post/49605b2f-63cc-46db-8e0d-31dd29c12365/image.png)

그러나 이 방법은 쿠버네티스 노드 내부에서 파드에 접근하는 것이므로, 사용자의 노트북에서 파드에 접속할 수 있는 다른 방법을 선택해야 한다.

따라서 노트북->노드, 노드->파드로 노트북에서 보낸 요청을 파드로 전달해주어야 하는데 이 역할을 하는 것이 쿠버네티스의 **Service**이다.

## Service Type

<br>

쿠버네티스에는 3가지 타입의 `Service`가 존재한다.

### 1) NodePort

노드 포트로 내부의 파드 포트에 접근가능하게 하는 서비스

![](https://velog.velcdn.com/images/ramu/post/cd2e0d2a-0fc8-4012-8a9c-5ae9a55e2970/image.png)

-   `TargetPort` : 실제 접속하려는 파드의 포트, Service가 최종적으로 요청을 전달하는 포트
-   `Port` : 서비스 자체의 포트, 라고 부른다. 서비스는 노드 내의 가상 서버와 같으며, `ClusterIP:Port`로 들어온 요청을 `TargetPort`로 전달한다.
-   `NodePort` : 외부에서 내부 파드에 액세스하기 위해 사용할 포트, 노드포트는 30000~32767 사이의 범위에서 값을 가짐

![](https://velog.velcdn.com/images/ramu/post/758d14cd-bf13-4f6d-83ef-5ceeb1da63a5/image.png)

<br>

### 서비스 생성 방법

#### 1) 구성 파일 작성

```
vim service-definition.yml
```

<br>

**구성파일 내용**

-   apiVersion: v1
-   kind: Service
-   metadata: 이름과 라벨 등 서비스에 대한 정보
-   spec: 서비스 역할을 정의하는 부분
    -   type: 서비스 타입 (`NodePort`, `ClusterIP`, `LoadBalancer`)
    -   ports: 포트 관련 정보, 리스트 형태로 여러 개의 포트 매핑을 가진다.
        -   targetPort : 실제 파드 포트
        -   port : 서비스 오브젝트 포트
        -   nodePort : 노드 포트 (값을 입력하지 않으면 범위 내에서 자동으로 할당됨)
    -   selector : 연결할 파드의 label과 같은 label을 설정하여 매핑

```
apiVersion: v1
kind: Service
metadata:
  name: myapp-service
spec:
  type: NodePort
  ports:
  - targetPort: 80
    port: 80
    nodePort: 30008
  selector:
    app: myapp
    type: front-end
```

<br>

#### 2) 서비스 오브젝트 생성

-   구성 파일을 적용하여 서비스 생성

```
kubectl create -f service-definition.yml
```

![](https://velog.velcdn.com/images/ramu/post/c46344ea-11a2-4aec-9e9d-d5462842d701/image.png)

-   서비스 오브젝트가 생성되었는지 확인

```
kubectl get services
```

![](https://velog.velcdn.com/images/ramu/post/510f7213-d763-4e8e-9ab9-9515022ce0a7/image.png)

-   `curl` 커맨드를 통해 애플리케이션 내용 확인

```
curl http://192.168.1.2:30008
```

![](https://velog.velcdn.com/images/ramu/post/e316d193-9aec-4fa3-93d9-6a6b9f0bf2fc/image.png)

<br>

만약 동일한 애플리케이션이 동작하는 3개의 파드가 있고, 이 파드들이 동일한 label을 가지고 있을 경우에는 `Service`가 랜덤 알고리즘으로 요청을 전달할 파드를 선택하여 로드 밸런싱을 한다.

![](https://velog.velcdn.com/images/ramu/post/be9e2c8d-be3c-44b2-a8fb-0b5b6abc5702/image.png)

<br>

파드가 여러 노드에 분산되어 있는 경우, 서비스가 생성될 때 쿠버네티스가 자동으로 클러스터의 모든 노드에 대상 파드의 포트를 매핑한다.

이때 매핑된 노드 포트들은 모두 동일하다.

어떤 경우든지 파드가 추가되거나 삭제되면 서비스는 자동으로 업데이트 되어 새로운 파드로 요청을 전달해준다.

![](https://velog.velcdn.com/images/ramu/post/f8a4f7f8-9770-459c-af5b-adbd987d44d0/image.png)

<br>

### 2) ClusterIP

`ClusterIP` 서비스는 서로 다른 서비스 간 통신이 가능하도록 클러스터 안에 Virtual IP를 생성한다.

<br>

### 3) LoadBalancer

`loadBalancer` 서비스는 파드들의 부하 분산 기능을 제공한다.
