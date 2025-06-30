# Argo Workflows セットアップ

## 前提条件
- Docker
- kubectl
- kind
- nginx ingress controller

## 環境立ち上げ

### 1. kindクラスタの作成
```bash
kind create cluster --config=kind-config.yaml
```

### 2. NGINX Ingress Controllerのインストール
```bash
kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/main/deploy/static/provider/kind/deploy.yaml
```

### 3. Argo Workflowsのインストール
```bash
kubectl create namespace argo
kubectl apply -n argo -f https://github.com/argoproj/argo-workflows/releases/download/v3.5.5/install.yaml
```

### 4. Argo Serverの設定
```bash
kubectl patch deployment argo-server -n argo -p '{"spec":{"template":{"spec":{"containers":[{"name":"argo-server","args":["server","--auth-mode=server"]}]}}}}'
kubectl apply -f argo-server-patch.yaml
kubectl apply -f argo-server-ingress.yaml
```

### 5. Ingress Controllerの準備完了を待機
```bash
kubectl wait --namespace ingress-nginx \
  --for=condition=ready pod \
  --selector=app.kubernetes.io/component=controller \
  --timeout=90s
```

## 環境
- kindクラスタ: `argo-workflows`
- Argo Workflows v3.5.5

## GUIアクセス

### 方法1: Ingress経由（推奨）
1. hostsファイルに追加:
```bash
echo "127.0.0.1 argo.local" | sudo tee -a /etc/hosts
```

1. ブラウザで http://argo.local:8080 にアクセス

### 方法2: ポートフォワード
```bash
kubectl port-forward -n argo svc/argo-server 2746:2746
```
ブラウザで http://localhost:2746 にアクセス

## サンプルワークフロー

### 1. Hello World
```bash
kubectl create -f hello-world-workflow.yaml
```

### 2. マルチステップワークフロー
```bash
kubectl create -f multi-step-workflow.yaml
```

### 3. DAGワークフロー
```bash
kubectl create -f dag-workflow.yaml
```

## ワークフロー確認
```bash
kubectl get workflows -n argo
kubectl logs -n argo <workflow-pod-name>
```

## クリーンアップ
```bash
kind delete cluster --name argo-workflows
```
# argo-workflows-test
