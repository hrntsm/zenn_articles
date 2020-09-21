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

AWS EC2でのWindows Server 2019のインスタンスの作り方はいろいろなサイトで紹介されているのでここではざっくり説明します。

1. aaa
2. aaa
3. aaa
4. aaa

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

あああ

## Compute.Rhino3dの実行環境構築

公式のドキュメント [Installation](https://github.com/mcneel/compute.rhino3d/blob/master/docs/installation.md)


あああ