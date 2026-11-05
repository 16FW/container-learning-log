# Container Learning Log

SWCON221 마이크로서비스 프로그래밍 수업과 공식 문서를 기반으로 쿠버네티스와 도커를 실습하고 기록하는 학습 로그입니다.

- 실습 환경: Killercoda Kubernetes Playground (쿠버네티스), 로컬 Docker (도커)
- 기준 자료: [Kubernetes 공식 튜토리얼 - Learn Kubernetes Basics](https://kubernetes.io/docs/tutorials/kubernetes-basics/)

## 진행표

### Kubernetes (공식 튜토리얼 기준)

| # | 모듈 | 구분 | 상태 | 문서 |
|---|------|------|------|------|
| 0 | 실습 환경 (Killercoda) | 준비 | ✅ | [00-environment](kubernetes/00-environment.md) |
| 1 | Create a cluster | 과제 | ✅ | [01](kubernetes/01-create-cluster.md) |
| 2 | Deploy an app | 과제 | ⬜ | [02](kubernetes/02-deploy-app.md) |
| 3 | Explore your app | 과제 | ⬜ | [03](kubernetes/03-explore-app.md) |
| 4 | Expose your app publicly | 과제 | ⬜ | [04](kubernetes/04-expose-app.md) |
| 5 | Scale up your app | 기록용 | ⬜ | [05](kubernetes/05-scale-app.md) |
| 6 | Update your app | 기록용 | ⬜ | [06](kubernetes/06-update-app.md) |

과제 영상 링크: (업로드 후 여기에 추가)

### Docker

[docker/README.md](docker/README.md) 참고

## 문서 작성 규칙

- 파일명: `번호-주제.md` (소문자, 하이픈)
- 커밋 메시지: `docs(k8s): 02 deploy-app 실습 로그` 처럼 `타입(범위): 내용`
  - 타입: `docs`(기록), `chore`(구조·설정), `fix`(오타·오류 수정)
  - 범위: `k8s`, `docker`, `repo`
