---
date: 2015-03-16T00:07:02+09:00
draft: true
title: HUGOをインストールしてみる
---
ブログはTumblrを使っているのだが、前からStaticSiteGenerator使ってみたいと思ってたので何か入れてみることにした。<br />
モダンなやつを使いたいってのと、ビルドが早いとの話を聞いたので[HUGO](http://gohugo.io/)を試すことにした。<br />
インストール時に若干手こずったのでメモしておく。

## インストール手順
1. [サイト](https://github.com/spf13/hugo/releases)からzipをダウンロードし、解凍<br />
Homebrewでも入れれるのだが、僕の場合エラーが出てしまったのでファイルをDLすることにした<br />
2. ファイルをリネーム<br />
解凍後に `hugo_0.12_darwin_amd64` みたいな名前の実行ファイルができるので、これを `hugo` とかにリネームする<br />
3. ファイルを移動<br />
パスを通すために先ほどの `hugo` ファイルを `/usr/local/bin` に移動する<br />
これで `hugo` コマンドを使えるようになる<br />
4. バージョン確認<br />
`hugo version` で `Hugo Static Site Generator v0.12` みたいなバージョンが表示されたらOK

## サイト作成
だいたい[ここ](http://gohugo.io/overview/quickstart)の通りに進めればOKだが、ちょこっと書いておく。

### サイトのルートディレクトリを作成

```
$ hugo new site ./path/to/site
```

必要最低限のファイルが生成される。<br />
最初、パスのところを`./`ではなく`/`から始めていて、僕の環境ではmkdirのpermission deniedが発生してちょっとハマったので注意

### 記事の作成

```
$ hugo new post/first.md
```

`content/post`以下に作成される。

### テーマの適用
#### テーマをインストール

```
$ git clone --recursive https://github.com/spf13/hugoThemes themes
```

シポジトリ上のすべてのテーマがインストールされるので気に入ったものだけインストるするのが良さそう。

#### 適用

```
$ hugo -t /theme内の名前
```

#### カスタムテーマ作成
試してないが、[このへん](http://gohugo.io/themes/creation/)に書いてある。

### 確認
#### ローカルサーバーを立ち上げて確認

```
$ hugo server --buildDrafts
```

`http://localhost:1313`でアクセスすれば見れるはずだ。

#### テーマを指定して確認

```
$ hugo server --theme=テーマ名 --buildDrafts
```

#### ウォッチ

```
$ hugo server --buildDrafts --watch
```

ローカルで記事を編集したらブラウザにすぐに反映される。
<br />

今回はローカル環境でサクッと試しただけだが、[GithubPages](https://pages.github.com/)とかにあげればすぐさま公開が可能だろう。<br />
またこんどやろうと思う。<br />

## ※追記 2015.02.28<br />
ローカルでのwatchのところで

```
$ hugo server --buildDrafts --watch
```

とすれば編集内容が即座に反映されると書いたが、自分のローカル環境だとこれだけでは反映されなかった。
下記のようにテーマを指定して行えば反映されるようになった。

```
$ hugo server --theme=hugo-incorporated --buildDrafts --watch
```

根本原因まで調べてないが、ちゃんとテーマを設定して公開すれば大丈夫なのかも。
