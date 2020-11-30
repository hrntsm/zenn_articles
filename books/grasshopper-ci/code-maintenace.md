---
title: "Code Maintainability を測る"
---

# はじめに

Code Climete というサービスを使って、コードのメンテナンス性を測ってみます。

# Code Climete とは

こちらもコードレビュータイプの CI サービスです。Codacy と同様にコードレビューするサービスはいくつかありますが、C# をレビューしてくれて Github との連携が簡単なことから今回はこのサービスを使用します。
このサービスはパブリックなリポジトリではフリーになっております。
Codacy 重複する部分もありますが、違いはメンテナンス性についてよく見てくれます。
例えば以下の点です。

- 1 つのクラスに〇個以上メソッドがある
- 1 つのメソッドが〇行以上ある

こちらももちろん直さなくてもビルドは通りますが、長いということはそのメソッドが複数の役割を持っていたりする可能性があります。より簡潔にしたほうがあとから見たときにわかりやすいので、そうしましょう。

公式サイト
[codeclimate](https://codeclimate.com/)

# 使い方

まず codeclimate のアカウントを作成します。Github と連携するので、Github のアカウントを使ってサインアップすると連携がスムーズです。
なお、Login は velocity と quality の 2 つあり、今回使うのは quality のほうなので間違えないようにしましょう。
アカウントを作成すると連携するリポジトリを選択する画面になるので、対象にしたいリポジトリを選んでください。

![](https://github.com/hrntsm/zenn_articles/blob/master/books/grasshopper-ci/image/LoginClimate.png?raw=true)

該当のファイルを選択するとどこでクオリティが下がっているか確認できるので、修正時は参考にしてください。

# プルリク時に動くようにする

Code Climate の以下の箇所から適宜設定しましょう。

![](https://github.com/hrntsm/zenn_articles/blob/master/books/grasshopper-ci/image/PullreqClimate.png?raw=true)
