# Helm 차트 사용 안내

## 1. 차트 설치

### 개발 환경 설치
```bash
# project1 개발 환경 설치
helm install project1-dev ./project1 -f ./project1/values/dev.yaml

# project2 개발 환경 설치
helm install project2-dev ./project2 -f ./project2/values/dev.yaml
```

### 운영 환경 설치
```bash
# project1 운영 환경 설치
helm install project1-prod ./project1 -f ./project1/values/prod.yaml

# project2 운영 환경 설치
helm install project2-prod ./project2 -f ./project2/values/prod.yaml
```

## 2. 설정 관리

### Filebeat 설정
Filebeat 설정은 다음과 같은 순서로 우선순위가 적용됩니다:
1. 환경별 설정 (`values/dev.yaml` 또는 `values/prod.yaml`)
2. 프로젝트별 설정 (`values.yaml`)
3. 공통 설정 (`common/values.yaml`)

#### Filebeat 활성화/비활성화
```yaml
# values.yaml 또는 환경별 values 파일에서
filebeat:
  enabled: true  # 또는 false
```

#### Elasticsearch 설정 변경
```yaml
# values.yaml 또는 환경별 values 파일에서
filebeat:
  elasticsearch:
    host: "your-elasticsearch:9200"
    username: "your-username"
    password: "your-password"
```

## 3. 주의사항

### 보안
- Elasticsearch 접속 정보는 Secret으로 관리됩니다.
- 프로덕션 환경에서는 반드시 기본 비밀번호를 변경해야 합니다.
- 민감한 정보는 values 파일에 직접 작성하지 않고, Secret으로 관리하는 것을 권장합니다.

### 리소스 관리
- 개발 환경과 운영 환경의 리소스 요청/제한이 다르게 설정되어 있습니다.
- 운영 환경에서는 더 많은 리소스가 할당됩니다.
- 필요에 따라 `values.yaml`에서 리소스 설정을 조정할 수 있습니다.

### 로그 관리
- Filebeat는 sidecar 패턴으로 배포됩니다.
- 각 애플리케이션의 로그는 독립적인 인덱스로 저장됩니다.
- 로그 인덱스 이름은 `{프로젝트명}-{날짜}` 형식입니다.

## 4. 문제 해결

### 로그 수집 문제
1. Filebeat Pod 상태 확인
```bash
kubectl get pods -n {네임스페이스} -l app.kubernetes.io/name={프로젝트명}
```

2. Filebeat 로그 확인
```bash
kubectl logs -n {네임스페이스} {파드명} -c filebeat
```

3. ConfigMap과 Secret 확인
```bash
kubectl get configmap,secret -n {네임스페이스} -l app.kubernetes.io/name={프로젝트명}
```

### 설정 변경 후
1. ConfigMap과 Secret 업데이트 확인
```bash
kubectl describe configmap {configmap명} -n {네임스페이스}
kubectl describe secret {secret명} -n {네임스페이스}
```

2. Filebeat Pod 재시작
```bash
kubectl rollout restart deployment {배포명} -n {네임스페이스}
```

## 5. 유지보수

### 차트 업그레이드
```bash
# 차트 의존성 업데이트
helm dependency update ./project1
helm dependency update ./project2

# 차트 업그레이드
helm upgrade {릴리즈명} ./{차트경로} -f ./{차트경로}/values/{환경}.yaml
```

### 차트 제거
```bash
helm uninstall {릴리즈명} -n {네임스페이스}
```
