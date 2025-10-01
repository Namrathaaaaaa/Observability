# ArgoCD Intermediate Concepts

## 1. Reconciliation Loop

ArgoCD continuously monitors and synchronizes the desired state (Git) with actual state (Kubernetes cluster).

### How it works:

- **Poll Interval**: Default 3 minutes, configurable
- **Drift Detection**: Compares Git manifest with live cluster state
- **Auto-sync**: Automatically applies changes when enabled
- **Manual Sync**: Requires user intervention when auto-sync is disabled

```yaml
# Application with reconciliation settings
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: my-app
spec:
  syncPolicy:
    automated:
      prune: true
      selfHeal: true
    syncOptions:
      - CreateNamespace=true
```

## 2. GitHub Webhooks

Webhooks enable real-time synchronization instead of waiting for polling intervals.

### Setup:

1. **Configure webhook in GitHub repository**
2. **Point to ArgoCD webhook endpoint**
3. **Use shared secret for security**

```bash
# Webhook URL format
https://argocd-server.example.com/api/webhook

# GitHub webhook configuration
Payload URL: https://argocd-server.example.com/api/webhook
Content type: application/json
Secret: <shared-secret>
Events: push, pull_request
```

### ArgoCD Configuration:

```yaml
# argocd-server configmap
apiVersion: v1
kind: ConfigMap
metadata:
  name: argocd-server-config
data:
  webhooks.github.secret: <base64-encoded-secret>
```

## 3. Custom Health Checks

Define custom health assessments for resources ArgoCD doesn't natively understand.

### Example - Custom CRD Health Check:

```lua
-- healthcheck.lua
health_status = {}
if obj.status ~= nil then
  if obj.status.phase == "Running" then
    health_status.status = "Healthy"
    health_status.message = "Custom resource is running"
  elseif obj.status.phase == "Failed" then
    health_status.status = "Degraded"
    health_status.message = obj.status.message
  else
    health_status.status = "Progressing"
    health_status.message = "Custom resource is starting"
  end
end
return health_status
```

### ConfigMap for Custom Health:

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: argocd-cm
data:
  resource.customizations.health.example.com_MyCustomResource: |
    health_status = {}
    if obj.status ~= nil then
      if obj.status.phase == "Running" then
        health_status.status = "Healthy"
      else
        health_status.status = "Progressing"
      end
    end
    return health_status
```

## 4. Sync Strategies

Different approaches to synchronize applications.

### Sync Options:

```yaml
spec:
  syncPolicy:
    syncOptions:
      - CreateNamespace=true # Create namespace if missing
      - PrunePropagationPolicy=foreground # Deletion order
      - PruneLast=true # Prune after sync
      - RespectIgnoreDifferences=true # Honor ignore rules
      - ApplyOutOfSyncOnly=true # Only sync out-of-sync resources
```

### Sync Phases:

1. **PreSync**: Jobs that run before sync (DB migrations)
2. **Sync**: Main application deployment
3. **PostSync**: Jobs that run after sync (tests, notifications)

```yaml
# PreSync hook example
apiVersion: batch/v1
kind: Job
metadata:
  name: db-migration
  annotations:
    argocd.argoproj.io/hook: PreSync
    argocd.argoproj.io/hook-delete-policy: BeforeHookCreation
spec:
  template:
    spec:
      containers:
        - name: migrate
          image: migrate/migrate
          command:
            [
              "migrate",
              "-path",
              "/migrations",
              "-database",
              "postgres://...",
              "up",
            ]
      restartPolicy: Never
```

### Sync Windows:

```yaml
# Restrict sync to maintenance windows
spec:
  syncPolicy:
    syncOptions:
      - SkipDryRunOnMissingResource=true
  syncWindows:
    - kind: allow
      schedule: "0 2 * * 1-5" # 2 AM, Mon-Fri
      duration: 1h
      applications:
        - "*"
```

## 5. Pruning and Self-Healing

### Pruning:

Removes resources that exist in cluster but not in Git.

```yaml
spec:
  syncPolicy:
    automated:
      prune: true # Enable automatic pruning
    syncOptions:
      - PruneLast=true # Prune after successful sync
