# 06. Update your app

- 구분: 기록용 (영상 없음)
- 공식 튜토리얼: https://kubernetes.io/docs/tutorials/kubernetes-basics/update/update-intro/
- 실습 날짜: YYYY-MM-DD

## 1. 목표

- 롤링 업데이트를 수행하고 롤백한다.

## 2. 개념 정리 (내 말로)

- 롤링 업데이트(무중단):
- 롤백이 필요한 상황:

## 3. 실습 로그

> 명령어 → 출력 → 한 줄 설명 순서로 기록

```text
$ kubectl set image deployments/kubernetes-bootcamp kubernetes-bootcamp=docker.io/jocatalin/kubernetes-bootcamp:v2
$ kubectl rollout status deployments/kubernetes-bootcamp
$ kubectl set image deployments/kubernetes-bootcamp kubernetes-bootcamp=gcr.io/google-samples/kubernetes-bootcamp:v10   # 일부러 실패시키기
$ kubectl rollout undo deployments/kubernetes-bootcamp
# 정리
$ kubectl delete deployments/kubernetes-bootcamp services/kubernetes-bootcamp
```

## 4. Killercoda에서 달라진 점 / 막힌 점

-

## 5. 한 줄 회고

-
