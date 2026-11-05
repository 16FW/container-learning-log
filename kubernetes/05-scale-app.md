# 05. Scale up your app

- 구분: 기록용 (영상 없음)
- 공식 튜토리얼: https://kubernetes.io/docs/tutorials/kubernetes-basics/scale/scale-intro/
- 실습 날짜: YYYY-MM-DD

## 1. 목표

- Deployment를 kubectl로 수동 확장·축소한다.

## 2. 개념 정리 (내 말로)

- ReplicaSet:
- 로드밸런싱이 확인되는 방식:

## 3. 실습 로그

> 명령어 → 출력 → 한 줄 설명 순서로 기록

```text
$ kubectl get rs
$ kubectl scale deployments/kubernetes-bootcamp --replicas=4
$ kubectl get pods -o wide
$ kubectl scale deployments/kubernetes-bootcamp --replicas=2
```

## 4. Killercoda에서 달라진 점 / 막힌 점

-

## 5. 한 줄 회고

-
