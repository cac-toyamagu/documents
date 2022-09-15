# TIPS

## WSLのローカルホストにWSL上に立てたコンテナからアクセスする

- `ifconfig` からIPを調べて行う。
  - `IP=`ifconfig eth0 | grep inet | grep -v inet6 | awk '{print $2}'``
  - `echo $IP`

## ArgoCD Development tool chainをWSL2で実行する

## 成功

- 0.0.0.0ではなく、WSLのIPアドレスを指定し、TLS証明書にWSLのIPアドレス用のSAN登録を行う。
  - `IP=`ifconfig eth0 | grep inet | grep -v inet6 | awk '{print $2}'``
  - k3d cluster create argocd --k3s-arg "--tls-san=${IP}@server:0" --api-port $IP:6550

### 失敗

- `k3d create cluster`, `kind create cluster`
  - `make verify-kube-connect` -> Connection refused
- `~/.kube/config` で `0.0.0.0:<port>` から以下のIPに変更 -> TLS証明書SANエラー
  - `IP=`ifconfig eth0 | grep inet | grep -v inet6 | awk '{print $2}'``
