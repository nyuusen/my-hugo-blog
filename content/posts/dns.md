+++
date = "2024-10-24T22:57:11+09:00"
draft = false
title = "DNS"
+++


## 名前解決の流れ

- クライアントがドメイン(example.com)へのアクセスを試行する
- クライアントが設定されたDNSリゾルバにドメイン名を問い合わせを行う
- DNSリゾルバがルートネームサーバ(.com)にネームサーバー(NS)情報を取得する
- DNSリゾルバがセカンドレベルドメイン(example.com)からIPアドレスを取得する
- DNSリゾルバがクライアントにIPアドレスを返す
- クライアントが名前解決されたドメインにアクセスする
- 参考: [新入社員「DNSってなんですか？」→ これ、どこまで答えられますか？？？？？？？？？](https://zenn.dev/msy/articles/e1e5aed46a3e49)

## ドメイン移管の流れ

業務でAWSアカウントを跨いだドメイン移管を行ったので、流れをまとめておく。  
AWSのRoute53を用いてドメイン管理を行なっているものとする。

- 移管先に対象ドメインのホストゾーンを作成(NSレコードが生成される)
- 親ドメインを管理しているホストゾーンの対象ドメインのNSレコードを更新する

### 注意ポイント

- 対象ドメインの向き先(Aレコード)はCloudFrontディストリビューション(CFD)であったが、代替ドメイン名(CNAMEs)はユニークである必要があった
  - そのため、旧アカウントと新アカウントのそれぞれのCFDの代替ドメイン名に今回のドメインを同時に設定しておくことができなかった
  - 以下の公式で紹介されている通り、ワイルドカードを使用する方法を採用したかったが、親ドメインの管理権限が自分たちになかったため、仕方なしに、旧の代替ドメイン名を一旦空にしてから、新の代替ドメイン名を設定するようにした。
    - [代替ドメイン名を別のディストリビューションに移動する - Amazon CloudFront](https://docs.aws.amazon.com/ja_jp/AmazonCloudFront/latest/DeveloperGuide/alternate-domain-names-move.html)
    - この方法では、一時的に証明書エラー(HTTPS接続エラー)が発生することに注意は必要
  - ちなみにこの代替ドメイン名は、ACM証明書の認証に使用されているらしい

## ACM認証用CNAMEレコードを登録するゾーンについて

ACMで発行した証明書のDV認証にはCNAMEレコードの登録が必要になる。  
その際に、Route53のどのホストゾーン(親？子？)にCNAMEレコードを登録すべきかという件について、ちょっと調べた。

- 結論：移譲されている子のホストゾーンに登録する
  - ちなみに移譲元である親のホストゾーンに登録しても認証は通らない(digコマンドとかでCNAMEレコードの名前解決がそもそもできない)
- そもそもDV認証というのは「該当ドメインの所有者であるか？」の確認するための目的で、ACM証明書作成時に発行されるCNAMEレコードをそのドメインに登録させるという手段を取っている
- そしてこの認証が**対象ドメインのDNSゾーン**に対して行われるため、権限を移譲してしまっている親ドメインでは認証が通らないためと思われる
  - 対象ドメイン名のネームサーバーを問い合わせる→指定されたCNAMEレコードがそのドメインのDNSに存在するかを定期的に確認している
- 結局、AレコードやCNAMEレコード等のレコード種別に関係なく、ネームサーバーを辿っていき、対象ドメインのDNSゾーンの何らかのレコードを解決するという方法になる   

### そもそもDNSゾーンとは？

- あるドメインに関連するDNSレコードを管理するための領域(namespace)
