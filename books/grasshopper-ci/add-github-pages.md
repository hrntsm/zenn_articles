---
title: "作ったものを紹介するページを作る"
---

# はじめに

Github Pages をつかうと簡単にサイトが作成できます。
ここで作ったコンポーネントを紹介するサイトを作りましょう。

# 作り方

Github の対象のリポの Settings を開きます。
スクロールして下のほうに行くと Github Pages という欄があります。そこで以下のように設定してください。

![](https://github.com/hrntsm/zenn_articles/blob/master/books/grasshopper-ci/image/Pages.png?raw=true)

次にその下の Theme Chooser の部分の Choose a Theme を選択すると、作成するページのテーマが選べます。
好きなものを選んでください。
ここでは例として最初に選択されている Cayman theme を選択してまずはそのままコミットします。
そうすると裏で CI が走って、docs/index.md の内容をもとにページが作成されます。
作成されたページはリポの右にある Environments からアクセスできます。

![](https://github.com/hrntsm/zenn_articles/blob/master/books/grasshopper-ci/image/environment.png?raw=true)

ページの URL は以下のルールで作成されます。

https:// YOUR_ACOUNT_NAME .github.io/REPOSITORY_Name/

ページの内容は、マークダウンでかけるので適宜紹介したいなりにページを作成してください。ちなみにこれは設定したフォルダ(ここでは main ブランチの doc)にあるものをページにしているだけなので、ここに HTML などを直接おいてもページが作成されます。

# 例

参考に作成したページを以下に示します。

index.md の内容 [GrasshopperCISample/docs](https://github.com/hrntsm/GrasshopperCISample/tree/main/docs)

デプロイされたサイト [GrasshopperCISample](https://hrntsm.github.io/GrasshopperCISample/)
