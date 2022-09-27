# Resources

- [Resources](#resources)
  - [Purpose](#purpose)
  - [General](#general)
    - [Blog, News](#blog-news)
  - [Kubernetes](#kubernetes)
    - [Documents](#documents)
    - [Books](#books)
    - [News, Blog](#news-blog)
  - [ServiceMesh](#servicemesh)
    - [Documents](#documents-1)
    - [Book](#book)
  - [CI/CD](#cicd)
    - [Documents](#documents-2)
      - [Git Flow](#git-flow)
      - [Tool](#tool)

## Purpose

- 学習・キャッチアップする際に参考になるリソースをまとめること。

## General

### Blog, News

- [The New Stack](https://thenewstack.io/)
- [InfoQ](https://www.infoq.com/)
- [DZone](https://dzone.com/)
- [MartinFowler](https://martinfowler.com/)
- [SRE Weekly](https://sreweekly.com/)
- [Medium](https://medium.com/)

## Kubernetes

### Documents

- [Kubernetes](https://kubernetes.io/)
  - K8s公式
- [learnk8s](https://learnk8s.io/)
  - 有名なK8s学習サイト
- [Introduction to Kubernetes](https://www.edx.org/course/introduction-to-kubernetes)

- [EKS Best Practices Guides](https://aws.github.io/aws-eks-best-practices/)
  - EKSのベストプラクティスリスト

### Books

- [Kubernetes in Action](https://www.manning.com/books/kubernetes-in-action-second-edition)
  - 書籍(英語)。Kubernetesを使う上で必要な知識を動かしながら学べる。
- [Kuberentes完全ガイド](https://www.amazon.co.jp/Kubernetes%E5%AE%8C%E5%85%A8%E3%82%AC%E3%82%A4%E3%83%89-%E7%AC%AC2%E7%89%88-Top-Gear-%E9%9D%92%E5%B1%B1/dp/4295009792)
  - 書籍（日本語）。広い範囲がカバーされている。
- [Kubernetesで実践するクラウドネイティブDevOps](https://www.oreilly.com/library/view/kubernetesdevops/9784873119014/)
  - 書籍（日本語）。DevOps・K8sの基礎からメトリクス取得などの運用まで述べられている。
  - 一冊目にも良い。

### News, Blog

以下はSlack等でfeedし、日々確認しておくと良い。

- [Kubenews](https://kubenews.net/)
  - K8s関連のニュースサイトがまとまっている。
- [Kubernetes official blog](https://kubernetes.io/blog/)
  - K8s公式ブログ
- [Kubeweekly](https://www.cncf.io/kubeweekly/)
  - CNCFのK8sニュースサイト
- [EKS News](https://eks.news/)
  - EKS関連のニュースサイト
- [AWS News](https://aws.amazon.com/about-aws/whats-new/recent/)
  - AWS News
- [AWS Blog](https://aws.amazon.com/en/blogs/)
  - AWS Blog。Containersはフォローしておくと良い。

## ServiceMesh

### Documents

- [Istio](https://istio.io/)
- [Linkerd](https://linkerd.io/)
- [Consul](https://www.consul.io/)

### Book

- [Introducing istio service mesh microservices](https://developers.redhat.com/e-books/introducing-istio-service-mesh-microservices-old)
  - 書籍（英語）。有名なServiceMeshであるIstioの入門書。

## CI/CD

### Documents

#### Git Flow

- [Git Flow](https://nvie.com/posts/a-successful-git-branching-model/)
  - 有名なブログ。注意事項に書いてあるとおり、複雑なのでWebアプリ向けでないこともある。
- [GitHub Flow](https://docs.github.com/en/get-started/quickstart/github-flow)
  - 上記を軽量化したもの。
- [GitLab Flow](https://docs.gitlab.com/ee/topics/gitlab_flow.html)
  - GitHub Flowを修正し、多少複雑になる分いくつかの問題を解決したもの。
  - 環境用ブランチを用意するなど、CI/CD環境があるとこのFlowが良いのではないかと思う。

#### Tool

- [CircleCI](https://circleci.com/)
- [GitHub Actions](https://docs.github.com/ja/actions)
- [GitLab CI](https://docs.gitlab.com/ee/ci/)
- [ArgoCD](https://argo-cd.readthedocs.io/en/stable/)
- [Spinnaker](https://spinnaker.io/)
