# Metallb+traefik+Https

해당 서비스들의 기능들의 대해 소개합니다.

- Metallb: 로드밸런서, 외부에서 Metallb의 VIP 을 통해서 서비스를 접근하게 한다.
- Traefik: ingressroute 서비스, Prefix 별 라우팅을 지원한다. 대시보드 UI를 지원한다.

해당 구성의 동작은 아래와 같습니다.

- 사용자 -> 접근 https://VIP/apple
- Metallb -> traefik -> nginx

해당 구성들을 설정하기 전 각 서비스에 대하여 namespace를 생성하길 바랍니다. 

## MetalLB

MetalLB 설치 및 설정 가이드입니다.


### 1. 설치
```
kubectl create namespace metallb-system
helm repo add metallb https://metallb.github.io/metallb
helm repo update
helm install metallb metallb/metallb -n metallb-system
```


### 2. IP 풀 + ARP 광고
```
apiVersion: metallb.io/v1beta1
kind: IPAddressPool
metadata:
  name: default-address-pool
  namespace: metallb-system
spec:
  addresses:
  - # 현재 본인 IP 대역과 동일한 IP range 지정
---
apiVersion: metallb.io/v1beta1
kind: L2Advertisement
metadata:
  name: default
  namespace: metallb-system
```


### 3. kube-proxy IPVS 설정
- ConfigMap 확인
```
kubectl -n kube-system get configmap kube-proxy -o yaml
```

- IPVS 모드 적용
```
mode: ipvs
```

- 각 노드 IPVS 모듈 활성화
```
sudo modprobe ip_vs
sudo modprobe ip_vs_rr
sudo modprobe ip_vs_wrr
sudo modprobe ip_vs_sh
sudo modprobe nf_conntrack
```

- 재부팅 후 자동 로드 가능하게 설정
```
sudo tee /etc/modules-load.d/ipvs.conf <<EOF
ip_vs
ip_vs_rr
ip_vs_wrr
ip_vs_sh
nf_conntrack
EOF
```

- kube-proxy 재시작
```
sudo tee /etc/modules-load.d/ipvs.conf <<EOF
ip_vs
ip_vs_rr
ip_vs_wrr
ip_vs_sh
nf_conntrack
EOF
```


## Traefik

Traefik Ingress Controller 설치 및 Dashboard 설정 가이드입니다.


### 설치
```
kubectl create namespace traefik
helm repo add traefik https://traefik.github.io/charts
helm repo update
helm install traefik traefik/traefik -f my-values.yaml -n traefik
```


### values.yaml 예시
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


## Nginx + HTTPS

Traefik과 연동된 Nginx 배포 및 HTTPS 설정 가이드입니다.


### 1. Cert-Manager 설치
```
kubectl apply -f https://github.com/cert-manager/cert-manager/releases/download/v1.12.0/cert-manager.yaml
```


### 2. Self-Signed Issuer 생성
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

### 3. Nginx 배포 및 ConfigMap 연결

- 설정 파일 참고
- Deployment와 Service 설정
- Traefik IngressRoute + Middleware 연결
  
