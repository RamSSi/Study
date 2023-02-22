3일차의 `Service` Part와 이어지는 내용입니다.

<br>

# Services

<br>

## <div style="background-color: #af6ef1">ClusterIP</div>

<br>

웹 애플리케이션에는 프론트엔드 웹서버 파드, 백엔드 웹서버 파드, 데이터베이스 서버 파드(Redis, MySQL, ...) 세트들이 존재할 수 있다.
프론트엔드 서버는 백엔드 서버와 통신해야 하고, 백엔드 서버는 DB 서버와 통신해야 한다.

<br>

![](https://velog.velcdn.com/images/ramu/post/7c3467dd-7696-420e-8aaf-f18ef601f470/image.png)

<br>

노드 내 모든 파드에는 IP 주소가 할당된다. 그러나 이 IP 주소는 동적인 주소이고, 파드가 삭제되고 생성되며 새로운 IP 주소가 할당될 것이다.

<br>

> **🎈 부가 설명**
> 쿠버네티스의 파드 오브젝트는 생성과 삭제가 자유롭다. 파드의 IP 주소는 파드가 생성될 때마다 새로 부여되기 때문에 IP 주소로 통신하는 경우 통신 성공을 보장할 수 없다.

<br><br>

### <div style="color: #f379fe"> 🤷‍♂️ 애플리케이션 계층 간 연결을 확립하는 올바른 방법은 무엇일까?</div>

<br>

만약 프론트엔드 서버 파드가 백엔드 서버에 요청을 전달해야 한다면 3개의 백엔드 파드 중 어떤 파드에 요청을 전달해야 할까?

이런 상황에서 쿠버네티스 서비스를 통해 백엔드 파드를 하나로 묶고 하나의 인터페이스(Virtual IP)를 통해 파드 세트에 접속할 수 있다. 요청은 무작위로 한 파드에 전달된다.

`Service`를 통해 쿠버네티스 클러스터에 MSA 구조의 애플리케이션을 쉽고 효과적으로 배포할 수 있다. 또한 다른 계층에 영향을 주지 않고 파드의 개수를 조정하거나 이동시킬 수 있다.

<br>

### <div style="color: #f379fe">ClusterIP 서비스 생성 방법</div>

<br>

> **🎈 부가 설명**

-   Service 타입을 지정하지 않으면 기본적으로 ClusterIP로 설정된다.
-   이 타입의 서비스는 클러스터 내부에 존재하는 파드 간 통신만 지원한다. (외부에서 접근 불가)

<br>

**1) 구성 파일 생성**

```
vim service-definition.yml
```

<br>

**2) 필수 항목 작성**

```
apiVersion: v1
kind: Service
metadata:
  name: back-end
spec:
  type: ClusterIP
  ports:
  - targetPort: 80
    port: 80
  selector:
    app: myapp
    type: back-end
```

-   `targetPort` : 대상 포트
-   `port` : 서비스가 노출하는 포트

<br>

**3) 서비스 생성**

```
kubectl create -f service-definition.yml
```

![](https://velog.velcdn.com/images/ramu/post/3eaa20b6-1194-442b-a862-17b7f8f1c2e7/image.png)

<br>

**서비스 생성 확인**

```
kubectl get services
```

![](https://velog.velcdn.com/images/ramu/post/12ec6907-de78-4023-a470-0da068b25984/image.png)

<br><br>

## <div style="background-color: #af6ef1">LoadBalancer</div>

<br>

앞서 이야기 했던 `NodePort` 타입 서비스는 외부에서 파드에 접근하기 위해 Node의 특정 Port와 파드에 매핑하는 방식의 서비스이다.

노드 포트 서비스는 노드 포트에 들어오는 트래픽을 각각의 파드로 라우팅 해준다.
그러나 결국에는 사용자가 단일 url로 프로그램에 접속할 수 있도록 해야 한다.

이를 위해서는 새 VM을 생성하여 로드밸런서를 설치하는 방법이 있다.

또 다른 방법으로는 AWS, GCP, Azure 등 클라우드 플랫폼에서 제공하는 `LoadBalancer` 서비스를 사용하는 것이다.
`LoadBalancer` 타입 서비스는 노드 포트 서비스의 부하분산을 돕는 서비스이다.

<br>

### <div style="color: #f379fe">`LoadBalancer` 사용 방법</div>

<br>

서비스 타입에 `LoadBalacner`를 기입해주면 된다.

```
apiVersion: v1
kind: Service
metadata:
  name: myapp-service
spec:
  type: LoadBalancer
  ports:
  - targetPort: 80
    port: 80
    nodePort: 30008
```

<br>

`LoadBalacer` 타입 서비스는 AWS, GCP, Azure에서는 지원하지만, VM에서 사용할 경우 NodePort 와 같은 서비스를 지원한다. 부하 분산 기능은 수행하지 않는다.

<br><br>

# Namespace

<br>

우리가 만든 모든 오브젝트는 네임스페이스에 생성되고 있다.

<br>

**1) `Default`**

지금까지 네임스페이스를 지정하지 않았지만 이 경우에는 쿠버네티스가 자동으로 생성한 `Default` 네임스페이스를 사용하게 된다.

**2) `kube-sytstem`**
쿠버네티스는 내부에 네트워킹 솔루션이나 DNS 서비스에 요구되는 시스템 파드와 서비스들을 생성한다.

