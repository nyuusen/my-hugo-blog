+++
date = "2025-03-27T22:12:34+09:00"
draft = false
title = "コネクション"
+++


## コネクションやコネクションプールについて

- アプリケーション側で接続先のDB情報とともにコネクションIDを保持し、データベース側ではコネクションIDとそれに対応したプロセスIDを保持する
  - 上記をメモリ上に保持しておくことで、接続情報を使い回すことができる（コネクションプール）
  - 再利用しない場合は、アプリケーション側でコネクションクローズ処理を忘れずに
- 基本的にはDBサーバー側には最大接続数があるので、アプリケーション側ではそれを超えないようにコネクションプールを実装する必要がある
  - 実装する必要があると言っても、基本的にはライブラリがよしなにやってくれるので、DBインスタンス初期化時に最大接続数の設定をするくらいで良いと思われる
  - [Amazon Aurora MySQL のパフォーマンスとスケーリングの管理 - Amazon Aurora](https://docs.aws.amazon.com/ja_jp/AmazonRDS/latest/AuroraUserGuide/AuroraMySQL.Managing.Performance.html)

## アプリケーション側でのコネクション管理の詳細

- 最大接続数
  - 少なすぎると処理待ちが多くなるし、多すぎるとDB側に負荷がかかる
  - DB側の最大接続数を超えないようにする
    - 例えばRDSだと、インスタンスタイプによって異なるので注意
- 最大アイドル時間
  - アクティブでないコネクションをどれくらいの時間維持するか
  - 長すぎると不要なコネクションが増えてリソースが無駄になるし、短すぎるとコネクションの再作成が頻発してオーバーヘッドが増える
- 最大生存期間
  - 1つのコネクションが使われ続ける最大時間
  - DBの設定によっては、一定の時間でコネクションが強制的に切られることがあるので、アプリケーション側で事前に適切なタイミングで切ることで予期せぬエラーを防ぐ

## RDS Proxyというアプローチ

- コネクションの管理を良い感じにマネージド管理してくれるのがRDS Proxyである
- 詳細は[ここ](https://github.com/nyuusen/TIL/blob/de7727820ca1fc9f8e3f39003df022446116edfc/reading/003_AWS%E8%A8%AD%E8%A8%88%E3%82%B9%E3%82%AD%E3%83%AB%E3%82%A2%E3%83%83%E3%83%97%E3%82%AC%E3%82%A4%E3%83%89.md#rds-proxy)に色々まとめている
- RDS Proxy自身は、VPC内のENIを使って、RDSに接続する
  - Proxyのスケールに応じて、ENIが多く使用される＝IP枯渇問題が発生する可能性がある点に注意
    - サブネットのCIDR範囲はあらかじめ広めにしておくとかの対策