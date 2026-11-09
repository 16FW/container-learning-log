# 03. Explore your app

- 구분: 과제 (영상 촬영)
- 공식 튜토리얼: https://kubernetes.io/docs/tutorials/kubernetes-basics/explore/explore-intro/
- 실습 날짜: 2026-11-09

## 1. 목표

- Pod가 무엇인지 이해한다.
- Node가 무엇인지 이해한다.
- `get`, `describe`, `logs`, `exec`로 배포된 앱을 확인하고 문제를 진단하는 방법을 익힌다.

## 2. 개념 정리 (내 말로)

### Pod

- 하나 이상의 컨테이너와 그 컨테이너들이 공유하는 자원을 묶은 단위다. 공유 자원은 세 가지다.
  - 공유 스토리지 (Volume)
  - 네트워크 (클러스터 내 고유 IP 주소)
  - 컨테이너 실행 정보 (이미지 버전, 사용할 포트 등)
- Pod 안의 컨테이너들은 IP 주소와 포트 공간을 공유하고, 항상 같은 Node에 함께 스케줄링된다.
- Pod는 쿠버네티스의 **원자적 단위**다. Deployment는 컨테이너를 직접 만들지 않고, **컨테이너가 들어 있는 Pod**를 만든다.
- 컨테이너를 한 Pod에 같이 넣는 것은 서로 밀접하게 연결되어 디스크 같은 자원을 공유해야 할 때뿐이다.
- Pod는 스케줄링된 Node에 묶여 있고, 종료되거나 삭제될 때까지 그 Node에 있다. Node가 고장 나면 동일한 Pod가 다른 Node에 만들어진다.

### Node

- Pod가 실제로 실행되는 워커 머신이다. 가상 머신일 수도, 물리 머신일 수도 있다.
- 한 Node에 Pod가 여러 개 있을 수 있고, Control Plane이 각 Node의 가용 자원을 고려해 자동으로 스케줄링한다.
- 모든 Node는 최소한 아래 두 가지를 실행한다.
  - **kubelet**: Control Plane과 Node 사이의 통신을 담당하고 Pod와 컨테이너를 관리한다.
  - **컨테이너 런타임** (예: containerd): 레지스트리에서 이미지를 받아 풀고 앱을 실행한다.

### 트러블슈팅 명령어 4종

| 명령어 | 용도 |
|---|---|
| `kubectl get` | 리소스 목록 조회 |
| `kubectl describe` | 리소스 상세 정보 조회 (사람이 읽기 위한 출력이며 스크립트 처리용이 아님) |
| `kubectl logs` | Pod 안 컨테이너의 로그 출력 |
| `kubectl exec` | Pod 안 컨테이너에서 명령어 실행 |

### Deployment, ReplicaSet, Pod의 관계 (보충)

- 이번 `describe` 출력의 `Controlled By: ReplicaSet/kubernetes-bootcamp-5cc66bcc9b`를 보면 Pod는 ReplicaSet이 관리하고 있다.
- Pod 이름은 `<Deployment 이름>-<ReplicaSet 식별자>-<무작위 문자>` 형태이고, 가운데 식별자 `5cc66bcc9b`가 Label `pod-template-hash`와 같다.

## 3. 실습 로그

> 명령어 → 출력 → 한 줄 설명 순서로 기록

### 3-0. 사전 확인: 환경 초기화로 Deployment 재배포

```text
root@controlplane:~$ kubectl get deployments
No resources found in default namespace.
```

- 02번 실습 이후 시간이 지나 Killercoda 환경이 초기화되어 Deployment가 사라져 있었다.
- 02번과 같은 명령어로 다시 배포했다.

```text
root@controlplane:~$ kubectl create deployment kubernetes-bootcamp --image=gcr.io/google-samples/kubernetes-bootcamp:v1
deployment.apps/kubernetes-bootcamp created
root@controlplane:~$ kubectl get deployments
NAME                  READY   UP-TO-DATE   AVAILABLE   AGE
kubernetes-bootcamp   0/1     1            0           3s
```

