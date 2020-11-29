---
title: "GitHub Actions でコンポーネントをビルドする"
---

# はじめに

この章では、Grasshopper というソフトで動作するコンポーネント（プラグイン）を、Github Actions を使ってビルドする方法についてを紹介します。

# GitHub Actions とは

以下公式より引用 [GitHub Actions](https://github.co.jp/features/actions)

> GitHub Actions を使用すると、ワールドクラスの CI / CD ですべてのソフトウェアワークフローを簡単に自動化できます。 GitHub から直接コードをビルド、テスト、デプロイでき、コードレビュー、ブランチ管理、問題のトリアージを希望どおりに機能させます。

GitHub のリポにプッシュやプルリクなどの設定したアクションをしたときに、仮想マシンやコンテナを使ってテストやビルドなどを行える機能です。

# やりたいこと

develop にプッシュしたとき、および main にプルリクした際 GitHub Actions を使ってコンポーネントをビルドして GitHub 上に保存することをやります。

GitHub Actions は Windows 環境にも対応しているため、Windows 環境で Visual Studio を起動してビルドさせることを行います。

# ローカルでの支度

Grasshopper コンポーネントの開発には Visual Studio 2019 を使います。0 から開発するのは大変なので、以下から開発用のテンプレートをダウンロードして使用します。

[Grasshopper templates for v6](https://marketplace.visualstudio.com/items?itemName=McNeel.GrasshopperAssemblyforv6)

こちらのテンプレートを使用すると、RhinoCommon.dll や GH_IO.dll などの参照がローカルになっています。GitHub の環境では当然ですがこれらの dll ファイルはローカルにないため、nuget を使ったものに修正してください。

nuget パッケージの管理形式は、Package.config ではなく、PackageReference にしてください。

![](https://github.com/hrntsm/zenn_articles/blob/master/image/PackageReference.png?raw=true)

# GitHub Actions の設定の仕方

GitHub Actions は、YAML 構文を使用してイベント、ジョブ、およびステップを定義しています。

この YAML ファイルは、コードリポジトリの .github/workflows というディレクトリに保存することで、動作の対象になります。

ファイルの内容は以下になります。適宜コメントで説明しています。
やっていることを要約すると、msbuild を使って対象のプロジェクトファイルをビルドしています。

```yml
# このワークフローの名前（バッジを作るときなどに使う）
name: Build Grasshopper Plugin

on:
  push: # develop にプッシュしたときに動く
    branches: [develop]
  pull_request: # main にプルリクしたときに動く
    branches: [main]

jobs:
  build:
    # Github Actions での Windows の最新の環境を指定
    #（現在は Windows Server 2019 になる ）
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
      - name: Upload build as artifact
        uses: actions/upload-artifact@v2
        with:
          name: GrasshopperComponent
          path: ./GrasshopperCISample/bin/GrasshopperCISample.gha
```

# 動作確認

上記ファイルをリモートの develop にプッシュすると、Actions が動き出します。動作は GitHub の対象のリポの Actions のタブをクリックすると確認できます。Actions が動いているときは以下のようにオレンジ色の丸が表示され、問題なく動作が完了すると緑のㇾマーク、何かエラーがあり止まると赤色の × マークになります。

![](https://github.com/hrntsm/zenn_articles/blob/master/image/CheckWorkFlow.png?raw=true)

問題なく動作が完了すると、 以下のように Artifact としてビルドしたものがアップされ、クリックすることでダウンロードできます。

![](https://github.com/hrntsm/zenn_articles/blob/master/image/Artifact.png?raw=true)

# これをやる利点

例えばリリースするデータを main ブランチで管理しているとします。main ブランチに直接プッシュしたとき、そのデータがちゃんとビルドできるデータであるかは個人の注意に依存しています。
それを避けるために、main ブランチにプルリクした際、ここで設定した CI が動くようにしています。
うっかり動かないものでも main ブランチにプルリクすると CI でチェックされるので以下のように check が failed になり、ミスを未然に防げます。

![](https://github.com/hrntsm/zenn_articles/blob/master/books/grasshopper-ci/image/pullreq.png?raw=true)
