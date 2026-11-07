# 02. Deploy an app

- 공식 튜토리얼: https://kubernetes.io/docs/tutorials/kubernetes-basics/deploy-app/deploy-intro/
- 실습 날짜: 2026-11-07

## 1. 목표

- Deployment가 무엇인지 이해한다.
- kubectl로 첫 앱을 배포한다.

## 2. 개념 정리

### Deployment의 역할

- 앱 인스턴스(Pod)를 **어떻게 만들고 업데이트할지** 쿠버네티스에 알려 주는 객체다. 이미지와 인스턴스 개수(replicas)를 지정한다.
- Deployment를 만들면 Control Plane이 인스턴스를 클러스터의 Node에 스케줄링한다.
- **자가 치유(self-healing)**: Deployment controller가 인스턴스를 계속 감시하다가, 인스턴스가 있던 Node가 죽거나 삭제되면 다른 Node에 대체 인스턴스를 만든다.
- 설치 스크립트로 앱을 시작하던 방식은 머신 장애에서 복구할 수 없었지만, Deployment는 인스턴스를 만들고 **계속 유지**한다는 점이 다르다.

### kubectl 명령어 형식

- `kubectl action resource` 형태다. 예: `get nodes`, `create deployment`, `delete ...`
- 막히면 `--help`를 붙여 사용법을 확인한다. (예: `kubectl get nodes --help`)

### kubectl proxy가 필요한 이유

- Pod는 사설 격리 네트워크에서 실행되어 기본적으로 같은 클러스터 안의 Pod, Service에서만 보이고 **클러스터 밖에서는 접근할 수 없다.**
- `kubectl proxy`는 내 쪽에서 클러스터 내부 네트워크로 들어가는 통로를 열어 준다.
- API 서버는 Pod 이름을 기준으로 각 Pod의 접근 경로를 자동으로 만든다. 프록시를 거치면 아래 형태로 접근한다.
  `/api/v1/namespaces/default/pods/<Pod 이름>:8080/proxy/`
- 프록시는 켜 두는 동안 출력이 없고 터미널을 점유하기 때문에 **별도 탭(Tab 2)** 에서 실행해야 한다. .

## 3. 실습 로그

> 명령어 → 출력 → 한 줄 설명 순서로 기록

### 3-1. Deployment 생성

```text
root@controlplane:~$ kubectl create deployment kubernetes-bootcamp --image=gcr.io/google-samples/kubernetes-bootcamp:v1
deployment.apps/kubernetes-bootcamp created
```

- 이름 `kubernetes-bootcamp`, 이미지 `gcr.io/google-samples/kubernetes-bootcamp:v1`로 Deployment를 만들었다.
- 이 명령이 적합한 Node를 찾고, 앱을 스케줄링하고, 필요할 때 다른 Node에 다시 배치하도록 클러스터를 구성한다.

### 3-2. Deployment 확인

```text
root@controlplane:~$ kubectl get deployments
NAME                  READY   UP-TO-DATE   AVAILABLE   AGE
kubernetes-bootcamp   0/1     1            0           15s
```

- 열 의미: READY(준비된 개수/원하는 개수), UP-TO-DATE(최신 설정으로 갱신된 개수), AVAILABLE(사용 가능한 개수), AGE(생성 후 경과 시간)
- 생성 직후 15초 시점에는 `0/1`, AVAILABLE `0`이었다. 이후 Pod 이름 조회와 `curl` 응답이 정상으로 왔으므로 Pod가 정상 실행된 것을 확인했다.

### 3-3. 프록시 실행 (Tab 2)

```text
# [Tab 2]
root@controlplane:~$ kubectl proxy
Starting to serve on 127.0.0.1:8001
```

- 로컬 `127.0.0.1:8001`에서 프록시가 클러스터로 요청을 전달한다.
- 이 탭은 이후 `curl` 요청에도 계속 켜 둔다.

### 3-4. 프록시를 통한 API 서버 질의

```text
root@controlplane:~$ curl http://localhost:8001/version
  "major": "1",
  "minor": "37",
  "emulationMajor": "1",
  "emulationMinor": "37",
  "minCompatibilityMajor": "1",
  "minCompatibilityMinor": "36",
  "gitVersion": "v1.37.1",
  "gitCommit": "f78e722310e50bcaca9276be22276d9e91d91308",
  "gitTreeState": "clean",
  "buildDate": "2026-09-23T17:06:23Z",
  "goVersion": "go1.26.8",
  "compiler": "gc",
  "platform": "linux/amd64"
```

- 프록시를 거쳐 API 서버의 버전 정보를 받았다.

### 3-4. Pod 이름 저장

```text
root@controlplane:~$ export POD_NAME=$(kubectl get pods -o go-template --template '{{range .items}}{{.metadata.name}}{{"\n"}}{{end}}')
root@controlplane:~$ echo Name of the Pod: $POD_NAME
Name of the Pod: kubernetes-bootcamp-5cc66bcc9b-fm6pv
```

- Pod 목록(`.items`)에서 이름(`.metadata.name`)만 뽑아 환경 변수 `POD_NAME`에 저장했다.

### 3-5. 프록시를 통한 Pod 접근

```text
root@controlplane:~$ curl http://localhost:8001/api/v1/namespaces/default/pods/$POD_NAME:8080/proxy/
Hello Kubernetes bootcamp! | Running on: kubernetes-bootcamp-5cc66bcc9b-fm6pv | v=1
```

- 프록시를 통해 Pod의 8080 포트에서 실행 중인 앱에 접근했다.
- 응답에 `Running on:` 뒤로 응답한 Pod의 이름이 나오고, `v=1`은 배포한 이미지 태그 `v1`과 일치한다.

## 4. Killercoda에서 달라진 점 / 막힌 점

- 공식 문서는 노드가 1개인 minikube 기준이지만, 이번 환경은 노드가 2개(controlplane, node01)다.
- 막힌 점: 없음

## 5. 한 줄 회고

- Deployment를 만들면 쿠버네티스가 Pod를 스케줄링하고 계속 감시한다는 점을 이해했고, Pod가 사설 네트워크에 있어서 `kubectl proxy`를 통해야만 접근할 수 있다는 점을 직접 확인했다.