- 생성 3초 시점이라 READY가 `0/1`이다.
- 02번의 Pod 이름은 `...-fm6pv`였고, 재배포로 새 Pod `...-wwkxv`가 만들어졌다.

### 3-1. Pod 목록 확인 (`get`)

```text
root@controlplane:~$ kubectl get pods
NAME                                   READY   STATUS    RESTARTS   AGE
kubernetes-bootcamp-5cc66bcc9b-wwkxv   1/1     Running   0          19s
```

- Pod가 `1/1 Running` 상태가 되었다.

### 3-2. Pod 상세 정보 확인 (`describe`)

```text
root@controlplane:~$ kubectl describe pods
Name:             kubernetes-bootcamp-5cc66bcc9b-wwkxv
Namespace:        default
...
Node:             node01/172.30.2.2
Start Time:       Mon, 09 Nov 2026 02:25:44 +0000
Labels:           app=kubernetes-bootcamp
                  pod-template-hash=5cc66bcc9b
Annotations:      <none>
Status:           Running
IP:               192.168.1.123
IPs:
  IP:           192.168.1.123
Controlled By:  ReplicaSet/kubernetes-bootcamp-5cc66bcc9b
Containers:
  kubernetes-bootcamp:
    Container ID:   containerd://6e1f5b08ca2de9848c76b31b933782d1c9ac264deed4f4ef95c7d19217723349
    Image:          gcr.io/google-samples/kubernetes-bootcamp:v1
    Image ID:       gcr.io/google-samples/kubernetes-bootcamp@sha256:0d6b8ee63bb57c5f5b6156f446b3bc3b3c143d233037f3a2f00e279c8fcc64af
    Port:           <none>
    Host Port:      <none>
    State:          Running
      Started:      Mon, 09 Nov 2026 02:25:59 +0000
    Ready:          True
    Restart Count:  0
    Environment:    <none>
    ...
Conditions:
  Type                        Status
  PodReadyToStartContainers   True 
  Initialized                 True 
  Ready                       True 
  ContainersReady             True 
  PodScheduled                True 
...
Events:
  Type    Reason     Age   From               Message
  ----    ------     ----  ----               -------
  Normal  Scheduled  20s   default-scheduler  Successfully assigned default/kubernetes-bootcamp-5cc66bcc9b-wwkxv to node01
  Normal  Pulling    17s   kubelet            spec.containers{kubernetes-bootcamp}: Pulling image "gcr.io/google-samples/kubernetes-bootcamp:v1"
  Normal  Pulled     5s    kubelet            spec.containers{kubernetes-bootcamp}: Successfully pulled image "gcr.io/google-samples/kubernetes-bootcamp:v1" in 11.82s (11.82s including waiting). Image size: 83642968 bytes.
  Normal  Created    5s    kubelet            spec.containers{kubernetes-bootcamp}: Container created
  Normal  Started    5s    kubelet            spec.containers{kubernetes-bootcamp}: Container started
```

(출력이 길어 Priority, Service Account, Volumes, QoS Class, Node-Selectors, Tolerations 부분은 `...`로 생략했다.)

주요 항목의 의미는 아래와 같다.

| 항목 | 값 | 의미 |
|---|---|---|
| Node | node01/172.30.2.2 | Pod가 올라간 Node. controlplane이 아니라 **워커 노드 node01**에 배치되었다. |
| IP | 192.168.1.123 | 이 Pod의 클러스터 내부 IP. 00번의 Node IP(172.30.x.x)와 다른 대역이다. |
| Labels | app=kubernetes-bootcamp 등 | Pod를 구분하는 이름표. 04번에서 Service와 연결할 때 사용한다. |
| Controlled By | ReplicaSet/... | 이 Pod를 관리하는 ReplicaSet |
| Image | gcr.io/google-samples/kubernetes-bootcamp:v1 | 컨테이너를 만든 이미지 |
| Port | `<none>` | Pod 스펙에 포트를 선언하지 않았다는 뜻 (보충). 앱은 이후 `server.js`에서 보듯 8080에서 동작한다. |
| State / Ready | Running / True | 컨테이너가 정상 실행 중 |
| Restart Count | 0 | 재시작 없음 |
| Events | Scheduled → Pulling → Pulled → Created → Started | Pod 생애주기 기록 |

