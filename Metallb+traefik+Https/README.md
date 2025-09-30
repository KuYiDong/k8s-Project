# Metallb+traefik+Https

MetalLB 설치 및 설정 가이드입니다.

## 1. 설치
\`\`\`bash
helm repo add metallb https://metallb.github.io/metallb
helm repo update
kubectl create namespace metallb-system
helm install metallb metallb/metallb -n metallb-system
\`\`\`

## 2. IP 풀 + ARP 광고
\`\`\`yaml
apiVersion: metallb.io/v1beta1
kind: IPAddressPool
metadata:
  name: default-address-pool
  namespace: metallb-system
spec:
  addresses:
  - 192.168.163.100-192.168.163.200
---
apiVersion: metallb.io/v1beta1
kind: L2Advertisement
metadata:
  name: default
  namespace: metallb-system
\`\`\`

## 3. kube-proxy IPVS 설정
- ConfigMap 확인
\`\`\`bash
kubectl -n kube-system get configmap kube-proxy -o yaml
\`\`\`

- IPVS 모드 적용, 노드 모듈 활성화 등은 기존 스크립트 참고
EOF

# 4️⃣ Traefik README.md
cat > Traefik/README.md << 'EOF'
# Traefik

Traefik Ingress Controller 설치 및 Dashboard 설정 가이드입니다.

## 설치
\`\`\`bash
helm repo add traefik https://traefik.github.io/charts
helm repo update
kubectl create namespace traefik
helm install traefik traefik/traefik -f my-values.yaml
\`\`\`

## values.yaml 예시
\`\`\`yaml
service:
  type: LoadBalancer
  single: true

ingressRoute:
  dashboard:
    enabled: true

Deployment:
  type: DaemonSet
\`\`\`

## 서비스 배포
- Nginx App1/2 생성 및 서비스 노출
- ConfigMap을 사용한 index.html 연결
EOF

# 5️⃣ Nginx-Https README.md
cat > Nginx-Https/README.md << 'EOF'
# Nginx + HTTPS

Traefik과 연동된 Nginx 배포 및 HTTPS 설정 가이드입니다.

## 1. Cert-Manager 설치
\`\`\`bash
kubectl apply -f https://github.com/cert-manager/cert-manager/releases/download/v1.12.0/cert-manager.yaml
\`\`\`

## 2. Self-Signed Issuer 생성
\`\`\`yaml
apiVersion: cert-manager.io/v1
kind: Issuer
metadata:
  name: selfsigned-issuer
  namespace: traefik
spec:
  selfSigned: {}
\`\`\`

## 3. Nginx 배포 및 ConfigMap 연결
- apple.html, banana.html, cherry.html 생성
- Deployment와 Service 설정
- Traefik IngressRoute + Middleware 연결
EOF
