+++
date = "2025-01-20T21:16:52+09:00"
draft = false
title = "Lambdaレイヤー"
+++


## Lambdaレイヤーとは

- 複数のLambda関数でライブラリを共有できる仕組み
- ライブラリをLayerとしてアップロードしておくことで、個々の関数はLayerを使えば良くなる

## メリット

- デプロイメントパッケージのサイズを縮小できる
  - 依存関係の一部または全てをレイヤーに配置できるので、デプロイメントパッケージのサイズが小さくなる
- 関数のコアなロジックから依存関係を分離できる
- 複数の関数間で依存関係を共有できる（保守性の向上？）

## GoやRustでは推奨されていない

> Go または Rust で Lambda 関数を使用している場合は、レイヤーの使用はお勧めしません。
> o および Rust 関数の場合、関数コードを実行可能ファイルとして提供します。これには、コンパイルされた関数コードとそのすべての依存関係が含まれます。
> 依存関係をレイヤーに配置すると、関数は初期化フェーズ中に追加のアセンブリを手動でロードする必要があり、コールド スタート時間が長くなる可能性があります。Go および Rust 関数のパフォーマンスを最適化するには、依存関係をデプロイメント パッケージと一緒に含めます。
 
## 参考

- [レイヤーによる Lambda 依存関係の管理 - AWS Lambda](https://docs.aws.amazon.com/lambda/latest/dg/chapter-layers.html)