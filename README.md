# SamplePageExhibition config

`sample.cagea1.com`의 Kubernetes/Kustomize 배포 설정입니다.

```sh
kubectl kustomize overlays/prod
kubectl apply -k overlays/prod
```

- 소스: https://github.com/taeboranger/SamplePageExhibition
- 컨테이너: `lightstone1/sample-page-exhibition-prod`
- namespace: `sample-page-exhibition-prod`
- ingress class: `nginx`
- TLS issuer: 기존 `letsencrypt-clusterissuer`

DNS 레코드를 기존 ingress 주소로 연결하고, 소스 저장소의 `PROD` 워크플로를 먼저 실행해 이미지를 업로드합니다. 이 저장소를 Argo CD에 연결하거나 위 apply 명령을 실행하면 됩니다. 소스 워크플로가 `overlays/prod/sample-page-exhibition/kustomization.yaml`의 이미지 태그를 커밋 SHA로 바꿉니다.

저장소는 private입니다. Argo CD에는 이 저장소를 읽을 수 있는 인증이 필요합니다. Docker Hub 이미지를 비공개로 설정하면 namespace에 imagePullSecret을 만들고 Deployment에 연결하세요. 기본 설정에는 앱 비밀값, 볼륨 저장소, Vault 연결이 필요 없습니다.
