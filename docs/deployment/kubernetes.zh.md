# Kubernetes 部署指南

在 Kubernetes 上部署 Ralph Orchestrator，实现可扩展、高韧性的 AI 编排。

## 前置要求

- Kubernetes 集群 1.20+（本地或云）
- `kubectl` 已配置集群访问权限
- Helm 3.0+（可选，用于 Helm 部署）
- 容器镜像仓库访问权限（Docker Hub、GCR、ECR 等）
- 最少 2 个节点，每个节点 4GB 内存

## 快速开始

### 使用 kubectl 进行基础部署

创建命名空间并部署：

```bash
# 创建命名空间
kubectl create namespace ralph-orchestrator

# 应用清单文件
kubectl apply -f k8s/ -n ralph-orchestrator

# 检查部署状态
kubectl get pods -n ralph-orchestrator
```

## Kubernetes 清单文件

### 1. 命名空间和 ConfigMap

```yaml
# k8s/00-namespace.yaml
apiVersion: v1
kind: Namespace
metadata:
  name: ralph-orchestrator
---
# k8s/01-configmap.yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: ralph-config
  namespace: ralph-orchestrator
data:
  RALPH_AGENT: "auto"
  RALPH_MAX_ITERATIONS: "100"
  RALPH_MAX_RUNTIME: "14400"
  RALPH_CHECKPOINT_INTERVAL: "5"
  RALPH_VERBOSE: "true"
  RALPH_ENABLE_METRICS: "true"
```

### 2. 密钥管理

```yaml
# k8s/02-secrets.yaml
apiVersion: v1
kind: Secret
metadata:
  name: ralph-secrets
  namespace: ralph-orchestrator
type: Opaque
stringData:
  CLAUDE_API_KEY: "sk-ant-..."
  GEMINI_API_KEY: "AIza..."
  Q_API_KEY: "..."
```

从命令行创建密钥：

```bash
# 从字面值创建密钥
kubectl create secret generic ralph-secrets \
  --from-literal=CLAUDE_API_KEY=$CLAUDE_API_KEY \
  --from-literal=GEMINI_API_KEY=$GEMINI_API_KEY \
  --from-literal=Q_API_KEY=$Q_API_KEY \
  -n ralph-orchestrator
```

### 3. 持久化存储

```yaml
# k8s/03-pvc.yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: ralph-workspace
  namespace: ralph-orchestrator
spec:
  accessModes:
    - ReadWriteOnce
  storageClassName: standard
  resources:
    requests:
      storage: 10Gi
---
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: ralph-cache
  namespace: ralph-orchestrator
spec:
  accessModes:
    - ReadWriteMany
  storageClassName: standard
  resources:
    requests:
      storage: 5Gi
```

### 4. 部署

```yaml
# k8s/04-deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: ralph-orchestrator
  namespace: ralph-orchestrator
  labels:
    app: ralph-orchestrator
spec:
  replicas: 1
  selector:
    matchLabels:
      app: ralph-orchestrator
  template:
    metadata:
      labels:
        app: ralph-orchestrator
    spec:
      serviceAccountName: ralph-sa
      containers:
      - name: ralph
        image: ghcr.io/mikeyobrien/ralph-orchestrator:v1.0.0
        imagePullPolicy: Always
        envFrom:
        - configMapRef:
            name: ralph-config
        - secretRef:
            name: ralph-secrets
        resources:
          requests:
            memory: "2Gi"
            cpu: "1"
          limits:
            memory: "4Gi"
            cpu: "2"
        volumeMounts:
        - name: workspace
          mountPath: /workspace
        - name: cache
          mountPath: /app/.cache
        - name: prompts
          mountPath: /prompts
        livenessProbe:
          exec:
            command:
            - python
            - -c
            - "import sys; sys.exit(0)"
          initialDelaySeconds: 30
          periodSeconds: 30
        readinessProbe:
          exec:
            command:
            - python
            - -c
            - "import os; sys.exit(0 if os.path.exists('/app/ralph_orchestrator.py') else 1)"
          initialDelaySeconds: 10
          periodSeconds: 10
      volumes:
      - name: workspace
        persistentVolumeClaim:
          claimName: ralph-workspace
      - name: cache
        persistentVolumeClaim:
          claimName: ralph-cache
      - name: prompts
        configMap:
          name: ralph-prompts
```

### 5. 服务和监控

```yaml
# k8s/05-service.yaml
apiVersion: v1
kind: Service
metadata:
  name: ralph-metrics
  namespace: ralph-orchestrator
  labels:
    app: ralph-orchestrator
spec:
  type: ClusterIP
  ports:
  - port: 8080
    targetPort: 8080
    name: metrics
  selector:
    app: ralph-orchestrator
---
# k8s/06-servicemonitor.yaml
apiVersion: monitoring.coreos.com/v1
kind: ServiceMonitor
metadata:
  name: ralph-orchestrator
  namespace: ralph-orchestrator
spec:
  selector:
    matchLabels:
      app: ralph-orchestrator
  endpoints:
  - port: metrics
    interval: 30s
    path: /metrics
```

