# KakaoPayのCSR改善

## 既存の問題

1. 物理サーバー単位のscale調整が難しい既存サービス環境
2. frontend サービスのdeploy/rollbackに時間が掛かる
3. productionオープン前、最終確認する方法がなかった
4. リンクから見れる、ページpreviewのog設定業務が存在

(補足：KakaoPayは韓国のPayPayです)

## 1. 物理サーバー単位のscale調整が難しい既存サービス環境

サービス初期、既存でCSRファイル(index.html)をserveしていたサーバーが存在した、時間が過ぎて認証やSEOも担当する大きな共通サーバーになってしまった。

- 既存のCSR環境で共通機能使用のため、FEサービスが共通サーバーに依存
- trafficの増加

対策と思ったのが

1. Static Web Hosting (csr結果ファイルを簡単にserveできるように)
　　参照しているすべてのサービスの修正が必要
2. SSRへの移行
    影響が大きい
3. ServerLessを導入
　　影響が大きい

### Legacyサーバーの管理が難しいかった理由

1. 物理サーバーだったため、scaleする場合、新規サーバーを生成してmigrationまで必要
2. 個別のサービスの結果ファイルが共通サーバーに保存されたため、管理が難しい。

対応

1. Containerベースの設計に変更
2. サービスの結果ファイルをサーバー外Storageに分離

## 2. deployとrollbackに時間が掛かる

### デプロイ時

1. Git repoからmaster branchのコードを
2. npm ci
3. npm run build
4. Release

### rollback流れ

1. Git repoからcommit id
2. npm ci 
3. npm run build
4. Release

問題

npm ci に時間がかかっていた。５分以上

対応

yarn berry2を導入してdependencyをcache、

- Packageをnode_modulesではなく.pnp.cjsファイルで管理
- Packageファイルはプロジェクト内のCacheフォルダに保存されgitで管理
  - .pnp.cjsはCacheフォルダのPackageを参照

rollbackの場合はCSRだったため、結果ファイルを残しておいてsymlinkで決めるように

適用後

dependency設置をskipしてデプロイにかかる時間を減らせた。

## 3.productionオープン前、最終確認する方法がなかった

- beta（本番に近い）環境とproduction環境は同じではない。
- 他のサービスと連携している部分の確認ができないケースがあった。

解決

ユーザー情報をベースで、指定されているユーザーの場合先に利用できるようにする。

- 指定されているユーザーは、指定のサーバーにアクセスするように？

## 4.CSR環境で、リンクの設定やthumbnailイメージの設定でコードの修正が必要

ラインなどでリンクを共有した際、og:titleやog:imageがチャットで表示される。

変更したい場合、エンジニアやhtmlの修正が必要となる。マーケチームとかで積極的に使いにくい。

解決

og情報を担当する経由サーバーを作って、管理できるように

1. ユーザーからリンクをコピーした時、経由サーバーのリンクをコピー
2. 経由サーバーはエンジニアではなくてもアクセスが可能、編集可能
3. ↑の機能はog:title, og:image情報と実際ページにリダイレクト
4. ユーザーがラインにリンクを貼ると、経由サーバーの情報で表示されるけど実際アクセスしたら、正しいページにリダイレクト

結果

エンジニアの工数が減った

## 参照記事

- [KakaoPay CSR環境で改善があった内容](https://www.youtube.com/watch?v=CUSy500EuDs)

















