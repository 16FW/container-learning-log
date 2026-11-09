# 04. Expose your app publicly

- 구분: 과제 (영상 촬영)
- 공식 튜토리얼: https://kubernetes.io/docs/tutorials/kubernetes-basics/expose/expose-intro/
- 실습 날짜: 2026-11-09

## 1. 목표

- 쿠버네티스의 Service가 무엇인지 이해한다.
- Label과 Selector가 Service와 어떻게 연결되는지 이해한다.
- 애플리케이션을 클러스터 밖으로 노출한다.

## 2. 개념 정리 (내 말로)

### 왜 Service가 필요한가

- Pod는 수명이 있다. Node가 죽으면 그 위의 Pod도 사라지고, ReplicaSet이 새 Pod를 만들어 원하는 상태로 되돌리며 이때 새 Pod는 새 IP를 받는다.
- 모든 Pod는 고유한 IP를 갖지만, 그 IP는 **Service 없이는 클러스터 밖에 노출되지 않는다.** (02, 03번에서 `kubectl proxy`를 쓴 이유)
- **Service**는 Pod들의 논리적 집합과 접근 정책을 정의하는 추상화 계층이다. 외부 트래픽 노출, 로드 밸런싱, 서비스 디스커버리를 제공하고, 의존하는 Pod 사이를 느슨하게 연결한다.

### Service 타입

| 타입 | 설명 |
|---|---|
| ClusterIP (기본) | 클러스터 내부 IP로만 노출. 클러스터 안에서만 접근 가능 |
| NodePort | 선택된 각 Node의 같은 포트로 노출. `NodeIP:NodePort`로 클러스터 밖에서 접근 가능. ClusterIP의 상위 집합 |
| LoadBalancer | 현재 클라우드(지원 시)에 외부 로드 밸런서를 만들고 고정 외부 IP 할당. NodePort의 상위 집합 |
| ExternalName | `externalName` 필드의 값으로 매핑하는 CNAME 레코드를 반환. 프록시는 설정되지 않음 |

### Label과 Selector

- **Label**은 객체에 붙이는 key/value 쌍이다. 개발, 테스트, 운영 구분이나 버전 태그, 분류 등에 쓰고 만들 때나 나중에 붙일 수 있으며 언제든 수정할 수 있다.
- **Service는 label selector로 자기가 담당할 Pod를 찾는다.** Deployment가 Pod에 자동으로 붙인 `app=kubernetes-bootcamp` label과 Service의 Selector가 같아서 서로 연결된다.
- 이 연결의 결과가 Service의 **Endpoints**(트래픽을 보낼 Pod의 `IP:포트`)다.

### Service와 Deployment의 관계

- Service를 삭제해도 Deployment와 Pod는 그대로 남는다. 외부 노출만 사라진다.
- 앱을 완전히 종료하려면 Deployment까지 삭제해야 한다.

## 3. 실습 로그

> 명령어 → 출력 → 한 줄 설명 순서로 기록

### 3-0. 사전 확인

```text
root@controlplane:~$ kubectl get pods
NAME                                   READY   STATUS    RESTARTS   AGE
kubernetes-bootcamp-5cc66bcc9b-wwkxv   1/1     Running   0          10m
root@controlplane:~$ kubectl get services
NAME         TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)   AGE
kubernetes   ClusterIP   10.96.0.1    <none>        443/TCP   3d15h
```

- 03번에서 만든 Pod(`...-wwkxv`)가 그대로 `Running` 상태이므로 재배포 없이 진행했다.
- Service는 기본 `kubernetes`(ClusterIP) 하나만 있다.

### 3-1. NodePort Service 생성

```text
root@controlplane:~$ kubectl expose deployment/kubernetes-bootcamp --type="NodePort" --port 8080
service/kubernetes-bootcamp exposed
```

- `kubernetes-bootcamp` Deployment를 `NodePort` 타입으로, Service 포트 8080으로 노출했다.

### 3-2. Service 상세 정보 확인

