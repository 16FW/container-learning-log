# 01. Create a cluster

- 구분: 과제 (영상 촬영)
- 공식 튜토리얼: https://kubernetes.io/docs/tutorials/kubernetes-basics/create-cluster/cluster-intro/
- 실습 날짜: 2026-11-05

## 1. 목표

- 쿠버네티스 클러스터가 무엇인지 이해한다.
- Minikube가 무엇인지 이해한다.
- 클러스터 상태를 확인한다.

## 2. 개념 정리

### 클러스터란?

- 컨테이너화된 애플리케이션을 배포하고 관리하기 위해 여러 Node를 하나로 묶은 집합
- 크게 **Control Plane**과 **Worker Node**로 구성된다.
  - Control Plane: 클러스터 전체를 관리한다.
  - Worker Node: 실제 애플리케이션 컨테이너가 실행되는 환경을 제공한다.
- 여러 Node를 하나의 클러스터로 묶어 애플리케이션의 배포, 확장, 복구 등을 통합적으로 관리할 수 있다.

### Control Plane (클러스터를 관리하는 쪽)

| 구성 요소 | 역할 |
|---|---|
| kube-apiserver | 클러스터의 모든 요청을 받는 진입점. kubectl 명령도 API Server를 통해 전달된다. |
| etcd | 클러스터의 상태와 설정 정보를 저장하는 데이터 저장소 |
| kube-scheduler | 새로 생성된 Pod를 어떤 Node에 배치할지 결정 |
| kube-controller-manager | 실제 클러스터 상태가 사용자가 원하는 상태와 일치하도록 관리 |

### Node (앱이 실행되는 쪽)

| 구성 요소 | 역할 |
|---|---|
| kubelet | Node에서 Pod의 생성 및 실행 상태를 관리하고 Control Plane과 통신 |
| kube-proxy | Service의 네트워크 통신 및 트래픽 전달을 담당 (일반 개념, 아래 참고) |
| Container Runtime | 컨테이너를 실제로 실행하는 환경 (예: containerd) |

> 공식 튜토리얼 본문에는 Control Plane, Node, kubelet, containerd/CRI-O까지만 나온다.
> apiserver, etcd, scheduler, controller-manager, kube-proxy의 역할 설명은 (보충)이다.

### Minikube란?

- 로컬 환경에서 쿠버네티스 클러스터를 실행할 수 있도록 지원하는 도구
- 내 컴퓨터에 VM을 만들어 **노드 1개짜리** 단순한 클러스터를 띄운다.
- Minikube CLI는 start, stop, status, delete 같은 기본 동작을 제공한다.

### 이번 실습 환경과 비교

| 구분 | 공식 튜토리얼 (Minikube) | 이번 실습 (Killercoda) |
|---|---|---|
| 노드 수 | 1개 | 2개 (controlplane, node01) |
| 클러스터 생성 | `minikube start` | 이미 실행 중 |
| 컨테이너 런타임 | (환경에 따라 다름) | containerd (00번 로그 참고) |

## 3. 실습 로그

> 명령어 → 출력 → 한 줄 설명 순서로 기록

### 3-1. 버전 확인

```text
root@controlplane:~$ kubectl version
Client Version: v1.37.1
Kustomize Version: v5.8.1
Server Version: v1.37.1
```

- kubectl(Client)과 클러스터(Server)의 버전이 v1.37.1로 같다.

### 3-2. 클러스터 정보 확인

```text
root@controlplane:~$ kubectl cluster-info
Kubernetes control plane is running at https://172.30.1.2:6443
CoreDNS is running at https://172.30.1.2:6443/api/v1/namespaces/kube-system/services/kube-dns:dns/proxy

To further debug and diagnose cluster problems, use 'kubectl cluster-info dump'.
```

- API 서버가 `https://172.30.1.2:6443`에서 동작 중이다.

### 3-3. 노드 확인

```text
root@controlplane:~$ kubectl get nodes
NAME           STATUS   ROLES           AGE   VERSION
controlplane   Ready    control-plane   31d   v1.37.1
node01         Ready    <none>          31d   v1.37.1
```

- 노드 2개가 모두 `Ready` 상태다.

### 3-4. 모든 네임스페이스의 Pod 확인 (보충)

```text
root@controlplane:~$ kubectl get pods -A
NAMESPACE            NAME                                      READY   STATUS    RESTARTS      AGE
kube-system          cilium-envoy-7nwgk                        1/1     Running   2 (15m ago)   31d
kube-system          cilium-envoy-bp5gr                        1/1     Running   1 (15m ago)   31d
kube-system          cilium-h4qjm                              1/1     Running   2 (15m ago)   31d
kube-system          cilium-kbr55                              1/1     Running   1 (15m ago)   31d
kube-system          cilium-operator-86477b5bf8-cn7dc          1/1     Running   5 (15m ago)   31d
kube-system          coredns-854c976c6-8fwvq                   1/1     Running   1 (15m ago)   31d
kube-system          coredns-854c976c6-zwrlp                   1/1     Running   1 (15m ago)   31d
kube-system          etcd-controlplane                         1/1     Running   2 (15m ago)   31d
kube-system          kube-apiserver-controlplane               1/1     Running   2 (15m ago)   31d
kube-system          kube-controller-manager-controlplane      1/1     Running   4 (15m ago)   31d
kube-system          kube-scheduler-controlplane               1/1     Running   4 (15m ago)   31d
local-path-storage   local-path-provisioner-5dc7cd49fd-rkvvt   1/1     Running   2 (15m ago)   31d
```

- `-A`는 모든 네임스페이스를 조회하는 옵션이다.
- 평소 `kubectl get pods`가 비어 있던 이유는 시스템 Pod들이 `kube-system` 네임스페이스에 있기 때문이다.

출력에서 확인한 구성 요소는 아래와 같다.

| Pod 이름 | 분류 | 설명 |
|---|---|---|
| kube-apiserver-controlplane | Control Plane | API 서버 |
| etcd-controlplane | Control Plane | 클러스터 상태 저장소 |
| kube-scheduler-controlplane | Control Plane | Pod 배치 결정 |
| kube-controller-manager-controlplane | Control Plane | 원하는 상태 유지 |
| coredns-... (2개) | 클러스터 DNS | 클러스터 내부 이름 해석 |
| cilium-... | 네트워크 | Pod 간 네트워크를 담당하는 CNI 플러그인 |
| local-path-provisioner-... | 스토리지 | 로컬 경로 기반 저장소 제공 |

- 이 출력에는 `kube-proxy` Pod가 보이지 않는다. 대신 `cilium`이 있다. 네트워킹은 cilium이 담당하는 것으로 보이지만, 확인하지 못했으므로 추정이다.
- `kubelet`은 Pod가 아니라 각 노드에서 서비스로 실행되기 때문에 이 목록에 나오지 않는다. (보충)

(참고: `minikube start`, `minikube status`는 Killercoda에서 생략)

## 4. Killercoda에서 달라진 점 / 막힌 점

- 공식 문서의 `minikube start`, `minikube status`는 Killercoda에서 실행하지 않았다.
  대신 `kubectl cluster-info`, `kubectl get nodes`로 클러스터 상태를 확인했다.
- 막힌 점: 없음

## 5. 한 줄 회고

- 쿠버네티스 클러스터가 Control Plane과 Worker Node로 구성되며, 각 구성 요소가 역할을 분담해 애플리케이션의 실행 환경을 관리한다는 것을 이해했다. 또한 kubectl 명령어로 Node와 Pod의 상태를 직접 확인하면서 클러스터의 전체적인 구조를 파악할 수 있었다.