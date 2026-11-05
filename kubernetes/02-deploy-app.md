# 02. Deploy an app

- 구분: 과제 (영상 촬영)
- 공식 튜토리얼: https://kubernetes.io/docs/tutorials/kubernetes-basics/deploy-app/deploy-intro/
- 실습 날짜: YYYY-MM-DD

## 1. 목표

- Deployment가 무엇인지 이해한다.
- kubectl로 첫 앱을 배포한다.

## 2. 개념 정리 (내 말로)

- Deployment의 역할:
- kubectl proxy가 필요한 이유:

## 3. 실습 로그

> 명령어 → 출력 → 한 줄 설명 순서로 기록

```text
$ kubectl create deployment kubernetes-bootcamp --image=gcr.io/google-samples/kubernetes-bootcamp:v1
$ kubectl get deployments
# [Tab 2] kubectl proxy
$ curl http://localhost:8001/version
$ export POD_NAME=$(kubectl get pods -o go-template --template '{{range .items}}{{.metadata.name}}{{"\n"}}{{end}}')
$ echo Name of the Pod: $POD_NAME
$ curl http://localhost:8001/api/v1/namespaces/default/pods/$POD_NAME:8080/proxy/
```

## 4. Killercoda에서 달라진 점 / 막힌 점

-

## 5. 한 줄 회고

-
