# 04. Expose your app publicly

- 구분: 과제 (영상 촬영)
- 공식 튜토리얼: https://kubernetes.io/docs/tutorials/kubernetes-basics/expose/expose-intro/
- 실습 날짜: YYYY-MM-DD

## 1. 목표

- Service를 이해한다.
- Label과 Selector가 Service와 어떻게 연결되는지 이해한다.
- 클러스터 밖으로 앱을 노출한다.

## 2. 개념 정리 (내 말로)

- Service란 / NodePort란:
- Label과 Selector:
- Service를 지워도 Pod는 살아 있는 이유:

## 3. 실습 로그

> 명령어 → 출력 → 한 줄 설명 순서로 기록

```text
$ kubectl get services
$ kubectl expose deployment/kubernetes-bootcamp --type="NodePort" --port 8080
$ kubectl describe services/kubernetes-bootcamp
$ export NODE_PORT="$(kubectl get services/kubernetes-bootcamp -o go-template='{{(index .spec.ports 0).nodePort}}')"
$ kubectl get nodes -o wide     # INTERNAL-IP 확인 (minikube ip 대체)
$ curl <노드IP>:$NODE_PORT
$ kubectl label pods "$POD_NAME" version=v1
$ kubectl get pods -l version=v1
$ kubectl delete service -l app=kubernetes-bootcamp
$ kubectl exec -ti $POD_NAME -- curl http://localhost:8080
```

## 4. Killercoda에서 달라진 점 / 막힌 점

-

## 5. 한 줄 회고

-
