---
title: "AWSでのCompute.Rhino3dの始め方"
emoji: "🦏"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["rhinoceros", "aws"]
published: false
---

# はじめに

AWSでのCompute.Rhino3dの始め方についての記事です。mcneel公式のドキュメントは以下

[Getting started with Rhino Compute on AWS](https://github.com/mcneel/compute.rhino3d/blob/master/docs/getting-started.md)

# AWSの支度

## 作成するインスタンスの種類

環境としては最低でもWindows Server2016、推奨ではWindows Server 2019 となっています。ここではAWSのWindows Server 2019 Baseを使うことを想定します。無料枠で使用可能な t2.micro (1vCPU, 1GB RAM) のインスタンスでも可能ですが、メモリが1GBでつらいです。公式のドキュメントでは、t2.medium (2vCPU, 4GB RAM)  がスタートするには良いと書かれています。

## インスタンスの作成

AWS EC2でのWindows Server 2019のインスタンスの作り方はいろいろなサイトで紹介されているのでここでは割愛します。以下とかを参考にしました。

[AWSでEC2(WindowsServer2019)を作成し一般公開するまで方法](https://qiita.com/og_omochi/items/c85bfd61fd4bd9e5baab)

上記記事にもありますが、外部からアクセスできるようにするために以下をしてください。

1. 管理者でPowerShellを起動
2. Install-WindowsFeature -name Web-Server -IncludeManagementTools を実行して、IIS をインストールし、自動で 80 と 443 が開かれます。
3. URLの設定をします
   1. HTTPの場合:
      > netsh http add urlacl url="http://+:80/" user="Everyone"
   2. HTTPSの場合:
      > netsh http add urlacl url="https://+:443/" user="Everyone"

# Compute.Rhino3dの支度

## ライセンスの価格

公式のドキュメント [Pricing - Rhino 7 on Servers](https://www.rhino3d.com/compute-pricing)

通常のシングルコンピューターライセンスでは認証ではじかれてしまいます。サーバーインスタンスでのRhinoの実行には専用のライセンスが必要で、このライセンスはコア時間での課金になっています。現在WIP版なのでこの価格が正式版になるかわかりませんが、2020年■■現在では0.10米ドル/core-hourです。価格については要確認です。

Core-Hourなので課金形態は例えば以下のようになります。

+ 1台の32コアのサーバーで1時間実行した場合
  + 1台 × 32コア × 1時間 × 0.10/core-hour = 3.20 米ドル
+ 200台の4コアのサーバーで6分実行した場合
  + 200台 × 4コア × 0.1時間 × 0.10/core-hour = 8.00 米ドル

## ライセンスの作成方法

上記で書いたようにAWS向けには専用のライセンスが必要になります。作り方としては自分のRhinoのアカウント内に専用のチームを作成します。

1. [Rhino Accounts](https://accounts.rhino3d.com/) にアクセスする
2. ライセンスのページに行く
3. 新規チームを作成する
4. チームの管理 から コア時間課金の管理 を選ぶ
5. コア時間課金を有効にして保存する


## Compute.Rhino3dの実行環境構築

公式のドキュメント [Installation](https://github.com/mcneel/compute.rhino3d/blob/master/docs/installation.md)

1. ここから [最新のビルド](https://ci.appveyor.com/api/projects/mcneel/compute-rhino3d/artifacts/compute.zip?branch=master&pr=false) のデータをダウンロードして回答する
2. RhinoWIP(Rhino7)を少なくとも1回起動する（ライセンス認証するため）
3. PowerShell を起動してCompute.Rhino3dのフォルダ内にある compute.frontend.exe を実行する
4. ブラウザーで http://localhost/version にアクセスして例えば以下のようにバージョンが表示されれば問題なく実行されています

    ```
    {
      "rhino":"7.0.20259.15365",
      "compute":"1.0.0.493",
      "git_sha":"a612c257"
    }
    ```
この手順はAWSに限らずローカル（windows10）でテストする場合なども同じです。

# サンプルを実行