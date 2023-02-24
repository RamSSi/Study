# Section 5: Application Lifecycle Management

<br>

# Encrypting Secret Data at Rest

<br>

## secret을 생성해보자!

-   secret 생성

```
kubectl create secret generic my-secret --from-literal=key1=supersecret
```

![](https://velog.velcdn.com/images/ramu/post/f2f2ae23-bb02-4a3d-a255-6f32be0af279/image.png)

<br>

-   secret 확인

```
kubectl get secret
```

![](https://velog.velcdn.com/images/ramu/post/a4ca05ad-4cfd-40ee-8b8b-eb35f4979a38/image.png)

<br>

-   secret 더 자세한 정보 확인

```
kubectl describe secret my-secret
```

![](https://velog.velcdn.com/images/ramu/post/6f980b02-76b2-4352-8521-42a29cd79976/image.png)

<br>

-   secret 구성 파일 확인하기

```
kubectl get secret my-secret -o yaml
```

![](https://velog.velcdn.com/images/ramu/post/16f87798-370b-4e28-a6ac-299ef4c23912/image.png)

<br>

-   secret decode

```
echo "key" | base64 --decode
```

![](https://velog.velcdn.com/images/ramu/post/00b32400-d4d9-43ed-a0fd-7839d0020996/image.png)

---

이번 강의에서는 encoding 을 중요하게 다루지는 않는다.

**etcd에 데이터가 어떻게 저장되는지**에 초점을 맞추어 학습해보자!

<br>

## etcd에 데이터가 어떻게 저장될까?

이를 확인하기 전에 etcd 명령을 사용할 수 있어야 한다.

<br>

### `etcdctl` 설치

```
apt-get install etcd-client
```

![](https://velog.velcdn.com/images/ramu/post/2b1d4c76-159b-4e0f-a8df-91c940a5f833/image.png)

<br>

### etcd 컨트롤 플레인 확인

etcd는 pod 형태로 실행되고 있다.

```
kubectl get pods -n kube-system
```

![](https://velog.velcdn.com/images/ramu/post/337ea00b-a5d4-461e-8a40-3b66b9907bc0/image.png)

<br>

### etcd 데이터 내용 확인

앞에서 생성한 secret `my-secret`이 etcd에 어떻게 저장되어 있는지 확인하려고 한다.

<br>

-   확인하는 명령어

이 명령을 실행하기 위해서는 인증키가 필요하다.

`ca.crt`, `server.crt`, `server.key`가 해당 경로에 있는지 확인해야 한다. (kubernetes 클러스터 생성 시 키가 함께 생성된다.)

```
ETCDCTL_API=3 etcdctl \
   --cacert=/etc/kubernetes/pki/etcd/ca.crt   \
   --cert=/etc/kubernetes/pki/etcd/server.crt \
   --key=/etc/kubernetes/pki/etcd/server.key  \
   get /registry/secrets/default/my-secret | hexdump -C
```

![](https://velog.velcdn.com/images/ramu/post/4ead1d06-4344-4095-8710-a9121cd80adb/image.png)

<br>

확인해보면 데이터는 암호화되지 않은 평문 그대로 저장되는 것을 알 수 있다.

즉, etcd에 접근하는 사용자는 누구든지 모든 secret을 볼 수 있기 때문에 보안 측면의 문제가 발생할 수 있다.

![](https://velog.velcdn.com/images/ramu/post/acad9e49-7331-4c2f-8fa1-6f688c4ec607/image.png)

<br>

따라서 이 문제를 해결해야 한다!!!

<br>

먼저 암호화 주소가 이미 활성화 되어 있는지 여부를 확인해야 한다.

이것은 kube-apiserver의 옵션인 `--encryption-provider-config`를 통해 수행된다.

실행되고 있는 프로세스에서 kube-apiserver에 대한 내용을 확인해보자.

<br><br>

## kube-apiserver 옵션 확인하기

<br>

### 1) 커맨드로 확인

<br>

```
ps -aux | grep kube-api
```

![](https://velog.velcdn.com/images/ramu/post/cbdde509-c873-441f-b4c5-06db915b0f49/image.png)

<br>

이제 옵션들 중 `--encryption-provider-config` 옵션이 활성화 되어 있는지 확인해보자.

<br>

-   `--encryption-provider-config` 옵션 확인하기

```
ps -aux | grep kube-api | grep "--encryption-provider-config"
```

![](https://velog.velcdn.com/images/ramu/post/24a4cfed-c7d1-40c0-9fc5-40b20e1cf458/image.png)

<br>

커맨드 결과를 반환하지 않는다는 것은 옵션이 활성화되지 않았다는 의미이다.

<br><br>

### 2) 정의 파일로 확인

kube-apiserver의 정의 파일 내용으로 활성화된 옵션을 확인할 수 있다.

<br>

-   kube-apiserver 정의 파일 찾기

```
ls /etc/kubernetes/manifests/
```

![](https://velog.velcdn.com/images/ramu/post/38130d76-2e11-4bc6-b431-4e09d475d03a/image.png)

<br>

-   내용 확인하기

```
vi /etc/kubernetes/manifests/kube-apiserver.yaml
```

...
![](https://velog.velcdn.com/images/ramu/post/fa1ebee5-a04b-43bb-ac52-22ff8c2df378/image.png)
...

`--encryption-provider-config` 옵션이 존재하지 않는 것을 확인할 수 있다.

<br><br>

## `EncryptionConfiguration` 생성