```text
root@controlplane:~$ kubectl describe services/kubernetes-bootcamp
Name:                     kubernetes-bootcamp
Namespace:                default
Labels:                   app=kubernetes-bootcamp
Annotations:              <none>
Selector:                 app=kubernetes-bootcamp
Type:                     NodePort
IP Family Policy:         SingleStack
IP Families:              IPv4
IP:                       10.100.4.120
IPs:                      10.100.4.120
Port:                     <unset>  8080/TCP
TargetPort:               8080/TCP
NodePort:                 <unset>  30642/TCP
Endpoints:                192.168.1.123:8080
Session Affinity:         None
External Traffic Policy:  Cluster
Internal Traffic Policy:  Cluster
Events:                   <none>
```

| 항목 | 값 | 의미 |
|---|---|---|
| Selector | app=kubernetes-bootcamp | 이 label을 가진 Pod를 대상으로 삼는다. |
| Type | NodePort | 각 Node의 포트로 외부에 노출한다. |
| IP | 10.100.4.120 | Service의 클러스터 내부 IP (ClusterIP) |
| Port / TargetPort | 8080/TCP / 8080/TCP | Service가 받는 포트 / Pod로 전달하는 포트 |
| NodePort | 30642/TCP | 외부에 열린 포트. 기본 NodePort 범위(30000~32767) 안의 값이다. (보충) |
| Endpoints | 192.168.1.123:8080 | 트래픽을 보내는 Pod의 `IP:포트` |

- **Endpoints `192.168.1.123:8080`은 03번과 이번 실습에서 확인한 Pod IP `192.168.1.123`과 같다.** Selector로 찾은 Pod가 Service의 대상으로 연결된 것을 확인했다.

### 3-3. NodePort 번호를 변수에 저장

```text
root@controlplane:~$ export NODE_PORT="$(kubectl get services/kubernetes-bootcamp -o go-template='{{(index .spec.ports 0).nodePort}}')"
root@controlplane:~$ echo "NODE_PORT=$NODE_PORT"
NODE_PORT=30642
```

- Service의 첫 번째 포트에서 `nodePort` 값만 뽑아 `NODE_PORT`에 저장했다.

### 3-4. 노드 IP 확인 (minikube ip 대체)

```text
root@controlplane:~$ kubectl get nodes -o wide
NAME           STATUS   ROLES           AGE     VERSION   INTERNAL-IP   EXTERNAL-IP   OS-IMAGE             KERNEL-VERSION              CONTAINER-RUNTIME
controlplane   Ready    control-plane   3d15h   v1.37.1   172.30.1.2    <none>        Ubuntu 24.04.5 LTS   6.8.0-147-generic (amd64)   containerd://2.2.1
node01         Ready    <none>          3d14h   v1.37.1   172.30.2.2    <none>        Ubuntu 24.04.5 LTS   6.8.0-147-generic (amd64)   containerd://2.2.1
```

- 공식 문서의 `minikube ip` 대신 노드의 INTERNAL-IP를 사용한다. Pod가 올라가 있는 `node01`의 `172.30.2.2`를 썼다.

### 3-5. NodePort로 외부 접속 테스트

```text
root@controlplane:~$ export NODE_IP=172.30.2.2
root@controlplane:~$ curl http://$NODE_IP:$NODE_PORT
Hello Kubernetes bootcamp! | Running on: kubernetes-bootcamp-5cc66bcc9b-wwkxv | v=1
```

- `NodeIP:NodePort`(`172.30.2.2:30642`)로 접속하니 프록시 없이 앱이 응답했다. Service로 앱을 노출한 것이다.
- 응답의 Pod 이름이 `kubernetes-bootcamp-5cc66bcc9b-wwkxv`로, 02, 03번과 같다.

### 3-6. Label 확인과 사용

Deployment의 label 확인:

