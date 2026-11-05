# 00. 실습 환경 (Killercoda)

## 접속

- 주소: https://killercoda.com/playgrounds/scenario/kubernetes
- 로그인 필요 여부, 세션 시간 제한: (직접 확인해서 기록)

## 공식 튜토리얼(minikube)과 달라지는 점

| 공식 튜토리얼 | Killercoda에서는 |
|---|---|
| `minikube start`, `minikube status` | 생략 (클러스터가 이미 떠 있음) |
| `curl http://"$(minikube ip):$NODE_PORT"` | `kubectl get nodes -o wide`로 노드 INTERNAL-IP 확인 후 `curl <노드IP>:$NODE_PORT` |
| `minikube service ... --url` | 사용하지 않음 |
| `kubectl proxy` 실행 | 새 탭(Tab 2)에서 실행 |

## 확인 로그

```text
$ kubectl get nodes
(출력 붙여넣기)
```

## 막힌 점
-
