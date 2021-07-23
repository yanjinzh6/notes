---
title: 部署 kubernetes dashboard UI
date: 2020-12-12 21:00:00
tags: '部署'
categories:
  - ['部署', 'k8s']
permalink: deploy-kubernetes-dashboard
photo:
---

https://artifacthub.io/packages/helm/k8s-dashboard/kubernetes-dashboard
https://www.cnblogs.com/baoshu/p/13326480.html
https://kuboard.cn/guide/#kuboard-%E7%9A%84%E8%AE%BE%E8%AE%A1%E7%9B%AE%E6%A0%87
https://github.com/kubernetes/dashboard/blob/master/docs/user/access-control/README.md

```yaml
image:
  repository: kubernetesui/dashboard
  tag: v2.0.3
  pullPolicy: IfNotPresent
  pullSecrets: []
replicaCount: 1
annotations: {}
labels: {}
extraEnv: []
podAnnotations:
  seccomp.security.alpha.kubernetes.io/pod: 'runtime/default'
nodeSelector: {}
tolerations: []
affinity: {}

resources:
  requests:
    cpu: 100m
    memory: 200Mi
  limits:
    cpu: 2
    memory: 200Mi
protocolHttp: false

service:
  type: ClusterIP
  externalPort: 443
  annotations: {}
  labels: {}

ingress:
  enabled: true
  annotations:
    nginx.ingress.kubernetes.io/secure-backends: "true"
    nginx.ingress.kubernetes.io/backend-protocol: "HTTPS"
  paths:
    - /
  customPaths: []
  hosts:
    - d.361163.xyz # 你的域名
  tls:
    # 注意这个名字要跟前面新建的 secret 对上
    - secretName: dashboard-tls
      hosts:
        - d.361163.xyz # 你的域名

metricsScraper:
  enabled: false
  image:
    repository: kubernetesui/metrics-scraper
    tag: v1.0.4
  resources: {}
  containerSecurityContext:
    allowPrivilegeEscalation: false
    readOnlyRootFilesystem: true
    runAsUser: 1001
    runAsGroup: 2001

metrics-server:
  enabled: false

rbac:
  create: true
  clusterRoleMetrics: true
  clusterReadOnlyRole: false

serviceAccount:
  create: true
  name:

livenessProbe:
  initialDelaySeconds: 30
  timeoutSeconds: 30

podDisruptionBudget:
  enabled: false
  minAvailable:
  maxUnavailable:

containerSecurityContext:
  allowPrivilegeEscalation: false
  readOnlyRootFilesystem: true
  runAsUser: 1001
  runAsGroup: 2001

networkPolicy:
  enabled: false
```

```sh
kubectl describe secret $(kubectl get secret -n kube-system | grep kubernetes-dashboard-token | awk '{print $1}') -n kube-system
kubectl describe secret $(kubectl get secret | grep kubernetes-dashboard-token | awk '{print $1}')
```
