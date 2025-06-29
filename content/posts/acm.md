+++
date = "2024-11-27T22:55:34+09:00"
draft = false
title = "AWS Certificate Manager"
+++


## ACMとリージョンの関係

ACM証明書はリージョナルリソースであるため、作成する際に適用するリソースに合わせ、リージョンを意識する必要がある。

- CloudFront
  - グローバルサービスなので、**米国東部 (バージニア北部, us-east-1) リージョン**に作成する
- ALB
  - リージョンサービスなので、**ALBが配置されているリージョン**に作成する

## 通信が暗号化する流れ(おさらい)

- サーバー側に証明書と秘密鍵をインストールする(証明書発行前の手続き等は省略)
  - これを担うのがACM証明書
- クライアントからのHTTPS通信に対し、証明書を返却する
  - クライアント側でCAやドメインの検証が行われる
- クライアントからの通信が公開鍵で暗号化される
- サーバー側で保持している秘密鍵で通信を復号する

この辺り(というか証明書発行・認証周りを)を抽象化、マネージドに提供してくれているのがACMとなる     

## ACMの認証について

- ACMに限らない話だが、証明書を発行する際に、証明書を発行した者が問題ないか？ドメインに対する権限を持っているか？等を検証する必要がある
  - この確認がないと、好き勝手に証明書作り放題になる
  - ここを実施しているのがCA(認証局) 
- その検証の部分を「認証」と呼ぶ
- 認証には、DV・OV・EV認証という種類がある
  - 参考: [DV、OV、EV の各 SSL 証明書の違いとは | DigiCert](https://www.digicert.com/jp/difference-between-dv-ov-and-ev-ssl-certificates) 
  - 右に行くほど厳格な認証方法となる
  - ACMはDV認証(ドメイン認証＝該当ドメインに対する権限を保持しているかどうかのみを確認する)
- ACMはDV認証をどうやっているのか？という話になるが、以下の手順で確認する
  - ACM証明書を発行すると、CNAME名とCNAME値が発行される(この段階ではACM証明書は未承認状態)
  - 該当ドメインのホストゾーンにCNAME名とCNAME値をCNAMEレコードとして登録する
  - ACM証明書が認証状態になる 🎉



