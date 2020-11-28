---
title: "Github を使った Grasshopper コンポーネントのビルドの仕方"
emoji: "🦏"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["rhinoceros", "grasshopper"]
published: false
---

# はじめに

この記事は、[AEC and Related Tech Advent Calendar 2020](https://adventar.org/calendars/5473) の 1 日目の記事です。

Rhinceros の Grasshopper で動作するコンポーネントを、Github Actions を使って build する方法についてを紹介します。

# GitHub Actions とは

以下公式より引用 [GitHub Actions](https://github.co.jp/features/actions)

> GitHub Actions を使用すると、ワールドクラスの CI / CD ですべてのソフトウェアワークフローを簡単に自動化できます。 GitHub から直接コードをビルド、テスト、デプロイでき、コードレビュー、ブランチ管理、問題のトリアージを希望どおりに機能させます。

GitHub のリポにプッシュやプルリクなどの設定したアクションをしたときに、仮想マシンやコンテナを使ってテストやビルドなどを行える機能です。

# やりたいこと

develop にプッシュしたとき、および main にプルリクした際 GitHub Actions を使ってコンポーネントをビルドして GitHub 上に保存することをやります。

GitHub Actions は Windows 環境にも対応しているため、そこで VisualStudio を起動してビルドさせることを行います。

# ローカルでの支度

Grasshopper コンポーネントの開発には VisualStudio2019 を使います。0 から開発するのは大変なので、以下から開発用にテンプレートを使用します。

[Grasshopper templates for v6](https://marketplace.visualstudio.com/items?itemName=McNeel.GrasshopperAssemblyforv6)

こちらのテンプレートを使用すると、RhinoCommon.dll や GH_IO.dll などの参照がローカルになっています。GitHub の環境では当然ですがこれらの dll ファイルはローカルにないため、Nuget を使ったものに修正します。

nuget パッケージの管理形式は、Package.config ではなく、PackageReference にしてください。

# GitHub Actions の設定の仕方

GitHub Actions は、YAML 構文を使用してイベント、ジョブ、およびステップを定義しています。これらの YAML ファイルは、コードリポジトリの .github/workflows というディレクトリに保存することで、動作の対象になります。

```yml
name: Build Grasshopper Plugin

on:
  push: # develop にプッシュしたときに動く
    branches: [develop]
  pull_request: # main にプルリクしたときに動く
    branches: [main]

jobs:
  build:
    # Windows の最新の環境を使用する（現在は Windows Server 2019 になる ）
    runs-on: windows-latest # windows-2019 でも同じ意味

    steps:
      # git のチェックアウトを行い、この環境に対象のリポを取得する
      - name: Checkout
        uses: actions/checkout@v2

      # Vusial Studio のセットアップをする
      - name: Setup MSBuild.exe
        uses: microsoft/setup-msbuild@v1

      # nuget のセットアップをする
      - name: Setup NuGet
        uses: NuGet/setup-nuget@v1

      # solution ファイルの状態を復元する
      # 例えば、nuget で参照しているファイルを取得するなど
      - name: Restore the application
        run: msbuild /t:Restore /p:Configuration=Release

      # Build 実行
      - name: Build the application
        run: msbuild /p:Configuration=Release

      # 対象パスにあるファイルを GitHub にアップロードする
      - name: Upload release build of plugin as artefact
        uses: actions/upload-artifact@v2
        with:
          name: GrasshopperComponent
          path: |
            ./GrasshopperCISample/bin/GrasshopperCISample.gha
```

# 参考リポ

この内容は以下のリポで環境構築しています。参考にしてください。

[GrasshopperCISample](https://github.com/hrntsm/GrasshopperCISample)
