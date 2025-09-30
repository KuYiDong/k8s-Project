# Openebs + DB 

해당 서비스들의 기능들의 대해 소개합니다.

- **Openebs**: Kubernetes 내부에서 동적 스토리지 프로비저닝을 제공하는 Cloud Native Storage(CNS) 솔루션, StorageClass를 생성하여 자동으로 PV를 할당한
- **Stateful**: 상태가 필요한(Stateful) 애플리케이션을 안정적으로 배포/운영하기 위한 리소스, 데이터 베이스마다 고유의 PVC 할당 
- **headless**: Pod들을 개별적으로 DNS 이름으로 직접 접근 가능하게 해주는 서비스 타입


해당 구성들을 설정하기 전 각 서비스에 대하여 namespace를 생성하길 바랍니다. 


## Openebs

### 1.설치 

```
kubectl create ns openebs
helm repo add openebs https://openebs.github.io/charts
helm repo update
helm install openebs openebs/openebs -n openebs
```

### 2.확인

```
kubectl get sc
openebs-device     openebs.io/local   Delete          WaitForFirstConsumer   false                  12s
openebs-hostpath   openebs.io/local   Delete          WaitForFirstConsumer   false                  12s
```

### 3.기본 StorageClass로 변경

'''
kubectl patch storageclass openebs-hostpath -p '{"metadata": {"annotations":{"storageclass.kubernetes.io/is-default-class":"true"}}}'
'''

## DataBase

### 1.ConfigMap

```
apiVersion: v1
kind: ConfigMap
metadata:
  name: db-configmap
data:
  MYSQL_ROOT_USER: root
  MYSQL_USER: user
```

### 2.Secret

```
apiVersion: v1
kind: Secret
metadata:
  name: db-secret
type: Opaque
stringData:
  MYSQL_PASSWORD: mypass
  MYSQL_ROOT_PASSWORD: rootpass
```

### 3.Stateful

```
apiVersion: apps/v1
kind: StatefulSet
metadata:
  creationTimestamp: null
  labels:
    app: db-deployment
  name: db-deployment
spec:
  replicas: 2
  selector:
    matchLabels:
      app: db-deployment
  template:
    metadata:
      creationTimestamp: null
      labels:
        app: db-deployment
    spec:
      containers:
      - image: mysql
        name: mysql
        ports:
        - containerPort: 3306
        envFrom:
        - configMapRef:
            name: db-configmap
        - secretRef:
            name: db-secret
        volumeMounts:
        - name: db-storage
          mountPath: /var/lib/mysql
  volumeClaimTemplates:
  - metadata:
      name: db-storage
    spec:
      accessModes: [ "ReadWriteOnce" ]
      storageClassName: openebs-hostpath
      resources:
        requests:
          storage: 4Gi
```

### 4.Headless

```
apiVersion: v1
kind: Service
metadata:
  name: db-service
  labels:
    app: db-deployment
spec:
  ports:
  - port: 3306
    name: mysql
  clusterIP: None   # 핵심
  selector:
    app: db-deployment
```





