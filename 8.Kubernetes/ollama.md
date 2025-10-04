# 虚拟机部署运行Ollama

## 部署方案

### 1. 存储PVC

```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: ollama-pvc
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 8Gi
```

### 2. Ollama Deployment

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: ollama
spec:
  replicas: 1
  selector:
    matchLabels:
      app: ollama
  template:
    metadata:
      labels:
        app: ollama
    spec:
      containers:
        - name: ollama
          image: ollama/ollama:latest
          ports:
            - containerPort: 11434
          volumeMounts:
            - mountPath: /root/.ollama
              name: ollama-storage
      volumes:
        - name: ollama-storage
          persistentVolumeClaim:
            claimName: ollama-pvc
```

### 3. Service

```yaml
apiVersion: v1
kind: Service
metadata:
  name: ollama
spec:
  selector:
    app: ollama
  ports:
    - port: 11434
      targetPort: 11434
      nodePort: 31134    # 宿主机访问的端口
  type: NodePort
---
apiVersion: v1
kind: Service
metadata:
  name: openwebui
spec:
  selector:
    app: openwebui
  ports:
    - port: 8080
      targetPort: 8080
  type: NodePort
```

### 4. Open-WebUI Deployment

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: openwebui
spec:
  replicas: 1
  selector:
    matchLabels:
      app: openwebui
  template:
    metadata:
      labels:
        app: openwebui
    spec:
      containers:
        - name: openwebui
          image: ghcr.io/open-webui/open-webui:main
          imagePullPolicy: Never
          ports:
            - containerPort: 8080
          env:
            - name: OLLAMA_API_BASE
              value: "http://ollama:11434"
```

### 5. PV

```yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: ollama-pv
spec:
  capacity:
    storage: 8Gi
  accessModes:
    - ReadWriteOnce
  hostPath:
    path: /mnt/data/ollama    # 请确认该目录存在且有权限
  persistentVolumeReclaimPolicy: Retain
```

## 启动（按顺序）

```sh
kubectl apply -f ollama-pv.yaml       # PV
kubectl apply -f cunchu.yaml          # PVC
kubectl apply -f ai.yaml              # Ollama
kubectl apply -f web.yaml             # Open-WebUI
kubectl apply -f open.yaml            # Services
```

## 项目展示

![image-20250813160636366](https://img.lhjeong.cn/20250813160636479.png)