```text
root@controlplane:~$ kubectl describe deployment
Name:                   kubernetes-bootcamp
Namespace:              default
CreationTimestamp:      Mon, 09 Nov 2026 02:25:44 +0000
Labels:                 app=kubernetes-bootcamp
...
Selector:               app=kubernetes-bootcamp
Replicas:               1 desired | 1 updated | 1 total | 1 available | 0 unavailable
StrategyType:           RollingUpdate
RollingUpdateStrategy:  25% max unavailable, 25% max surge
Pod Template:
  Labels:  app=kubernetes-bootcamp
  Containers:
   kubernetes-bootcamp:
    Image:         gcr.io/google-samples/kubernetes-bootcamp:v1
...
Conditions:
  Type           Status  Reason
  ----           ------  ------
  Available      True    MinimumReplicasAvailable
  Progressing    True    NewReplicaSetAvailable
OldReplicaSets:  <none>
NewReplicaSet:   kubernetes-bootcamp-5cc66bcc9b (1/1 replicas created)
Events:
  Type    Reason             Age   From                   Message
  ----    ------             ----  ----                   -------
  Normal  ScalingReplicaSet  11m   deployment-controller  Scaled up replica set kubernetes-bootcamp-5cc66bcc9b from 0 to 1
```

(출력이 길어 Annotations, MinReadySeconds, Volumes 등은 `...`로 생략했다.)

- Deployment가 자동으로 만든 label은 `app=kubernetes-bootcamp`이고, Selector와 Pod Template의 Labels도 같은 값이다.
- Events를 보면 `deployment-controller`가 ReplicaSet을 0에서 1로 늘렸다. Deployment가 ReplicaSet을 통해 Pod를 만든다는 구조가 그대로 보인다.
- `RollingUpdateStrategy`(25% max unavailable, 25% max surge)는 06번 업데이트에서 다시 나올 내용이다. (보충)

label로 Pod와 Service 조회:

```text
root@controlplane:~$ kubectl get pods -l app=kubernetes-bootcamp
NAME                                   READY   STATUS    RESTARTS   AGE
kubernetes-bootcamp-5cc66bcc9b-wwkxv   1/1     Running   0          11m
root@controlplane:~$ kubectl get services -l app=kubernetes-bootcamp
NAME                  TYPE       CLUSTER-IP     EXTERNAL-IP   PORT(S)          AGE
kubernetes-bootcamp   NodePort   10.100.4.120   <none>        8080:30642/TCP   19s
```

- `-l`은 label로 걸러서 조회하는 옵션이다. 같은 label로 Pod와 Service가 모두 조회된다.
- `PORT(S)`의 `8080:30642/TCP`는 `Service 포트:NodePort`를 뜻한다.

Pod에 새 label 붙이기:

```text
root@controlplane:~$ export POD_NAME="$(kubectl get pods -o go-template --template '{{range .items}}{{.metadata.name}}{{"\n"}}{{end}}')"
root@controlplane:~$ echo "Name of the Pod: $POD_NAME"
Name of the Pod: kubernetes-bootcamp-5cc66bcc9b-wwkxv
root@controlplane:~$ kubectl label pods "$POD_NAME" version=v1
pod/kubernetes-bootcamp-5cc66bcc9b-wwkxv labeled
root@controlplane:~$ kubectl describe pods "$POD_NAME"
Name:             kubernetes-bootcamp-5cc66bcc9b-wwkxv
Namespace:        default
...
Node:             node01/172.30.2.2
...
Labels:           app=kubernetes-bootcamp
                  pod-template-hash=5cc66bcc9b
                  version=v1
...
Status:           Running
IP:               192.168.1.123
...
Controlled By:    ReplicaSet/kubernetes-bootcamp-5cc66bcc9b
...
root@controlplane:~$ kubectl get pods -l version=v1
NAME                                   READY   STATUS    RESTARTS   AGE
kubernetes-bootcamp-5cc66bcc9b-wwkxv   1/1     Running   0          11m
```

(`describe pods` 출력은 03번과 같은 부분을 `...`로 생략했다.)