```

### Self-Healing:

Reverts manual changes to match Git state.

```yaml
spec:
  syncPolicy:
    automated:
      selfHeal: true # Enable self-healing
    retry:
      limit: 3
      backoff:
        duration: 5s
        factor: 2
        maxDuration: 3m
```

### Example with both enabled:

```yaml
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: web-app
spec:
  syncPolicy:
    automated:
      prune: true
      selfHeal: true
    retry:
      limit: 5
      backoff:
        duration: 5s
        maxDuration: 10m
```

## 6. Helm Charts in ArgoCD

ArgoCD natively supports Helm charts with GitOps workflow.

### Direct Helm Chart:

```yaml
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: nginx-helm
spec:
  source:
    repoURL: https://kubernetes.github.io/ingress-nginx
    chart: ingress-nginx
    targetRevision: 4.7.1
    helm:
      parameters:
        - name: controller.service.type
          value: LoadBalancer
        - name: controller.metrics.enabled
          value: "true"
      values: |
        controller:
          replicas: 2
          resources:
            requests:
              cpu: 100m
              memory: 128Mi
```

### Helm Chart from Git:

```yaml
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: my-app-helm
spec:
  source:
    repoURL: https://github.com/myorg/myapp
    path: helm-chart
    targetRevision: HEAD
    helm:
      valueFiles:
        - values-prod.yaml
      parameters:
        - name: image.tag
          value: v1.2.3
```

### Helm Hooks with ArgoCD:

```yaml
# Pre-install job
apiVersion: batch/v1
kind: Job
metadata:
  name: pre-install-job
  annotations:
    "helm.sh/hook": pre-install
    "argocd.argoproj.io/hook": PreSync
spec:
  template:
    spec:
      containers:
        - name: setup
          image: busybox
          command: ["sh", "-c", "echo 'Setting up environment'"]
      restartPolicy: Never
```

## 7. Multi-Cluster Management

Manage applications across multiple Kubernetes clusters.

### Add External Cluster:

```bash
# Login to ArgoCD
argocd login argocd-server.example.com

# Add cluster
argocd cluster add prod-cluster-context --name production

# List clusters
argocd cluster list
```

### Multi-Cluster Application:

```yaml
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: web-app-prod
  namespace: argocd
spec:
  destination:
    server: https://prod-cluster-api-server.com
    namespace: production
  source:
    repoURL: https://github.com/myorg/web-app
    path: k8s/overlays/production
    targetRevision: HEAD
```

### ApplicationSet for Multi-Cluster:

```yaml
apiVersion: argoproj.io/v1alpha1
kind: ApplicationSet
metadata:
  name: multi-cluster-apps
spec:
  generators:
    - clusters:
        selector:
          matchLabels:
            environment: production
  template:
    metadata:
      name: "{{name}}-web-app"
    spec:
      destination:
        server: "{{server}}"
        namespace: web-app
      source:
        repoURL: https://github.com/myorg/web-app
        path: "k8s/overlays/{{metadata.labels.environment}}"
        targetRevision: HEAD
      syncPolicy:
        automated:
          prune: true
          selfHeal: true
```

### Cluster Configuration with Labels:

```bash
# Add cluster with labels
argocd cluster add staging-context \
  --name staging \
  --label environment=staging \
  --label region=us-east-1

argocd cluster add prod-context \
  --name production \
  --label environment=production \
  --label region=us-west-2
```

## Quick Reference Commands

```bash
# Application management
argocd app create myapp --repo https://github.com/myorg/myapp --path k8s --dest-server https://kubernetes.default.svc --dest-namespace default
argocd app sync myapp
argocd app delete myapp

# Manual sync with options
argocd app sync myapp --prune --force

# Cluster management
argocd cluster add my-cluster-context
argocd cluster list
argocd cluster rm https://cluster-api-server.com

# Check application health
argocd app get myapp
argocd app history myapp
argocd app rollback myapp 5
```

## Best Practices

1. **Use Git branches for environments** (dev/staging/prod)
2. **Enable pruning carefully** - test in non-prod first
3. **Set up proper RBAC** for multi-cluster access
4. **Use ApplicationSets** for managing multiple similar apps
5. **Configure webhooks** for faster sync cycles
6. **Monitor sync status** and set up alerts
7. **Use sync windows** for production deployments
8. **Test custom health checks** thoroughly before production use
