# SamplePageExhibition config

`sample.cagea1.com`의 Kubernetes/Kustomize 배포 설정입니다. 기존 Argo CD 앱은 `overlays/prod`를 자동 동기화합니다.

```sh
kubectl kustomize overlays/prod
```

- 소스: https://github.com/taeboranger/SamplePageExhibition
- 이미지: `lightstone1/sample-page-exhibition-prod` (검증된 커밋 SHA 태그)
- namespace: `sample-page-exhibition-prod`
- ingress class: `nginx`
- TLS issuer: 기존 `letsencrypt-clusterissuer`
- Node 런타임은 UID/GID 1000, 읽기 전용 파일 시스템으로 실행합니다.

## Vault 비밀번호

기존 `vault-secrets-operator-system/cagea1-vault` 연결과 `config-vso` Kubernetes 인증 역할을 그대로 사용합니다.

```text
Vault kv/data/sample-page-exhibition-prod-secret
  site-password
      ↓ VaultStaticSecret 변환
Kubernetes Secret sample-page-exhibition-prod-secret
  SITE_PASSWORD
      ↓ envFrom
Deployment sample-page-exhibition-deployment
```

KV v2 설정은 `mount: kv`, `path: sample-page-exhibition-prod-secret`입니다. path에 `data/`를 넣지 않습니다. 원본 비밀 필드는 제외하고 `SITE_PASSWORD`만 내보냅니다. 키가 비어 있으면 동기화를 실패 처리합니다.

리소스 적용 순서는 namespace(-4), ServiceAccount/VaultAuth(-3), VaultStaticSecret(-2), 앱(0)입니다. 비밀번호 변경은 VSO의 1시간 주기 동기화 후 자동 롤아웃됩니다. 서버 재시작 시 로그인 세션도 만료됩니다. 앱 세션은 메모리 기반이므로 replicas=1을 유지합니다.

현재 홈서버의 기존 config-vso 역할은 새 서비스 계정과 경로를 허용하므로 Vault 정책을 수정할 필요가 없습니다. 다른 클러스터에서는 해당 역할이 `sample-page-exhibition-vault-auth` 및 `sample-page-exhibition-prod` namespace를 허용하고, 연결된 정책에 아래 읽기 권한이 있어야 합니다. 기존 역할/정책을 덮어쓰지 말고 필요한 권한을 보존하세요.

```hcl
path "kv/data/sample-page-exhibition-prod-secret" {
  capabilities = ["read"]
}
```

## 이미지 갱신

소스의 `PROD` 워크플로가 테스트 통과 후 Docker Hub에 이미지를 올립니다. 소스 저장소의 `ACTION_TOKEN`에 config 저장소 쓰기 권한이 있으면 이미지 태그도 자동 갱신합니다. 없으면 게시된 SHA를 이 저장소의 `overlays/prod/sample-page-exhibition/kustomization.yaml`에 기록하고 커밋합니다. 이미지가 게시된 뒤 config를 동기화하세요.