- Events의 **From** 열에 따라 역할이 나뉜다. 배치는 `default-scheduler`가, 이미지 받기, 컨테이너 생성과 시작은 `kubelet`이 했다. 01번에서 배운 Control Plane의 scheduler와 Node의 kubelet이 실제로 일한 흔적이다.
- 이미지를 받는 데 11.82초가 걸렸다. 이것이 02번과 3-0에서 `0/1`이던 이유로 확인된다.

### 3-3. 프록시를 통한 앱 접근

```text
# [Tab 2] kubectl proxy 실행 중
root@controlplane:~$ export POD_NAME="$(kubectl get pods -o go-template --template '{{range .items}}{{.metadata.name}}{{"\n"}}{{end}}')"
root@controlplane:~$ echo Name of the Pod: $POD_NAME
Name of the Pod: kubernetes-bootcamp-5cc66bcc9b-wwkxv
root@controlplane:~$ curl http://localhost:8001/api/v1/namespaces/default/pods/$POD_NAME:8080/proxy/
Hello Kubernetes bootcamp! | Running on: kubernetes-bootcamp-5cc66bcc9b-wwkxv | v=1
```

- 02번과 같은 방식으로 프록시를 통해 Pod의 앱에 접근했다.
- 컨테이너가 하나뿐이라 주소에 컨테이너 이름을 지정하지 않았다.

### 3-4. 컨테이너 로그 확인 (`logs`)

```text
root@controlplane:~$ kubectl logs $POD_NAME
Kubernetes Bootcamp App Started At: 2026-11-09T02:25:59.173Z | Running On:  kubernetes-bootcamp-5cc66bcc9b-wwkxv 

Running On: kubernetes-bootcamp-5cc66bcc9b-wwkxv | Total Requests: 1 | App Uptime: 35.787 seconds | Log Time: 2026-11-09T02:26:34.960Z
```

- 앱 시작 시각 `02:25:59`는 `describe`의 컨테이너 `Started` 시각(02:25:59)과 일치한다.
- `Total Requests: 1`은 방금 프록시로 보낸 `curl` 요청 1건이다. 요청이 올 때마다 로그가 한 줄씩 쌓인다.

### 3-5. 컨테이너 환경변수 확인 (`exec ... -- env`)

```text
root@controlplane:~$ kubectl exec "$POD_NAME" -- env
PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin
HOSTNAME=kubernetes-bootcamp-5cc66bcc9b-wwkxv
NPM_CONFIG_LOGLEVEL=info
NODE_VERSION=6.3.1
KUBERNETES_SERVICE_PORT_HTTPS=443
KUBERNETES_PORT=tcp://10.96.0.1:443
KUBERNETES_PORT_443_TCP=tcp://10.96.0.1:443
KUBERNETES_PORT_443_TCP_PROTO=tcp
KUBERNETES_PORT_443_TCP_PORT=443
KUBERNETES_PORT_443_TCP_ADDR=10.96.0.1
KUBERNETES_SERVICE_HOST=10.96.0.1
KUBERNETES_SERVICE_PORT=443
HOME=/root
```

