---
title: 部署 cert-manager
date: 2020-12-12 19:00:00
tags: '部署'
categories:
  - ['部署', 'k8s']
permalink: deploy-cert-manager
photo:
---

https://cert-manager.io/docs/installation/kubernetes/
https://feisky.gitbooks.io/kubernetes/content/practice/ingress_letsencrypt.html
https://learnku.com/articles/42528
https://cloud.tencent.com/document/product/457/49368
https://cert-manager.io/docs/tutorials/acme/ingress/

```sh
kubectl create namespace cert-manager
helm repo add jetstack https://charts.jetstack.io
helm repo update
# Kubernetes 1.15+
$ kubectl apply -f https://github.com/jetstack/cert-manager/releases/download/v1.1.0/cert-manager.crds.yaml

# Kubernetes <1.15
$ kubectl apply --validate=false -f https://github.com/jetstack/cert-manager/releases/download/v1.1.0/cert-manager-legacy.crds.yaml

# Helm v3+
$ helm install \
  cert-manager jetstack/cert-manager \
  --namespace cert-manager \
  --version v1.1.0 \
  # --set installCRDs=true

# Helm v2
$ helm install \
  --name cert-manager \
  --namespace cert-manager \
  --version v1.1.0 \
  jetstack/cert-manager \
  # --set installCRDs=true

$ cat <<EOF > test-resources.yaml
apiVersion: v1
kind: Namespace
metadata:
  name: cert-manager-test
---
apiVersion: cert-manager.io/v1
kind: Issuer
metadata:
  name: test-selfsigned
  namespace: cert-manager-test
spec:
  selfSigned: {}
---
apiVersion: cert-manager.io/v1
kind: Certificate
metadata:
  name: selfsigned-cert
  namespace: cert-manager-test
spec:
  dnsNames:
    - example.com
  secretName: selfsigned-cert-tls
  issuerRef:
    name: test-selfsigned
EOF
kubectl apply -f test-resources.yaml
kubectl describe certificate -n cert-manager-test
kubectl delete -f test-resources.yaml
```

```yaml
# cluster-issuer-letsencrypt-staging.yaml
apiVersion: cert-manager.io/v1
kind: ClusterIssuer
metadata:
  name: letsencrypt-staging
spec:
  acme:
    # 务必将此处替换为你自己的邮箱, 否则会配置失败。当证书快过期时 Let's Encrypt 会与你联系
    email: yanjinzh6@gmail.com
    server: https://acme-staging-v02.api.letsencrypt.org/directory
    privateKeySecretRef:
      # 将用来存储 Private Key 的 Secret 资源
      name: letsencrypt-staging
    # Add a single challenge solver, HTTP01 using nginx
    solvers:
    - http01:
        ingress:
          class: nginx
```

```yaml
apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  name: kuard
  annotations:
    # 务必添加以下两个注解, 指定 ingress 类型及使用哪个 cluster-issuer
    kubernetes.io/ingress.class: "nginx"
    cert-manager.io/cluster-issuer："letsencrypt-staging"

    # 如果你使用 issuer, 使用以下注解
    # cert-manager.io/issuer: "letsencrypt-staging"

spec:
  tls:
  - hosts:
    - example.example.com                # TLS 域名
    secretName: quickstart-example-tls   # 用于存储证书的 Secret 对象名字
  rules:
  - host: example.example.com
    http:
      paths:
      - path: /
        backend:
          serviceName: kuard
          servicePort: 80
```
