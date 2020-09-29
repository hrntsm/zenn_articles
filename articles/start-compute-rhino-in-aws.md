---
title: "AWSでできる！ウェブでのジオメトリ計算サービスCompute.Rhino3dの始め方"
emoji: "🦏"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["rhinoceros", "aws"]
published: false
---

# はじめに

AWSでのCompute.Rhino3dの始める方法についての記事です。

Compute.Rhino3dは開発中なため、2020/9/30時点での情報です。公式のドキュメントの更新も頻繁なため、参照しているリンクが切れている可能性があります。もしリンクが切れていたら公式のGitHubのリポのトップページは以下になりますので、そこから最新の情報を探してみてください。

[Rhino Compute Server](https://github.com/mcneel/compute.rhino3d)

## そもそもCompute.Rhino3dって？

以下公式より引用です。[Rhino Compute Service (ワークインプログレス)](https://www.rhino3d.com/compute)

> McNeelクラウドを介してステートレスREST APIを通じてRhinoのジオメトリライブラリへアクセスできるようにする実験的なプロジェクトです。Computeは、 Rhino Inside™ のテクノロジをベースに、Rhinoの高度なジオメトリ計算をオンラインのウェブサービスに埋め込みます。

Rhinocerosという3DCADのジオメトリ計算機能をオンラインのウェブサービスとして埋め込むことができます。公式が参考としてHerokuを使って以下のようなとげとげのものの形状を変更させながら試せるものを公開しています。

https://compute-rhino3d-appserver.herokuapp.com/example/

# AWSの支度

## 作成するインスタンスの種類

環境としては最低でもWindows Server2016、推奨ではWindows Server 2019 となっています。ここではAWSのWindows Server 2019 Baseを使うことを想定します。無料枠で使用可能な t2.micro (1vCPU, 1GB RAM) のインスタンスでも可能ですが、メモリが1GBでつらいです。公式のドキュメントでは、t2.medium (2vCPU, 4GB RAM)  がスタートするには良いと書かれています。
わたしも最初はt2.microでやっていましたが、動作が重すぎてt2.mediumに変えました。

## インスタンスの作成

AWS EC2でのWindows Server 2019のインスタンスの作り方はいろいろなサイトで紹介されているのでここでは割愛します。以下とかを参考にしました。”2.EC2インスタンに接続する” までやればよいです。

[AWSでEC2(WindowsServer2019)を作成し一般公開するまで方法](https://qiita.com/og_omochi/items/c85bfd61fd4bd9e5baab)

# Compute.Rhino3dの支度

## ライセンスの価格

公式のドキュメント [Pricing - Rhino 7 on Servers](https://www.rhino3d.com/compute-pricing)

通常のシングルコンピューターライセンスでは認証ではじかれてしまいます。サーバーインスタンスでのRhinoの実行には専用のライセンスが必要で、このライセンスはコア時間での課金になっています。現在WIP版なのでこの価格が正式版になるかわかりませんが、2020年9月30日現在では0.10米ドル/core-hourです。

Core-Hourなので課金形態は以下のようになります。

+ 1台の32コアのサーバーで1時間実行した場合
  + 1台 × 32コア × 1時間 × 0.10/core-hour = 3.20 米ドル
+ 200台の4コアのサーバーで6分実行した場合
  + 200台 × 4コア × 0.1時間 × 0.10/core-hour = 8.00 米ドル

## ライセンスの作成方法

上記で書いたようにWindows Server向けには専用のライセンスが必要になります。作り方としては自分のRhinoのアカウント内に専用のチームを作成します。

1. [Rhino Accounts](https://accounts.rhino3d.com/) にアクセスする
2. ライセンスのページに行く
3. 新規チームを作成する
4. チームの管理 から コア時間課金の管理… を選ぶ
5. コア時間課金を有効にして保存する
6. 再度コア課金の管理のページに行き、操作▼ から Get Auth Token を選ぶ
   + このAuthTokenをRhinoをインストールする際にライセンス認証するために使います。


## 実行環境構築

公式のドキュメント [Deploying Rhino Compute](https://github.com/mcneel/compute.rhino3d/blob/master/docs/deploy.md)

対象の環境で以下を行います。AWSでやるならばそのWindows Server内で行ってください。

1. PowerShell を起動して以下を入力する
    ```bash
    iwr -useb https://raw.githubusercontent.com/mcneel/compute.rhino3d/master/script/bootstrap-server.ps1 -outfile bootstrap.ps1; .\bootstrap.ps1 -install
    ```
2. 1.を実行すると必要なものがダウンロードされる途中で以下の入力を求められるためそれぞれを入力する
   + EmailAdress : RhinoWIPをダウンロードするために使用する
   + ApiKey : APIのキーでAPIアクセスする際に使うため
   + RhinoToken : ”ライセンスの作成方法”の部分で取得したAuthToken
3. ブラウザーで  http://public-dns-or-ip/version にアクセスして例えば以下のようにバージョンが表示されれば問題なく実行されています

    ```json
    {
      "rhino":"7.0.20259.15365",
      "compute":"1.0.0.493",
      "git_sha":"a612c257"
    }
    ```

# Compute.Rhino3dを実行する

## サンプルファイルを取得

mcneelのGitHubからサンプルファイルをクローンして使います。

[compute.rhino3d-samples](https://github.com/mcneel/compute.rhino3d-samples)

```bash
git clone https://github.com/mcneel/compute.rhino3d-samples.git
```

## APIキーとWebAdressの設定

Visual Studio などでSampleフォルダ内の RhinoComputeSamples.sln を開いて設定したAPIキーとcompute.rhinoのアドレス、compute.rhino3d-samples/samples/RhinoComputeフォルダ内にある RhinoCompute.csの以下の位置に入れてください。

Headerの追記箇所については近いうち追記しなくてもよいようにリポのデータを更新するとのことです。

```cs
namespace Rhino.Compute
{
    public static class ComputeServer
    {
        public static string WebAddress { get; set; } = " http://public-dns-or-ip/";
        public static string AuthToken { get; set; }

        public static T Post<T>(string function, params object[] postData)
        {
          .......
          request.Headers.Add("Authorization", "Bearer " + AuthToken);
          // ↓追記(25行目あたり)
          request.Headers.Add("RhinoComputeKey", "実行環境構築で設定したApiKeyを入れる");
          // ↑追記
          request.Method = "POST";
          .......
        }
    }
}
```

## AuthTokenの設定

”実行環境構築”の個所で入力したRhinoTokenをcompute.rhino3d-samples/samples/RhinoComputeフォルダ内にあるAuthToken.csの以下の位置に入れてください。

```cs
namespace Rhino.Compute {
    public static class AuthToken {
        public static string Get () {
          ....
          if (!String.IsNullOrEmpty (tokenFromEnv))
          {
            return tokenFromEnv;
          } else
          {
            return "ここにRhinoTokenいれる";
          }
          ....
```

## 実行！

後はSampleフォルダ内の各サンプルを実行し、うまく動作しているならば bin/Debugフォルダ内に各サンプルに応じたファイルが作成されます。
例えば BrepBooleanOperation では、cube_sphere_difference.obj、cube_sphere_intersection.obj、cube_sphere_union.obj の三つが作成されます。

cube_sphere_difference.obj ではbrepのメッシュ化とブーリアン演算をおこなった結果として以下のようなになっています。この機能のどちらも高級な関数を使うためCompute.Rhino3dでないとできない処理です。

![](https://storage.googleapis.com/zenn-user-upload/q4908ig96mxxu4es1yy3k95lm2ee)

## ちなみに

結局ローカルのRhinoAPI と RhinoCompute何が違うの？というのはここを見るとわかるかもしれません。

[Please Explain Local Rhino API vs Rhino Compute](https://discourse.mcneel.com/t/please-explain-local-rhino-api-vs-rhino-compute/108991?u=hiron)

# Next Step

今回はC#環境での実行でしたが、PythonやJavaScriptのCompute.Rhino3dもあるので自分のやりたいことに合った言語を使って、Compute.Rhino3dを満喫しましょう。

jsは 冒頭で紹介した参考例を作成する以下のチュートリアルがおすすめです。

[compute.rhino3d.appserver](https://github.com/mcneel/compute.rhino3d.appserver)

[2020 Digital Evolution Pre-Lab Workshop: Hosted by Steve Baer of McNeel](https://vimeo.com/442079095)

# まとめ

この記事では、AWSでのCompute.Rhino3dの使い方について解説しました。AWS上でRhinoの高級な関数が実行できることはとても魅力的ではないでしょうか。
計算自体はサーバーで行うため、スマフォのような端末でもRhinoの幾何計算の結果を取得できるのが面白いところだと思っています。

あなたもAWS、そしてCompute.Rhino3dへの重課金でのパケ死に気を付けてクラウドなRhinocerosを楽しみましょう！！！