- `HOSTNAME`이 **Pod 이름**과 같다. 앱이 `Running on:` 뒤에 Pod 이름을 출력하는 근거다(아래 `server.js` 참고).
- `NODE_VERSION=6.3.1`은 이 컨테이너가 Node.js 앱이라는 것을 보여 준다. (쿠버네티스 Node와 다른 개념)
- `KUBERNETES_SERVICE_HOST`, `KUBERNETES_PORT_*` 등은 쿠버네티스가 Pod 안에 자동으로 넣어 주는 값으로 보인다. (보충: 클러스터의 기본 `kubernetes` Service 주소 `10.96.0.1:443`)

### 3-6. 컨테이너 안으로 접속 (`exec -ti ... -- bash`)

```text
root@controlplane:~$ kubectl exec -ti $POD_NAME -- bash
root@kubernetes-bootcamp-5cc66bcc9b-wwkxv:/# cat server.js
var http = require('http');
var requests=0;
var podname= process.env.HOSTNAME;
var startTime;
var host;
var handleRequest = function(request, response) {
  response.setHeader('Content-Type', 'text/plain');
  response.writeHead(200);
  response.write("Hello Kubernetes bootcamp! | Running on: ");
  response.write(host);
  response.end(" | v=1\n");
  console.log("Running On:" ,host, "| Total Requests:", ++requests,"| App Uptime:", (new Date() - startTime)/1000 , "seconds", "| Log Time:",new Date());
}
var www = http.createServer(handleRequest);
www.listen(8080,function () {
    startTime = new Date();;
    host = process.env.HOSTNAME;
    console.log ("Kubernetes Bootcamp App Started At:",startTime, "| Running On: " ,host, "\n" );
});
root@kubernetes-bootcamp-5cc66bcc9b-wwkxv:/# curl http://localhost:8080
Hello Kubernetes bootcamp! | Running on: kubernetes-bootcamp-5cc66bcc9b-wwkxv | v=1
root@kubernetes-bootcamp-5cc66bcc9b-wwkxv:/# exit
exit
```

- `-ti` 옵션으로 터미널을 연결해 컨테이너 안에서 `bash`를 실행했다. 프롬프트가 `root@controlplane`에서 `root@kubernetes-bootcamp-...:/#`로 바뀐 것이 컨테이너 안에 들어왔다는 표시다.
- `server.js`를 읽어 앱의 동작을 확인했다.
  - `8080` 포트에서 HTTP 요청을 받는다.
  - 응답은 `Hello Kubernetes bootcamp! | Running on: <HOSTNAME> | v=1`이다. `host`는 환경변수 `HOSTNAME`에서 가져온다.
  - 요청이 올 때마다 `console.log`로 로그를 남긴다. 이것이 3-4에서 본 `kubectl logs` 출력의 정체다.
- 컨테이너 안에서는 `localhost:8080`으로 앱에 바로 접근된다. 같은 Pod 안이기 때문이다.
- `exit`로 컨테이너에서 빠져나왔다.

## 4. Killercoda에서 달라진 점 / 막힌 점

- 02번 이후 시간이 지나 Killercoda 환경이 초기화되어 Deployment가 사라져 있었다. 그래서 02번 명령어로 **다시 배포**했다. (Pod 이름 `fm6pv` → `wwkxv`)
- `kubectl proxy`는 Killercoda의 Tab 2에서 실행했다.
- 공식 문서 본문에는 `kubectl logs`가 설명만 있고 명령어 예시가 없어서, 강의 30번 슬라이드의 명령어를 같이 실행했다.
- 공식 문서는 노드가 1개인 minikube 기준이지만, 이번 환경은 노드가 2개이고 Pod는 **node01**에 배치되었다.
- 막힌 점: 없음

## 5. 한 줄 회고

- Pod가 컨테이너와 공유 자원을 묶은 단위이고 Node에 묶여 실행된다는 것을, `describe`의 Node, IP, Events와 `exec`로 컨테이너에 직접 들어가 본 경험으로 확인했다. `get`, `describe`, `logs`, `exec` 네 가지로 앱의 상태, 위치, 로그, 내부를 단계적으로 들여다볼 수 있다는 점을 알게 되었다.