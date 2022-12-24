---
title: "JavaScript で Photoshop 操作の自動化をする"
emoji: "📸"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["Photoshop"]
published: false
---

この記事は [mirko-san のアドベントカレンダー Advent Calendar 2022](https://adventar.org/calendars/8051) の 12月 24日の記事です。

## TL;DR

- Adobe Photoshop の自動化が JavaScript でできるので試した
- スクリプトの実行をさらにドロップレットで楽したので紹介

## 前提環境

- Windows 11
- Adobe Photoshop CC 2022

## 経緯

この間自分の描いた絵を印刷所に入稿しようとしたところ、入稿作業の一部は自動化できそうだなという予感があり、一部自動化をしました。

https://zenn.dev/mirko_san/articles/f063ffca91f8b4

本稿は上記の記事の続きとして、 Adobe Photoshop の自動化をしてみた記事となります。

## 最終的にできあがったもの

ドロップレット（後述）に psd ファイルをドラッグアンドドロップすると、以下の処理をされた `<元のファイル名>_flatten.psd` が生成されるようになった。

- すべてのパスを削除
- 画像の結合
  - このとき、`_` で始まるレイヤーセットは除外
- レイヤー名を `image` に変更

![](/images/eceacdbd355964/w3H0EBdzvt.png)

### 具体的には

これが

![](/images/eceacdbd355964/XFkiTPCQVc.png)

こうなる

![](/images/eceacdbd355964/b8jCdDWyGq.png)

## やったこと

### JavaScript を書く

Adobe によると、どうやら JavaScript 以外にも VB Script もあるらしい。まあ JavaScript のほうがマシなので JavaScript を使うこととする。

https://helpx.adobe.com/jp/photoshop/using/scripting.html

どうやらドキュメントは以下で管理されているらしい。
https://github.com/Adobe-CEP/CEP-Resources/tree/master/Documentation/Product%20specific%20Documentation/Photoshop%20Scripting

筆者の環境は Adobe Photoshop CC 2022 なので、現在最新に見える 2020 のドキュメントを参照した。

https://github.com/Adobe-CEP/CEP-Resources/blob/master/Documentation/Product%20specific%20Documentation/Photoshop%20Scripting/photoshop-javascript-ref-2020.pdf

出来上がったものがこちら。
https://github.com/mirko-san/photoshop_scripts/blob/cf8d32b520ce67ab623300b7fb6f4e7583c8f6c4/m_flatten.jsx#L1-L47

このスクリプトを `C:\Program Files\Adobe\Adobe Photoshop 2022\Presets\Scripts` 以下に `m_flatten.jsx` として配置する。

筆者は既存のスクリプトを誤って消したりしたくなかったので、自作スクリプトは `C:\Program Files\Adobe\Adobe Photoshop 2022\Presets\Scripts\original_scripts` 以下に置くこととした。

Photoshop を再起動すると「ファイル > スクリプト」から自作のスクリプトが選択できるようになっているはず。（今回は `m_flatten` が追加）

![](/images/eceacdbd355964/Fa7hsRFc1D.png)

### アクションを追加

新規アクションを追加する

![](/images/eceacdbd355964/cYk8wYEqzR.png)

アクション記録モードになるので、「ファイル > スクリプト > m_flatten」と選択していく。
スクリプトの実行が終わったらアクションの記録を終了する。

![](/images/eceacdbd355964/dZIX2eTHyH.png)

### ドロップレットを作成

ドロップレットとは、特定のファイルにひとつのアクションを適用するのに便利なやーつである。

> ドロップレットでは、ドロップレットアイコンに画像ファイルやフォルダーをドラッグ＆ドロップすると 1 つのアクションを適用することができます。

https://helpx.adobe.com/jp/photoshop/kb/5772.html

これを使って、お手軽に先ほど作ったアクションを実行できるようにする。

「ファイル > 自動処理 > ドロップレットを作成」と選択。

画像の通りに設定し「OK」を押す。

![](/images/eceacdbd355964/yqJPujehL4.png)

ドロップレット保存先に指定したフォルダにドロップレットが作成される。

:::message
以下の工程は、もしかしたら不要な方もいるかもしれません。
もしドロップレットに psd ファイルをドラッグアンドドロップしてみて「ドロップレットとphotoshopが通信できません」とウィンドウが出た方はお試しください。
:::

「ドロップレットを右クリック > プロパティ > 互換性 > "管理者としてこのプログラムを実行する" にチェック > OK」
上記を行ったのち、 PC を再起動。

## 使ってみた感想

アクションのデバッグをすると分かるのですが、 JavaScript での自動化は Photohop の操作をコード化したという感じでかなり手続き的でした。
たとえば、レイヤーの結合を繰り返して（おそらく）操作のフォーカスが非表示レイヤーに移ったとき「表示レイヤーを結合」が実行できなかったりします。
（実際の画面で非表示レイヤーの右クリックから「表示レイヤーを結合」が選択できないのと同じ利用感）
このあたりはあまり SDK 的な実装ではないなあと感じましたが、逆に実際の画面操作が分かっていれば迷うことは無いということでいい点かもしれません。
