# 00. 실습 환경 (Killercoda)

## 접속

- 주소: https://killercoda.com/playgrounds/scenario/kubernetes
- 로그인 필요 여부, 세션 시간 제한: 로그인 완료 , 4시간

## 공식 튜토리얼(minikube)과 달라지는 점

| 공식 튜토리얼 | Killercoda에서는 |
|---|---|
| `minikube start`, `minikube status` | 생략 (클러스터가 이미 떠 있음) |
| `curl http://"$(minikube ip):$NODE_PORT"` | `kubectl get nodes -o wide`로 노드 INTERNAL-IP 확인 후 `curl <노드IP>:$NODE_PORT` |
| `minikube service ... --url` | 사용하지 않음 |
| `kubectl proxy` 실행 | 새 탭(Tab 2)에서 실행 |

## 확인 로그

```
root@controlplane:~$ kubectl get nodes
NAME           STATUS   ROLES           AGE   VERSION
controlplane   Ready    control-plane   30d   v1.37.1
node01         Ready    <none>          30d   v1.37.1
root@controlplane:~$ kubectl get nodes -o wide
NAME           STATUS   ROLES           AGE   VERSION   INTERNAL-IP   EXTERNAL-IP   OS-IMAGE             KERNEL-VERSION              CONTAINER-RUNTIME
controlplane   Ready    control-plane   30d   v1.37.1   172.30.1.2    <none>        Ubuntu 24.04.5 LTS   6.8.0-146-generic (amd64)   containerd://2.2.1
node01         Ready    <none>          30d   v1.37.1   172.30.2.2    <none>        Ubuntu 24.04.5 LTS   6.8.0-146-generic (amd64)   containerd://2.2.1
root@controlplane:~$ kubectl version
Client Version: v1.37.1
Kustomize Version: v5.8.1
Server Version: v1.37.1
root@controlplane:~$ kubectl cluster-info
Kubernetes control plane is running at https://172.30.1.2:6443
CoreDNS is running at https://172.30.1.2:6443/api/v1/namespaces/kube-system/services/kube-dns:dns/proxy

To further debug and diagnose cluster problems, use 'kubectl cluster-info dump'.
root@controlplane:~$ kubectl get pods
No resources found in default namespace.
```

## kubectl이란?

kubectl은 클러스터의 API 서버에 요청을 보내는 명령줄 도구다. 아래 명령어는 모두 "API 서버에게 이 정보를 알려 달라"는 질문이다.

- 도커와 비교: `docker` CLI → Docker daemon(dockerd)에 요청, `kubectl` → API 서버에 요청

## 사용한 명령어 정리

| 명령어 | 의미 | 확인한 점 |
|---|---|---|
| `kubectl get nodes` | 클러스터의 노드 목록과 상태 조회 | 노드 2개, STATUS가 모두 Ready |
| `kubectl get nodes -o wide` | 노드의 IP, OS, 컨테이너 런타임 등 상세 조회 | INTERNAL-IP: 172.30.1.2 / 런타임: containerd://2.2.1 |
| `kubectl version` | kubectl(Client)과 클러스터(Server)의 버전 확인 | Client v1.37.1 / Server v1.37.1 |
| `kubectl cluster-info` | API 서버(control plane)와 CoreDNS 주소 확인 | API 서버: https://172.30.1.2:6443 |
| `kubectl get pods` | default 네임스페이스의 Pod 목록 조회 | 배포 전이라 비어 있음 |

- controlplane은 클러스터를 관리하는 노드, node01은 앱(Pod)이 실제로 올라가는 워커 노드 (ROLES `<none>`)
- AGE가 30d인 것은 미리 만들어 둔 플레이그라운드 이미지를 쓰기 때문으로 보임 (추정)

## 막힌 점
- 없음

## 한 줄 회고
- minikube 없이도 kubeadm 클러스터가 이미 준비돼 있어 바로 실습을 시작할 수 있었다.