# Section 2: Core Concepts

<br>

# 👮‍♂️ Kubelet

kubelet은 배의 선장과 같다!
kubelet은 워커 노드를 마스터 노드와 연결시켜주는 연락망 역할을 하며, 워커 노드에 컨테이너를 생성하거나 제거한다.

<br>

## <div style="background-color: #E5CCFF"> 역할</div>

<br>

![](https://velog.velcdn.com/images/ramu/post/0698b6d5-905e-4bf7-9824-904f2f53e505/image.png)

1. kubelet은 자신이 속해있는 워커 노드를 쿠버네티스 클러스터에 등록한다.

2. 노드에 컨테이너나 파드를 load하라는 지시를 받으면 api 서버에게 컨테이너 런타임 엔진을 요청한다.
   예. docker, containerd

3. 파드와 컨테이너의 상태를 계속 모니터링하며 그 결과를 api 서버에 보고한다.

<br>

## <div style="background-color: #E5CCFF"> 설치 방법</div>

<br>

클러스터를 배포하기 위해 kubeadm을 사용해도 다른 구성요소와는 다르게 `kubelet`은 자동으로 생성되지 않는다.

<br>

> 반드시 워커 노드에 수동으로 설치해야 한다.

<br>

```
# 다운로드

wget https://storage.googleapis.com/kubernetes-release/release/v1.13.0/bin/linux/amd64/kubelet


# kubelet.service

ExecStart=/usr/local/bin/kubelet \\
--config=/var/lib/kubelet/kubelet-config.yaml \\
--container-runtime=remote \\
--container-runtime-endpoint=unix:///var/run/containerd/containerd.sock \\
--image-pull-progress-deadline=2m \\
--kubeconfig=/var/lib/kubelet/kubeconfig \\
--network-plugin=cni \\
--register-node=true \\
--v=2
```

```
# kubelet 옵션 확인

ps -aux | grep kubelet


# 결과
root 2095 1.8 2.4 960676 98788 ? Ssl 02:32 0:36 /usr/bin/kubelet --bootstrapkubeconfig=/etc/kubernetes/bootstrap-kubelet.conf --kubeconfig=/etc/kubernetes/kubelet.conf --
config=/var/lib/kubelet/config.yaml --cgroup-driver=cgroupfs --cni-bin-dir=/opt/cni/bin --cniconf-dir=/etc/cni/net.d --network-plugin=cni
```

<br><br>

# 📣 kube-proxy

<br>
쿠버네티스 클러스터에서는 파드 네트워킹 솔루션을 배포함으로써 파드 간 통신이 가능하다.

<br>

![](https://velog.velcdn.com/images/ramu/post/4d462470-148e-4eeb-9b72-503ee5c9e62a/image.png)

<br>

파드 통신에는 IP가 사용되지만 파드의 IP는 달라질 수 있기 때문에 `service`를 이용하여 통신한다.

서비스도 IP를 할당받으며, 파드는 서비스의 이름이나 IP로 메시지를 보낸다. 서비스는 트래픽을 연결된 파드로 보낸다.

<br>

## <div style="background-color: #E5CCFF"> 서비스는 무엇인가?</div>

서비스는 파드와 같은 컨테이너가 아닌 쿠버네티스 메모리에만 존재하는 가상 구성 요소이다.

그런데 서비스가 어떻게 파드 간 통신을 가능하게 할까?
어떻게 파드 네트워크에 참여하여 다른 노드에 있는 파드와의 통신을 가능하게 할까?

<br>

## <div style="background-color: #E5CCFF"> kube-proxy가 필요하다!</div>

kube-proxy는 쿠버네티스 클러스터의 각 노드에서 실행되는 프로세스로, 파드와 서비스를 연결하는 역할을 수행한다.

kube-proxy는 서비스가 새로 생성될 때마다 각 노드에 <u>**적절한 규칙**</u>을 만들어 그 서비스로 트래픽을 전달한다.

<br>

### 무슨 규칙인가?

iptables을 사용하여 규칙을 만든다.
kube-proxy는 클러스터의 각 노드에서 서비스 IP(10.96.0.12)로 향하는 트래픽을 전달하기 위해 iptable 규칙을 생성한다.

<br>

> iptables 규칙은 실제 파드의 IP인 10.32.0.15를 서비스 IP 10.96.0.12로 변경하여 서비스 IP로 들어온 트래픽을 실제 파드의 IP로 전달하는 것이다.

<br>

## <div style="background-color: #E5CCFF"> 설치 방법</div>

```
# 다운로드

wget https://storage.googleapis.com/kubernetes-release/release/v1.13.0/bin/linux/amd64/kube-proxy
```

<br>

kubeadm은 각 노드에 kube-proxy를 pod 형태로 배포한다.

```
# kubeadm으로 자동 설치된 kube-proxy 확인

kubectl get pods -n kube-system
```

<br>

실제로는 deamonset으로 배포되어 있음을 확인할 수 있다.

```
# 데몬셋 형태의 kube-proxy 확인
kubectl get daemonset -n kube-system
```

<br><br>

# 🔲 Pod

몇 가지 가정을 해보자!

>

1. 애플리케이션이 이미 개발되어 도커 이미지로 빌드되어 도커 허브와 같은 레포지토리에서 사용 가능하다.
    >
2. 쿠버네티스 클러스터가 이미 셋업되어 작동하고 있다.
    >
3. 모든 서비스가 실행 중인 상태이다.

쿠버네티스의 궁극적인 목표는 클러스터에서 워커 노드에 컨테이너 형태로 애플리케이션을 배포하는 것이다.

그러나 쿠버네티스는 컨테이너를 워커노드에 직접 배포하지 않고 `pod`라는 쿠버네티스 오브젝트 내에 캡슐화된 상태로 배포한다.

<br>

## <div style="background-color: #E5CCFF">특징</div>

-   **파드는 애플리케이션의 단일 인스턴스로, 쿠버네티스에서 만들 수 있는 가장 작은 오브젝트이다.**

![](https://velog.velcdn.com/images/ramu/post/662ffc13-305d-4ce3-8cfd-4654fa2a4963/image.png)

-   **파드는 일반적으로 컨테이너와 1:1 관계를 가진다.**
    따라서 파드 내에서 실행중인 애플리케이션 컨테이너의 load를 분산하기 위해서는 새 파드를 생성하여 두 개의 파드로 애플리케이션을 실행하면 된다.
    또한 애플리케이션의 규모를 줄이려면 파드를 삭제하면 된다.

<br>

![](https://velog.velcdn.com/images/ramu/post/387a1c51-0d1b-45af-8bc2-c050104b426e/image.png)

-   **파드와 컨테이너가 1:1 관계를 가진다고 했지만 한 파드는 다른 종류의 컨테이너를 여러 개 가질 수 있다.**
    보통 컨테이너 실행에 필요한 기능을 제공하는 헬퍼 컨테이너와 애플리케이션 컨테이너가 동일한 파드에 속해있다.
    두 컨테이너는 동일한 네트워크 스페이스를 공유하므로 직접 소통할 수 있고, 동일한 저장 공간을 공유할 수 있다.

<br>

## <div style="background-color: #E5CCFF">배포 방법</div>

### `kubectl run`

<br>

![](https://velog.velcdn.com/images/ramu/post/d5e325a4-c635-4b79-a607-ef9d0b972089/image.png)

파드를 생성하여 docker 이미지로 컨테이너를 배포하는 것

`kubectl run nginx --image nginx`

파드를 생성하고, nginx 컨테이너 인스턴스를 배포한다.
이때 docker hub(또는 다른 레포지토리)에서 이미지를 가져오기 위해 `--image` 플래그를 통해 이미지 이름을 명시해야 한다.

<br><br>

# 📜 Pods with YAML

쿠버네티스는 yaml 파일을 파드, 레플리카셋, 디플로이, 서비스 등의 개체 생성을 위한 입력으로 사용한다.

`kubectl run` 대신 YAML 기반의 구성 파일을 사용하여 파드를 생성할 수도 있다.

## <div style="background-color: #E5CCFF"> Kubernetes definition file</div>

![](https://velog.velcdn.com/images/ramu/post/2ad5023f-4974-4911-b2a1-458c4e1ed6e5/image.png)

최상위 레벨에는 항상 아래 4가지가 포함된다.

>

-   apiVersion
-   kind
-   metadata
-   spec

이 4가지는 최상위 레벨 속성들로, 구성 파일에 반드시 필요하기 때문에 필수로 입력해야 한다.

<br>

### ✨ apiVersion

`apiVersion`은 오브젝트를 생성하는데 사용하는 kubernetes API 버전을 뜻한다.

생성할 오브젝트에 따라 알맞은 api 버전을 사용해야 한다.

![](https://velog.velcdn.com/images/ramu/post/d97e0462-fd87-474c-9abb-2725f8c4592b/image.png)

현재는 pod를 생성하고 있으므로 apiVersion은 `v1`으로 설정한다.

<br>

### ✨ kind

`kind`는 생성할 오브젝트 타입을 의미한다.
현재 생성할 오브젝트 타입은 `Pod`이므로 그렇게 적어주면 된다.

<br>

### ✨ metadata

`metadata`는 name, labels 등과 같이 오브젝트에 대한 데이터이다.

-   metadata에는 dictionary 값(key-value)이 들어간다.
-   metadata 내의 값들은 하위 항목으로 표현하기 위해 들여쓰기를 해야 한다.

-   공백은 2칸을 권장하며, `name`과 `labels`은 형제 데이터이므로 동일한 공백으로 들여쓰기를 해주어야 한다.

![](https://velog.velcdn.com/images/ramu/post/ac0bc511-c83d-4e1e-bbbc-1ab9a434cb29/image.png)

**`name`** : pod의 이름을 지정할 string 값
**`lables`** : 오브젝트를 식별하기 위한 값으로, key-value 데이터를 하위 항목으로 한다.

<br>

### ✨ spec

우리가 생성할 오브젝트 타입은 pod이고, 이름은 'myapp-pod'이지만 컨테이너에 사용할 이미지를 지정하지 않았다.

`spec`은 생설할 오브젝트에 따라 쿠버네티스에 제공할 오브젝트 관련 추가 정보를 작성하는 곳이다.

-   추가정보는 오브젝트마다 다르기 때문에 적합한 포맷을 알아야 한다.

-   spec 또한 dictionary 데이터를 하위 항목으로 가지며, 파드는 여러 개의 컨테이너를 가질 수 있기 때문에 `containers` 속성은 리스트 형태이다.

-   생성할 파드는 하나의 컨테이너를 가진 파드이므로, 파드 내 컨테이너의 이름 1개와 사용할 이미지 정보를 작성해야 한다.

<br>

![](https://velog.velcdn.com/images/ramu/post/27dde8e5-7464-451a-a719-3649c4706f3d/image.png)

이렇게 yaml 언어로 작성한 구성 파일로 파드를 생성해보자!

```
kubectl create -f pod-definition.yml
```

<br>

# 📜 Demo - Pods with YAML

YAML 파일은 모든 텍스트 편집기에서 만들 수 있다.
메모장, vim 등을 사용할 수 있지만 YAML 언어를 지원하는 편집기를 사용하는 것이 가독성, 이해도 측면에서 더 좋을 것이다!

따라서 Windows 환경에서 메모장 대신 Notepad++를 사용하고, Linux 환경에서는 vim을 사용하는 것이 나을 것입니다 😁👍

<br>

## 1. yaml 파일 생성하기

![](https://velog.velcdn.com/images/ramu/post/caf3de7e-5742-41ad-916c-ed02aa02e0c5/image.png)

`vim pod.yaml`로 'pod'라는 이름의 yaml 파일을 생성한다.

<br>

## 2. 파일 작성

![](https://velog.velcdn.com/images/ramu/post/21b0c0ee-2266-4540-b322-27fcbf297231/image.png)

생성할 오브젝트에 대한 구성 파일을 작성한다.

## 3. 파일 확인

![](https://velog.velcdn.com/images/ramu/post/2d6fcf98-479c-483a-b514-29f7f71f8d53/image.png)

`cat` 명령어를 통해 작성한 파일 내용을 확인할 수 있다.

## 4. 생성

```
kubectl apply -gf pod.yaml
```

![](https://velog.velcdn.com/images/ramu/post/b57ac0c0-648b-4ed4-97c5-94f739923b98/image.png)

작성한 파일에 대한 오브젝트가 생성된다.

## 5. 생성여부 확인

```
kubectl get pods

kubectl get po
```

![](https://velog.velcdn.com/images/ramu/post/441bb1b8-d1df-4f2a-8bfe-b3631f2f7658/image.png)

## 6. 파드 정보 확인

```
kubectl describe pod [파드 이름]
```

파드의 이름으로 해당 파드의 자세한 정보를 확인할 수 있다.

![](https://velog.velcdn.com/images/ramu/post/7b70dbb4-38a4-4c70-bffb-4b49d15cd599/image.png)

**[결과]**
![](https://velog.velcdn.com/images/ramu/post/a5535479-ee14-46ef-9f0d-e4b0f3dc8bec/image.png)
