# TIPS

## WSLのローカルホストにWSL上に立てたコンテナからアクセスする

- `ifconfig` からIPを調べて行う。
  - `IP=`ifconfig eth0 | grep inet | grep -v inet6 | awk '{print $2}'``
  - `echo $IP`

## ArgoCD Development tool chainをWSL2で実行する

## 成功

- TLS証明書のSANを変え、 `~/.kube/config` も変える。
  - `k3d cluster create --k3s-arg "--tls-san=${IP}@server:0"`

### 失敗

- `k3d create cluster`, `kind create cluster`
  - `make verify-kube-connect` -> Connection refused
- `~/.kube/config` で `0.0.0.0:<port>` から以下のIPに変更 -> TLS証明書SANエラー
  - `IP=`ifconfig eth0 | grep inet | grep -v inet6 | awk '{print $2}'``
