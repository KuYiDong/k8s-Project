# Metallb+traefik+Https

# MetalLB

MetalLB 설치 및 설정 가이드입니다.

## 1. 설치
```
helm repo add metallb https://metallb.github.io/metallb
helm repo update
kubectl create namespace metallb-system
helm install metallb metallb/metallb -n metallb-system
```

## 2. IP 풀 + ARP 광고
```
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
```

## 3. kube-proxy IPVS 설정
- ConfigMap 확인
```
kubectl -n kube-system get configmap kube-proxy -o yaml
```

- IPVS 모드 적용

# Traefik

Traefik Ingress Controller 설치 및 Dashboard 설정 가이드입니다.

## 설치
```
helm repo add traefik https://traefik.github.io/charts
helm repo update
kubectl create namespace traefik
helm install traefik traefik/traefik -f my-values.yaml
```

## values.yaml 예시
```
service:
  type: LoadBalancer
  single: true

ingressRoute:
  dashboard:
    enabled: true

Deployment:
  type: DaemonSet
```

## 서비스 배포
```
helm install traefik -f value.yaml .
```

# Nginx + HTTPS

Traefik과 연동된 Nginx 배포 및 HTTPS 설정 가이드입니다.

## 1. Cert-Manager 설치
```
kubectl apply -f https://github.com/cert-manager/cert-manager/releases/download/v1.12.0/cert-manager.yaml
```

## 2. Self-Signed Issuer 생성
```
apiVersion: cert-manager.io/v1
kind: Issuer
metadata:
  name: selfsigned-issuer
  namespace: traefik
spec:
  selfSigned: {}
---
apiVersion: cert-manager.io/v1
kind: Certificate
metadata:
  name: traefik-cert
  namespace: traefik
spec:
  secretName: traefik-tls
  issuerRef:
    name: selfsigned-issuer
    kind: Issuer
  commonName: # VIP 
  dnsNames:
    - # VIP
```

## 3. Nginx 배포 및 ConfigMap 연결
- apple.html, banana.html, cherry.html 생성
- Deployment와 Service 설정
- Traefik IngressRoute + Middleware 연결
EOF