사용자가 이 오브젝트들을 삭제하거나 수정하는 것을 막기 위해, 클러스터 생성 시 `kube-system`이라는 이름의 네임스페이스를 생성해 파드와 서비스를 보호한다.

**3) `kube-public`**
모든 사용자가 사용할 수 있는 자원을 구분하는 영역이다.

<br>

소규모 시스템이라면 네임스페이스를 신경쓰지 않아도 된다.
하지만 기업에서 사용하는 클러스터 또는 상업용 클러스터의 경우에는 네임스페이스 사용을 고려해야 한다.

<br>

## <div style="background-color: #af6ef1">Namespace 기능</div>

<br>

### 1) Isolation

예를 들어 개발 및 프로덕션 환경에서 동일한 클러스터를 사용하려는 경우에는 리소스 격리를 위해 각 환경에 대한 고유 네임스페이스를 생성할 수 있다.

이렇게 하면 개발 환경에서 작업하는 동안 프로덕션 리소스를 실수로 수정할 일은 없다.

### 2) Resource Limits

각 네임스페이스 별로 역할과 정책을 정의할 수 있다.

사용자마다 네임스페이스 별 역할을 지정할 수 있고, 네임스페이스마다 리소스 할당량을 지정할 수 있다.

네임스페이스는 리소스의 일정량을 보장받고, 허용된 한도 이상을 사용하지 않는다.

### 3) DNS

같은 네임스페이스 내에 존재하는 리소스 간에는 이름으로 통신할 수 있다.

만약 파드가 다른 네임스페이스의 서비스와 통신하려면 서비스 이름에 네임스페이스 이름을 추가하여 접근할 수 있다.

<br>

