# 03. Explore your app

- 구분: 과제 (영상 촬영)
- 공식 튜토리얼: https://kubernetes.io/docs/tutorials/kubernetes-basics/explore/explore-intro/
- 실습 날짜: YYYY-MM-DD

## 1. 목표

- Pod와 Node를 이해한다.
- 배포된 앱을 디버깅하는 방법을 익힌다.

## 2. 개념 정리 (내 말로)

- Pod란:
- Node란:
- get / describe / logs / exec 차이:

## 3. 실습 로그

> 명령어 → 출력 → 한 줄 설명 순서로 기록

```text
$ kubectl get pods
$ kubectl describe pods
$ kubectl logs $POD_NAME
$ kubectl exec "$POD_NAME" -- env
$ kubectl exec -ti $POD_NAME -- bash
# (Pod 안) curl http://localhost:8080
```

## 4. Killercoda에서 달라진 점 / 막힌 점

-

## 5. 한 줄 회고

-
