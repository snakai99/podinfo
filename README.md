# podinfo

[![e2e](https://github.com/stefanprodan/podinfo/workflows/e2e/badge.svg)](https://github.com/stefanprodan/podinfo/blob/master/.github/workflows/e2e.yml)
[![test](https://github.com/stefanprodan/podinfo/workflows/test/badge.svg)](https://github.com/stefanprodan/podinfo/blob/master/.github/workflows/test.yml)
[![cve-scan](https://github.com/stefanprodan/podinfo/workflows/cve-scan/badge.svg)](https://github.com/stefanprodan/podinfo/blob/master/.github/workflows/cve-scan.yml)
[![Go Report Card](https://goreportcard.com/badge/github.com/stefanprodan/podinfo)](https://goreportcard.com/report/github.com/stefanprodan/podinfo)
[![Docker Pulls](https://img.shields.io/docker/pulls/stefanprodan/podinfo)](https://hub.docker.com/r/stefanprodan/podinfo)

Podinfoは、Kubernetesでマイクロサービスを実行するためのベストプラクティスを紹介する、Goで作られた小さなWebアプリケーションです。
Podinfo is used by CNCF projects like [Flux](https://github.com/fluxcd/flux2) and [Flagger](https://github.com/fluxcd/flagger)
for end-to-end testing and workshops.

Specifications:

* 健康診断(準備と生活)

*割り込み信号の優雅なシャットダウン

* シークレットとコンフィグマップのファイルウォッチャー

*プロメテウスとオープンテレメトリで計装

* zapによる構造化ロギング

*バイパー付き12ファクターアプリ

* 障害注入(ランダムエラーとレイテンシ)

*スワッガードキュメント

* CUE、Helm、Kustomizeインストーラー

* Kubernetes KindとHelmによるエンドツーエンドのテスト

* GitHub ActionsとOpen Policy AgentによるKustomizeテスト

* Docker buildxとGithubアクションを備えたマルチアーチコンテナイメージ

* Sigstore cosignによるコンテナイメージ署名

* TrivyによるCVEスキャン

Web API:

* `GET /`はランタイム情報を出力します

* `GET /version`はpodinfoバージョンとgitコミットハッシュを出力します

* `GET /metrics`はHTTPリクエストの期間とGoランタイムメトリクスを返します

* Kubernetesライブネスプローブで使用される「GET /healthz」

* Kubernetes準備プローブで使用される「GET /readyz」

* `POST /readyz/enable`は、このインスタンスがトラフィックを受信する準備ができていることをKubernetes LBに通知します

* `POST /readyz/disable`は、このインスタンスへのリクエストの送信を停止するようにKubernetes LBに通知します

* `GET /status/{code}`はステータスコードを返します

* `GET /panic`は、終了コード255でプロセスをクラッシュさせます

* `POST /echo`はコールをバックエンドサービスに転送し、投稿されたコンテンツをエコーします

* `GET /env`は環境変数をJSON配列として返します

* `GET /headers`は、リクエストHTTPヘッダーを含むJSONを返します

* `GET /delay/{seconds}`は指定された期間を待ちます

* `POST /token`は1分間有効なJWTトークンを発行します `JWT=$(curl -sd 'anon' podinfo:9898/token | jq -r .token)`

* `GET /token/validate`はJWTトークンを検証します `curl -H "Authorization: Bearer $JWT" podinfo:9898/token/validate`

* `GET /configs` は、configmaps および/またはシークレットが `config` ボリュームにマウントされた JSON を返します

* `POST/PUT /cache/{key}`は、投稿されたコンテンツをRedisに保存します

* `GET /cache/{key}`は、キーが存在する場合、Redisからコンテンツを返します

* `DELETE /cache/{key}`は、存在する場合、Redisからキーを削除します

* `POST /store`は、投稿されたコンテンツを/data/hashのディスクに書き込み、コンテンツのSHA1ハッシュを返します

* `GET /store/{hash}`は、存在する場合、ファイル/data/hashの内容を返します

* `GET /ws/echo`はウェブソケットを介してコンテンツをエコーします `podcli ws ws://localhost:9898/ws/echo`

* `GET /chunked/{seconds}` は、`transfer-encoding` タイプ `chunked` を使用して部分的な応答を与え、指定された期間を待ちます

* `GET /swagger.json`は、LinkerdサービスのプロファイリングとGlooルート検出に使用されるAPI Swaggerドキュメントを返します。

gRPC API:

* `/grpc.health.v1。ヘルス/チェック ヘルスチェック

ウェブUI:

![Podinfo-ui](https://raw.githubusercontent.com/stefanprodan/podinfo/gh-pages/screens/podinfo-ui-v3.png)

Swagger UIにアクセスするには、ブラウザで`<podinfo-host>/swagger/index.html`を開きます。

### ガイド

* [GitOps Progressive Deliver with Flagger、Helm v3、Linkerd](https://helm.workshop.flagger.dev/intro/)

* [GitOps Progressive Deliver on EKS with Flagger and AppMesh](https://eks.handson.flagger.dev/prerequisites/)

* [FlaggerとIstioによる自動カナリア展開](https://medium.com/google-cloud/automated-canary-deployments-with-flagger-and-istio-ac747827f9d1)

* [Istioメトリクスを使用したKubernetes自動スケーリング](https://medium.com/google-cloud/kubernetes-autoscaling-with-istio-metrics-76442253a45a)

* [カスタムメトリクスを使用したFargateでのEKSの自動スケーリング](https://aws.amazon.com/blogs/containers/autoscaling-eks-on-fargate-with-custom-metrics/)

* [ヘルムの管理はGitOpsの方法をリリース](https://medium.com/google-cloud/managing-helm-releases-the-gitops-way-207a6ac6ff0e)

* [ContourでEKS入力を保護し、GitOpsの方法を暗号化しましょう](https://aws.amazon.com/blogs/containers/securing-eks-ingress-contour-lets-encrypt-gitops/)
### Install

#### Helm

Install from github.io:

```bash
helm repo add podinfo https://stefanprodan.github.io/podinfo

helm upgrade --install --wait frontend \
--namespace test \
--set replicaCount=2 \
--set backend=http://backend-podinfo:9898/echo \
podinfo/podinfo

helm test frontend

helm upgrade --install --wait backend \
--namespace test \
--set redis.enabled=true \
podinfo/podinfo
```

Install from ghcr.io:

```bash
helm upgrade --install --wait podinfo --namespace default \
oci://ghcr.io/stefanprodan/charts/podinfo
```

#### Kustomize

```bash
kubectl apply -k github.com/stefanprodan/podinfo//kustomize
```

#### Docker

```bash
docker run -dp 9898:9898 stefanprodan/podinfo
```

### Continuous Delivery

In order to install podinfo on a Kubernetes cluster and keep it up to date with the latest
release in an automated manner, you can use [Flux](https://fluxcd.io).

Install the Flux CLI on MacOS and Linux using Homebrew:

```sh
brew install fluxcd/tap/flux
```

Install the Flux controllers needed for Helm operations:

```sh
flux install \
--namespace=flux-system \
--network-policy=false \
--components=source-controller,helm-controller
```

Add podinfo's Helm repository to your cluster and
configure Flux to check for new chart releases every ten minutes:

```sh
flux create source helm podinfo \
--namespace=default \
--url=https://stefanprodan.github.io/podinfo \
--interval=10m
```

Create a `podinfo-values.yaml` file locally:

```sh
cat > podinfo-values.yaml <<EOL
replicaCount: 2
resources:
  limits:
    memory: 256Mi
  requests:
    cpu: 100m
    memory: 64Mi
EOL
```

Create a Helm release for deploying podinfo in the default namespace:

```sh
flux create helmrelease podinfo \
--namespace=default \
--source=HelmRepository/podinfo \
--release-name=podinfo \
--chart=podinfo \
--chart-version=">5.0.0" \
--values=podinfo-values.yaml
```

Based on the above definition, Flux will upgrade the release automatically
when a new version of podinfo is released. If the upgrade fails, Flux
can [rollback](https://toolkit.fluxcd.io/components/helm/helmreleases/#configuring-failure-remediation)
to the previous working version.

You can check what version is currently deployed with:

```sh
flux get helmreleases -n default
```

To delete podinfo's Helm repository and release from your cluster run:

```sh
flux -n default delete source helm podinfo
flux -n default delete helmrelease podinfo
```

If you wish to manage the lifecycle of your applications in a **GitOps** manner, check out
this [workflow example](https://github.com/fluxcd/flux2-kustomize-helm-example)
for multi-env deployments with Flux, Kustomize and Helm.