### 6. 一次性任务的 Job

```yaml
# k8s/07-job.yaml
apiVersion: batch/v1
kind: Job
metadata:
  name: ralph-task
  namespace: ralph-orchestrator
spec:
  backoffLimit: 3
  activeDeadlineSeconds: 14400
  template:
    spec:
      restartPolicy: Never
      containers:
      - name: ralph
        image: ghcr.io/mikeyobrien/ralph-orchestrator:v1.0.0
        envFrom:
        - configMapRef:
            name: ralph-config
        - secretRef:
            name: ralph-secrets
        args:
        - "--agent=claude"
        - "--prompt=/prompts/task.md"
        - "--max-iterations=50"
        volumeMounts:
        - name: prompts
          mountPath: /prompts
        - name: output
          mountPath: /output
      volumes:
      - name: prompts
        configMap:
          name: ralph-prompts
      - name: output
        emptyDir: {}
```

## Helm Chart 部署

### 使用 Helm 安装

```bash
# 添加仓库
helm repo add ralph https://mikeyobrien.github.io/ralph-orchestrator/charts
helm repo update

# 使用自定义值安装
helm install ralph ralph/ralph-orchestrator \
  --namespace ralph-orchestrator \
  --create-namespace \
  --set apiKeys.claude=$CLAUDE_API_KEY \
  --set apiKeys.gemini=$GEMINI_API_KEY \
  --set config.maxIterations=100
```

### 自定义 values.yaml

```yaml
# values.yaml
replicaCount: 1

image:
  repository: ghcr.io/mikeyobrien/ralph-orchestrator
  tag: v1.0.0
  pullPolicy: IfNotPresent

apiKeys:
  claude: ""
  gemini: ""
  q: ""

config:
  agent: "auto"
  maxIterations: 100
  maxRuntime: 14400
  checkpointInterval: 5
  verbose: true
  enableMetrics: true

resources:
  requests:
    memory: "2Gi"
    cpu: "1"
  limits:
    memory: "4Gi"
    cpu: "2"

persistence:
  enabled: true
  storageClass: "standard"
  workspace:
    size: 10Gi
  cache:
    size: 5Gi

autoscaling:
  enabled: false
  minReplicas: 1
  maxReplicas: 10
  targetCPUUtilizationPercentage: 80

monitoring:
  enabled: true
  serviceMonitor:
    enabled: true
    interval: 30s

ingress:
  enabled: false
  className: "nginx"
  annotations: {}
  hosts:
    - host: ralph.example.com
      paths:
        - path: /
          pathType: Prefix
```

## 水平 Pod 自动扩缩容

```yaml
# k8s/08-hpa.yaml
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: ralph-hpa
  namespace: ralph-orchestrator
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: ralph-orchestrator
  minReplicas: 1
  maxReplicas: 10
  metrics:
  - type: Resource
    resource:
      name: cpu
      target:
        type: Utilization
        averageUtilization: 70
  - type: Resource
    resource:
      name: memory
      target:
        type: Utilization
        averageUtilization: 80
```

## 定时任务的 CronJob

```yaml
# k8s/09-cronjob.yaml
apiVersion: batch/v1
kind: CronJob
metadata:
  name: ralph-daily
  namespace: ralph-orchestrator
spec:
  schedule: "0 2 * * *"  # 每天凌晨 2 点
  jobTemplate:
    spec:
      template:
        spec:
          restartPolicy: OnFailure
          containers:
          - name: ralph
            image: ghcr.io/mikeyobrien/ralph-orchestrator:v1.0.0
            envFrom:
            - configMapRef:
                name: ralph-config
            - secretRef:
                name: ralph-secrets
            args:
            - "--agent=auto"
            - "--prompt=/prompts/daily-task.md"
```

## 服务账户和 RBAC

```yaml
# k8s/10-rbac.yaml
apiVersion: v1
kind: ServiceAccount
metadata:
  name: ralph-sa
  namespace: ralph-orchestrator
---
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  name: ralph-role
  namespace: ralph-orchestrator
rules:
- apiGroups: [""]
  resources: ["configmaps", "secrets"]
  verbs: ["get", "list", "watch"]
- apiGroups: [""]
  resources: ["pods", "pods/log"]
  verbs: ["get", "list", "watch"]
---
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: ralph-rolebinding
  namespace: ralph-orchestrator
roleRef:
  apiVersion: rbac.authorization.k8s.io/v1
  kind: Role
  name: ralph-role
subjects:
- kind: ServiceAccount
  name: ralph-sa
  namespace: ralph-orchestrator
```

## 网络策略

```yaml
# k8s/11-networkpolicy.yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: ralph-network-policy
  namespace: ralph-orchestrator
spec:
  podSelector:
    matchLabels:
      app: ralph-orchestrator
  policyTypes:
  - Ingress
  - Egress
  ingress:
  - from:
    - namespaceSelector:
        matchLabels:
          name: monitoring
    ports:
    - protocol: TCP
      port: 8080
  egress:
  - to:
    - namespaceSelector: {}
    ports:
    - protocol: TCP
      port: 443  # HTTPS 用于 API 调用
    - protocol: TCP
      port: 53   # DNS
    - protocol: UDP
      port: 53   # DNS
```

