---
title: "Github Actions を使った Grasshopper コンポーネントのビルドの仕方"
emoji: "🦏"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["grasshopper", "github"]
published: false
---

# はじめに

この記事は、[AEC and Related Tech Advent Calendar 2020](https://adventar.org/calendars/5473) の 1 日目の記事です。建築関連のことならば技術記事に限らずなんでも OK のアドカレになっていますので興味のある方はみてください!

ここでは、建築系で設計検討に最近使われている Grasshopper というソフトで動作するコンポーネント（プラグイン）を、Github Actions を使ってビルドする方法についてを紹介します。
要は、.NET Framework を Github Actions で使ってビルドする方法になります。

## GitHub Actions とは

以下公式より引用 [GitHub Actions](https://github.co.jp/features/actions)

> GitHub Actions を使用すると、ワールドクラスの CI / CD ですべてのソフトウェアワークフローを簡単に自動化できます。 GitHub から直接コードをビルド、テスト、デプロイでき、コードレビュー、ブランチ管理、問題のトリアージを希望どおりに機能させます。

GitHub のリポにプッシュやプルリクなどの設定したアクションをしたときに、仮想マシンやコンテナを使ってテストやビルドなどを行える機能です。

# やりたいこと

以下のときに、GitHub Actions を使ってコンポーネントをビルドして GitHub 上に保存する。

- develop にプッシュ、プルリク
- main にプルリク

GitHub Actions は Windows 環境にも対応しているため、Windows 環境で Visual Studio を起動してビルドさせることを行います。

# ローカルでの支度

Grasshopper コンポーネントの開発には Visual Studio 2019 を使います。0 から開発するのは大変なので、以下から開発用のテンプレートをダウンロードして使用します。

[Grasshopper templates for v6](https://marketplace.visualstudio.com/items?itemName=McNeel.GrasshopperAssemblyforv6)

こちらのテンプレートを使用すると、RhinoCommon.dll や GH_IO.dll などの参照がローカルになっています。GitHub の環境では当然ですが Rhino がインストールされていないため、これらの dll ファイルはローカルにないので、nuget を使ったものに修正してください。
余談ですが、nuget の Rhino 関連のものの最新版は Rhino7 向けになっています。Rhino6 向けに使う場合は 6.XX で書かれているバージョンを使いましょう。

nuget パッケージの管理形式は、Package.config ではなく、PackageReference にしてください。VisualStudio のオプションの以下から変更できます。

![](https://github.com/hrntsm/zenn_articles/blob/master/image/PackageReference.png?raw=true)

# GitHub Actions の設定の仕方

GitHub Actions は、YAML 構文を使用してイベント、ジョブ、およびステップを定義しています。

この YAML ファイルを、コードリポジトリの .github/workflows というディレクトリに保存することで、動作の対象になります。
ファイル名は何でも構いません。

ファイルの内容は以下になります。適宜コメントアウトで説明しています。
やっていることを要約すると、MSBuild を使って対象のプロジェクトファイルをビルドしています。

```yml
# このワークフローの名前（バッジを作るときなどに使う）
name: Build Grasshopper Plugin

on:
  push: # develop にプッシュしたときに動く
    branches: [develop]
  pull_request: # main と develop にプルリクしたときに動く
    branches: [main, develop]

jobs:
  build:
    # Github Actions での Windows の最新の環境を指定
    #（現在は Windows Server 2019 になる ）
    runs-on: windows-latest # windows-2019 でも同じ意味

    steps:
      # git のチェックアウトを行い、環境に対象のリポのデータを取得する
      - name: Checkout
        uses: actions/checkout@v2

      # Vusial Studio (MSBuild)のセットアップをする
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
      # 対象はリリースビルド
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

# バッジを付ける

Github Actions の結果をバッジとして取得できます。これを README に張り付けることで整備されたリポジトリのような気持ちになれます。

バッジの作成には [shields.io](https://shields.io/category/build) というサービスを使うと便利です。以下のようにリポの情報を入れると自動で情報を取得してバッジを作成してくれます。

ここでは build が通っているかどうかのバッジが作成されますので、それを README などに張り付けるとバッジをリポジトリに表示できます。

![](https://github.com/hrntsm/zenn_articles/blob/master/image/Shields.io.png?raw=true)

# 参考リポジトリ

この内容は以下のリポで環境構築しています。参考にしてください。
ちなみにいろんな CI が動いてどんなことが出力されるかわかるように、ここではビルドは失敗させてます。

[GrasshopperCISample](https://github.com/hrntsm/GrasshopperCISample)
