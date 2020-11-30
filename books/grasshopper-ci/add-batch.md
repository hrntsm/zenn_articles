---
title: "各 CI の結果をバッジにして README に張り付ける"
---

# はじめに

この章では、README に CI 結果のバッジのつけ方について説明します。

# ビルド結果のバッジを取得する

結果をバッジとして取得できます。これを使うことで整備されたリポジトリのような気持ちになれます。
バッジの作成には [shields.io](https://shields.io/category/build) というサービスを使うと便利です。以下のようにリポの情報を入れると自動で情報を取得してバッジを作成してくれます。

ここでは build が通っているかどうかのバッジが作成されますので、それを README などに張り付けるとバッジをリポジトリに表示できます。

![](https://github.com/hrntsm/zenn_articles/blob/master/image/Shields.io.png?raw=true)

# Codacy の結果のバッジを取得する

Codacy

対象の Codacy のページに行き Settings の General から Codacy Badge の箇所で取得できます。

![](https://github.com/hrntsm/zenn_articles/blob/master/books/grasshopper-ci/image/CodacyBatch.png?raw=true)

# Code Climate の結果のバッジを取得する

対象の Code Climate のページに行き Repo Settings の Badges から Maintainability Badge の箇所で取得できます。

![](https://github.com/hrntsm/zenn_articles/blob/master/books/grasshopper-ci/image/ClimateBadge.png?raw=true)

# 結果

各箇所で取得したバッジの画像のリンクを README に書き込みましょう。

いいね 👍

![](https://github.com/hrntsm/zenn_articles/blob/master/books/grasshopper-ci/image/README.png?raw=true)