![](https://velog.velcdn.com/images/ramu/post/86e5ff23-67c3-4a13-a068-f474089c5e79/image.png)

이름으로 통신이 가능한 이유는 서비스가 생성될 때 DNS 항목이 자동으로 추가되기 때문이다.

현재 접근하려는 db 서비스의 전체 이름 중 끝부분을 확인해보면 `.cluster.local`이라고 되어 있다. 이는 클러스터의 기본 도메인 이름이다.

서비스 이름 다음에 네임스페이스 이름이 나오고, 그 뒤 하위 도메인 `svc`는 서비스 타입을 나타낸다.

<br><br>

## <div style="background-color: #af6ef1">Namespace 운영</div>

<br>

### <div style="color: #f379fe">`kubectl get`</div>

<br>

```
kubectl get pods
```

![](https://velog.velcdn.com/images/ramu/post/efd48cd9-5507-458a-bb61-844ebc4e6c91/image.png)

위의 명령은 모든 파드를 출력하는 명령이지만 `Default` 네임스페이스 내에 있는 파드만 출력한다.

다른 네임스페이스에 있는 파드를 확인하고 싶다면 `--namespace` 또는 `-n` 옵션을 사용하면 된다.

<br>

```
kubectl get pods --namespace kube-system
```

![](https://velog.velcdn.com/images/ramu/post/ad0dc057-029a-407b-ad3b-409e20a9b5c4/image.png)

<br>

### <div style="color: #f379fe">`kubectl create`</div>

<br>

![](https://velog.velcdn.com/images/ramu/post/3c46c38a-b7c8-4f44-a7fd-7eea4c53f37a/image.png)

파드를 생성하는 구성 파일이다.
이 파드를 `kubectl create` 명령으로 생성하는 경우 `Default` 네임스페이스에 생성될 것이다.
네임스페이스를 지정하여 생성하려는 경우에도 `--namespace` 옵션을 사용하면 된다.

<br>

```
kubectl create -f pod-definition.yml --namespace=dev
```

![](https://velog.velcdn.com/images/ramu/post/9f639bd8-64fe-490d-a86d-134937f9447e/image.png)

<br>

![](https://velog.velcdn.com/images/ramu/post/421f8642-6e66-4b84-9b5a-69214d49d498/image.png)

<br>

> `--namespace` 옵션을 사용하지 않고, 구성 파일의 `metadata` 하위 항목에 `namespace`를 지정할 수 있다.

![](https://velog.velcdn.com/images/ramu/post/724bd444-4ec2-4cc5-ab15-2d335a785fde/image.png)

<br><br>

## <div style="background-color: #af6ef1">Namespace 생성</div>

<br>

### <div style="color: #f379fe">방법 1) 구성 파일</div>

<br>

```
apiVersion: v1
kind: Namespace
metadata:
  name: dev
```

![](https://velog.velcdn.com/images/ramu/post/a460fa0f-3864-4d15-b1b1-460c777ac5bf/image.png)

<br>

```
kubectl create -f namespace-dev.yml
```

![](https://velog.velcdn.com/images/ramu/post/333cbee2-d01d-4300-a699-45bee4b7c799/image.png)

<br>

### <div style="color: #f379fe">방법 2) `kubectl create`</div>

<br>

```
kubectl create namespace dev
```

![](https://velog.velcdn.com/images/ramu/post/37851a39-648f-4211-8dd5-1113defa6038/image.png)

<br>

## <div style="background-color: #af6ef1">기본 네임스페이스 변경</div>

<br>

우리가 더이상 `Default` 네임스페이스를 사용하고 싶지 않다면 쿠버네티스가 기본으로 생성, 설정한 네임스페이스를 변경할 수 있다.

![](https://velog.velcdn.com/images/ramu/post/9da8a74b-fbf0-4b8e-a315-08c0c73026b6/image.png)

<br><br>

만약 `dev` 네임스페이스를 기본으로 설정하고 싶다면

```
kubectl config set-context $(kubectl config current-context) --namespace=dev
```

<br>

명령으로 기본 네임스페이스를 `dev`로 변경할 수 있다.

![](https://velog.velcdn.com/images/ramu/post/833e835a-93eb-470c-8123-bf35e8e0f22a/image.png)

<br><br>

## <div style="background-color: #af6ef1">모든 네임스페이스 리소스 출력</div>

<br>

-   pod 출력

```
kubectl get pods --all-namespaces
```

-   service 출력

```
kubectl get services --all-namespaces
```

<br>

## <div style="background-color: #af6ef1">Resource Quota (리소스 제한)</div>

<br>

![](https://velog.velcdn.com/images/ramu/post/7f599227-51af-4db0-908e-78dd238a7d0d/image.png)

네임스페이스 별로 리소스 할당량을 지정하려면 `ResourceQuota`를 생성하면 된다.

<br>

구성파일을 작성하여,

```
apiVersion: v1
kind: ResourceQuota
metadata:
  name: compute-quota
  namespace: dev
spec:
  hard:
    pods: "10"
    requests.cpu: "4"
    requests.memory: 5Gi
    limits.cpu: "10"
    limits.memory: 10Gi
```

<br>

오브젝트를 생성한다.

```
kubectl create -f compute-quota.yaml
```

![](https://velog.velcdn.com/images/ramu/post/1468da90-e1af-431e-b1a5-b7a00a512f38/image.png)

<br><br>

# Imperative vs Declarative

지금까지 쿠버네티스 오브젝트를 생성, 관리하는 다양한 방법을 알아보았다.

코드로 인프라를 관리하는 방법은 **명령적 접근 방식(imperative)**과 **선언적 접근 방식(declarative)**으로 나뉜다.

<br>

## <div style="background-color: #af6ef1">접근법</div>

비교를 통해 이해해보자!

### <div style="color: #f379fe">명령적 접근 방식 (Imperative)</div>

<br>

![](https://velog.velcdn.com/images/ramu/post/feec6c1b-c1c8-41df-a740-7ecdf1785e8e/image.png)

예전에는 택시 기사님께 목적지까지 가는 방법을 단계별로 알려야 했다.
즉, "B 거리에서 우회전, C 거리에서 좌회전, 다시 D 거리에서 우회전" 이라고 알려야 했다.

<br>

### <div style="color: #f379fe">선언적 접근 방식 (Declarative)</div>

<br>

![](https://velog.velcdn.com/images/ramu/post/1613e07e-460f-41da-9b00-f08830862524/image.png)

반면에 요즘에는 택시 기사님께 "~로 가주세요." 한 마디만 하면 택시 기사님이 알아서 목적지까지 데려다준다.

<br>

> 명령적 접근 방식은 어떻게(how) 해야할지에 대해 명시하는 것이고, 선언적 접근 방식은 무엇(What)을 해야할 지 명시하는 것이다.

<br><br>

## <div style="background-color: #af6ef1">IaC에서의 접근법</div>

<br>

### <div style="color: #f379fe">명령적 접근 방식 (Imperative)</div>

<br>

![](https://velog.velcdn.com/images/ramu/post/864df1d5-ad8d-4e8e-a707-e21a83914d60/image.png)

무엇이 요구되는지, 어떻게 작업을 수행해야 하는지에 대한 내용이다.

<br>

### <div style="color: #f379fe">선언적 접근 방식 (Declarative)</div>

<br>

![](https://velog.velcdn.com/images/ramu/post/f23b9a3a-e03a-4447-a1ce-a14f544e2696/image.png)

요구사항이 선언되어 있다.

<br>

인프라에 요구되는 기능을 갖추기 위한 모든 작업은 시스템이나 소프트웨어가 하는 일이므로, 우리는 직접 상세하게 설명하지 않아도 된다.

요구사항을 선언하기만 하면 시스템이 알아서 다 해줄 것이기 때문이다! 😎

<br>
<br>

## <div style="background-color: #af6ef1">쿠버네티스에서의 접근법</div>

<br>

### <div style="color: #f379fe">명령적 접근 방식 (Imperative)</div>

<br>

쿠버네티스의 imperative 방식은 `kubectl` 커맨드를 사용하는 것이다.

오브젝트를 생성, 수정, 삭제하기 위해 우리의 요구 사항을 쿠버네티스에게 정확히 어떻게 적용해야 하는지 명령한다.

<br>

#### Create Objects

-   `kubectl run --image=nginx nginx`
-   `kubectl create depoyment --image=nginx nginx`
-   `kubectl expose deployment nginx --port 80`

#### Update Objects

-   `kubectl edit deployment nginx`
-   `kubectl scale deployment nginx --replicas=5`
-   `kubectl set image deployment nginx nginx=nginx:1.18`

<br>

**장점**

생성(Create)과 수정(Update)에 관한 명령은 yaml 파일 없이 오브젝트 생성과 수정을 빠르게 할 수 있게 해준다.

**단점**

그러나 기능에 한계가 있기 때문에 추가적인 옵션이 많다면 명령이 길고 복잡해진다.

또한 이 명령은 한 번 실행하는 일회성 명령이기 때문에 사용자의 세션 히스토리에서만 다시 볼 수 있고, 어떻게 생성되었는지 알아내기 힘들다.

<br>

#### Manifest

따라서 매니페스트(오브젝트 정의 파일, 구성 파일이라고 불리는)를 사용할 수 있다. 파일 형태이기 때문에 git과 같은 레포지토리에 저장할 수 있다.

매니페스트에 yaml 포맷으로 오브젝트의 정보를 입력하고, `kubectl` 생성 명령으로 생성할 수 있다.

또한 오브젝트의 상태를 수정하려면 매니페스트의 내용을 수정한 후 `kubectl replace` 명령으로 수정된 내용을 적용하면 된다.

-   `kubectl create -f nginx.yml`
-   `kubectl replace -f nginx.yml`
-   `kubectl delete -f nginx.yml`

<br>

![](https://velog.velcdn.com/images/ramu/post/9c5ce08a-7847-452b-8397-40ba4b15ce35/image.png)

> 명령정 접근은 관리자에게 매우 까다로운 접근법이다.
> 항상 현재 환경에 대해 인지하고 변경하기 전에 확인해야 하기 때문이다.

<br>
<br>

### <div style="color: #f379fe">선언적 접근 방식 (Declarative)</div>

<br>

쿠버네티스의 declarative 방식은 클러스터에 애플리케이션 및 서비스의 상태를 정의하는 파일을 생성하여 `kubectl apply` 커맨드로 쿠버네티스에게 선언하는 것이다.

<br>

#### Create Objects

-   `kubectl apply -f nginx.yaml`
-   `kubectl apply -f /path/to/config-files`

<br>

#### Update Objects

-   `kubectl apply -f nginx.yaml`

<br>

`kubectl apply` 명령을 사용할 경우, 파일에 정의된 오브젝트를 생성, 수정할 수 있다.

오브젝트 구성 파일이 여러 개라면 파일이 아닌 디렉터리 경로를 지정하여 한번에 생성할 수 있다.

오브젝트를 수정할 때에도 구성 파일을 수정한 후 다시 `kubectl apply` 명령을 실행하면 이미 존재하는 오브젝트를 업데이트 한다. 오브젝트의 존재를 알고 있기 때문에 오류가 발생하지 않는다.

<br><br>

## 💯 시험 팁!

시험에서는 **명령적 접근**을 사용하면 시간을 절약할 수 있다.

주어진 이미지로 파드나 디플로이를 생성해야 하는 경우, 존재하는 오브젝트의 속성을 수정해야 하는 경우 명령적 접근으로 빠르게 풀 수 있다.

복잡한 요구사항을 만족해야 하는 경우에는 매니페스트를 사용하는 것이 더 좋을 수 있다.

![](https://velog.velcdn.com/images/ramu/post/cd09522c-c5a2-4240-9cea-bcf3ec7a0618/image.png)
