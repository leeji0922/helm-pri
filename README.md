# Helm Chart with ArgoCD

이 프로젝트는 Helm 차트와 ArgoCD를 사용한 Kubernetes 애플리케이션 배포를 위한 템플릿입니다.

## 프로젝트 구조

```
basic-helm2/
├── project1/              # Project1 Helm 차트
│   ├── Chart.yaml        # Helm 차트 메타데이터
│   ├── values.yaml       # 기본 values
│   ├── templates/        # Kubernetes 리소스 템플릿
│   └── values/           # 환경별 values
│       ├── dev.yaml      # 개발 환경 설정
│       └── prod.yaml     # 프로덕션 환경 설정
├── project2/              # Project2 Helm 차트
│   ├── Chart.yaml
│   ├── values.yaml
│   ├── templates/
│   └── values/
│       ├── dev.yaml
│       └── prod.yaml
└── argocd-applicationset.yaml  # ArgoCD ApplicationSet 정의
```

## 환경별 설정

### 개발 환경 (dev)
- 브랜치: `develop`
- 자동 배포: 비활성화 (Jenkins에서 수동 배포)

### 프로덕션 환경 (prod)
- 브랜치: `main`
- 자동 배포: 비활성화 (수동 배포)

## 이미지 태그 관리
- 버전 번호 형식 사용 (예: 1.0.0)
- 각 배포마다 고유한 버전 번호 부여
- 문제 발생 시 특정 버전으로 롤백 가능

## 배포 프로세스

### 개발 환경 배포
1. `develop` 브랜치에 변경사항 푸시
2. Jenkins 파이프라인 실행
3. Jenkins에서 ArgoCD 싱크 명령어 실행:
   ```bash
   argocd app sync project1-dev
   # 또는
   argocd app sync project2-dev
   ```

### 프로덕션 환경 배포
1. `main` 브랜치에 변경사항 푸시
2. ArgoCD 웹 UI 또는 CLI에서 수동으로 싱크:
   ```bash
   argocd app sync project1-prod
   # 또는
   argocd app sync project2-prod
   ```

## 새로운 프로젝트 추가

1. 새로운 프로젝트 폴더 생성:
   ```
   basic-helm2/
   └── project3/
       ├── Chart.yaml
       ├── values.yaml
       ├── templates/
       └── values/
           ├── dev.yaml
           └── prod.yaml
   ```

2. `argocd-applicationset.yaml` 파일에 새 프로젝트 추가:
   ```yaml
   generators:
     - list:
         elements:
         - project: project3
           profiles:
           - name: dev
             namespace: project3-dev
             syncPolicy: {}
             branch: develop
           - name: prod
             namespace: project3-prod
             syncPolicy: {}
             branch: main
   ```

3. ArgoCD ApplicationSet 업데이트:
   ```bash
   kubectl apply -f basic-helm2/argocd-applicationset.yaml
   ```

## 모니터링 및 로깅
- Filebeat 설정 포함
- 각 프로젝트별 독립적인 ConfigMap과 Secret 사용
- 환경별로 다른 Elasticsearch/Kibana 설정 가능

## 노드 셀렉터
- 개발 환경: `type: dev`
- 프로덕션 환경: `type: prod`