- `kubectl label pods <Pod 이름> version=v1`로 Pod에 `version=v1` label을 추가했다.
- `describe pods`의 Labels에 기존 `app`, `pod-template-hash`와 함께 `version=v1`이 추가된 것을 확인했다.
- 새 label로도 `get pods -l version=v1`로 Pod가 조회된다.

### 3-7. Service 삭제

```text
root@controlplane:~$ kubectl delete service -l app=kubernetes-bootcamp
service "kubernetes-bootcamp" deleted from default namespace
root@controlplane:~$ kubectl get services
NAME         TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)   AGE
kubernetes   ClusterIP   10.96.0.1    <none>        443/TCP   3d15h
```

- label(`-l`)로 지정해서 `kubernetes-bootcamp` Service만 삭제했고, 기본 `kubernetes` Service는 남았다.

### 3-8. Service 삭제 후 외부 접속과 Pod 내부 접속

삭제 직후 바로 실행한 결과:

```text
root@controlplane:~$ curl --max-time 5 http://$NODE_IP:$NODE_PORT
Hello Kubernetes bootcamp! | Running on: kubernetes-bootcamp-5cc66bcc9b-wwkxv | v=1
root@controlplane:~$ kubectl exec -ti $POD_NAME -- curl http://localhost:8080
Hello Kubernetes bootcamp! | Running on: kubernetes-bootcamp-5cc66bcc9b-wwkxv | v=1
```

- `kubectl exec`로 Pod 안에서 한 `curl`이 성공했다. Service를 삭제해도 Deployment가 Pod를 관리하고 있어서 앱은 계속 실행 중이다.
- **예상과 다른 결과**: Service를 삭제한 직후 실행한 외부 `curl`이 실패하지 않고 응답을 받았다. 공식 문서는 이때 앱에 외부에서 접근할 수 없어야 한다고 설명한다. 삭제 명령과 `curl`을 바로 이어서 실행해서, Service 삭제가 노드의 네트워크 규칙에 반영되기 전이었을 가능성이 있다. (추정, 확인하지 못함)

잠시 뒤 다시 실행한 결과:

```text
root@controlplane:~$ curl --max-time 5 http://172.30.2.2:30642
(재실행 결과를 여기에 붙여넣기)
```

- (재실행 결과가 실패일 때) 몇 초 뒤 다시 실행하니 외부 접속이 되지 않았다. Service 삭제가 반영되는 데 시간이 걸린 것이다. 이로써 Service가 외부 노출의 통로였음을 확인했다.
- (재실행 결과가 성공일 때) 몇 초 뒤에도 외부 접속이 계속 되었다. 원인은 확인하지 못했고, 이 환경의 네트워크 구성(cilium)과 관련이 있는지 추가 확인이 필요하다.

## 4. Killercoda에서 달라진 점 / 막힌 점

- 공식 문서의 `curl http://"$(minikube ip):$NODE_PORT"`는 minikube 전용이라, `kubectl get nodes -o wide`로 확인한 노드 INTERNAL-IP(`172.30.2.2`)와 NodePort(`30642`)로 접속했다.
- 공식 문서의 `minikube service kubernetes-bootcamp --url`은 사용하지 않았다.
- 03번에서 만든 Deployment와 Pod를 그대로 사용했다. (재배포 없음)
- `--max-time 5` 옵션은 응답이 없을 때 5초 뒤 끊기 위해 덧붙인 것이다. (보충)
- 막힌 점: Service 삭제 직후 외부 접속이 실패하지 않고 응답이 왔다. 3-8 참고.

## 5. 한 줄 회고

- Pod의 IP는 Service 없이는 외부에서 접근할 수 없고, Service가 label selector로 Pod를 찾아 Endpoints로 연결해 `NodeIP:NodePort`로 앱을 노출해 준다는 것을 직접 확인했다. Service를 지워도 Deployment와 Pod는 남아 앱이 계속 실행된다는 점도 `exec`로 확인했다.