## 使用 Prometheus 监控

```yaml
# k8s/12-prometheus-config.yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: prometheus-config
  namespace: monitoring
data:
  prometheus.yml: |
    global:
      scrape_interval: 15s
    scrape_configs:
    - job_name: 'ralph-orchestrator'
      kubernetes_sd_configs:
      - role: pod
        namespaces:
          names:
          - ralph-orchestrator
      relabel_configs:
      - source_labels: [__meta_kubernetes_pod_label_app]
        action: keep
        regex: ralph-orchestrator
```

## 云服务商特定配置

### Google Kubernetes Engine (GKE)

```bash
# 创建集群
gcloud container clusters create ralph-cluster \
  --zone us-central1-a \
  --num-nodes 3 \
  --machine-type n1-standard-2

# 获取凭据
gcloud container clusters get-credentials ralph-cluster \
  --zone us-central1-a

# 为 GCR 创建密钥
kubectl create secret docker-registry gcr-json-key \
  --docker-server=gcr.io \
  --docker-username=_json_key \
  --docker-password="$(cat ~/key.json)" \
  -n ralph-orchestrator
```

### Amazon EKS

```bash
# 创建集群
eksctl create cluster \
  --name ralph-cluster \
  --region us-west-2 \
  --nodegroup-name workers \
  --node-type t3.medium \
  --nodes 3

# 更新 kubeconfig
aws eks update-kubeconfig \
  --name ralph-cluster \
  --region us-west-2
```

### Azure AKS

```bash
# 创建集群
az aks create \
  --resource-group ralph-rg \
  --name ralph-cluster \
  --node-count 3 \
  --node-vm-size Standard_DS2_v2

# 获取凭据
az aks get-credentials \
  --resource-group ralph-rg \
  --name ralph-cluster
```

## 使用 ArgoCD 进行 GitOps

```yaml
# k8s/argocd-app.yaml
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: ralph-orchestrator
  namespace: argocd
spec:
  project: default
  source:
    repoURL: https://github.com/mikeyobrien/ralph-orchestrator
    targetRevision: HEAD
    path: k8s
  destination:
    server: https://kubernetes.default.svc
    namespace: ralph-orchestrator
  syncPolicy:
    automated:
      prune: true
      selfHeal: true
```

## 故障排查

### 检查 Pod 状态

```bash
# 获取 Pod 列表
kubectl get pods -n ralph-orchestrator

# 查看 Pod 详情
kubectl describe pod <pod-name> -n ralph-orchestrator

# 查看日志
kubectl logs -f <pod-name> -n ralph-orchestrator

# 进入 Pod 执行命令
kubectl exec -it <pod-name> -n ralph-orchestrator -- /bin/bash
```

### 常见问题

#### ImagePullBackOff

```bash
# 检查镜像拉取密钥
kubectl get secrets -n ralph-orchestrator

# 创建拉取密钥
kubectl create secret docker-registry regcred \
  --docker-server=ghcr.io \
  --docker-username=USERNAME \
  --docker-password=TOKEN \
  -n ralph-orchestrator
```

#### PVC 未绑定

```bash
# 检查 PVC 状态
kubectl get pvc -n ralph-orchestrator

# 检查可用的存储类
kubectl get storageclass

# 如需要，创建 PV
kubectl apply -f persistent-volume.yaml
```

#### OOMKilled

```bash
# 增加内存限制
kubectl set resources deployment ralph-orchestrator \
  --limits=memory=8Gi \
  -n ralph-orchestrator
```

## 最佳实践

1. **使用命名空间** 隔离 Ralph 部署
2. **实施 RBAC** 实现最小权限访问
3. **使用密钥管理**（Sealed Secrets、External Secrets）
4. **设置资源限制** 防止资源耗尽
5. **启用监控** 使用 Prometheus/Grafana
6. **使用网络策略** 增强安全性
7. **实施健康检查** 实现自动恢复
8. **使用 GitOps** 进行声明式部署
9. **定期备份** 持久化卷数据
10. **使用 Pod 中断预算** 实现高可用性

## 生产环境考虑事项

- **高可用性**：跨多个可用区部署
- **灾难恢复**：定期备份和跨区域复制
- **安全性**：Pod 安全策略、网络策略、RBAC
- **可观测性**：日志（ELK）、指标（Prometheus）、链路追踪（Jaeger）
- **成本优化**：使用竞价实例、自动扩缩容、资源配额
- **合规性**：审计日志、静态和传输加密

## 后续步骤

- [CI/CD 集成](ci-cd.md) - 自动化 Kubernetes 部署
- [生产指南](production.md) - 生产环境最佳实践
- [监控设置](../advanced/monitoring.md) - 完整可观测性
