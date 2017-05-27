---
date: 2017-05-27T14:22:26+09:00
draft: true
title: git管理下から除外したgoogle-services.jsonをCircle CIから読み込む
---
## TL;DR
- jsonファイルをbase64化してテキストにする
- それをCircle CIの環境変数に設定
- `circle.yml`でjsonファイルにデコードする命令を書く

## 概要
`google-services.json`はFirebaseなどのgoogleサービスと連携するために必要なファイルだが、これはプライベートな情報であるためGithub等のpublicリポジトリの管理下に置くべきではない。

だが、単純にファイルを除外するだけだとCIサービスからファイルを参照できなくなり、ビルドに失敗する。

Circle CIでは[ドキュメントに方法が記載されている](https://circleci.com/docs/1.0/google-auth/)が、Firebaseの場合、単純にこの通りではうまくいかないので、本記事ではファイルを除外し、Circle CIで正常にビルドできるようにするまでをメモしておく。

## 手順
### google-services.jsonの除外
`.gitignore`に`google-services.json`を追加すればok。

### google-services.jsonをbase64化
ターミナルから以下コマンドを実行。
```
$ openssl base64 -in <SOURCE_FILE_NAME> -out <OUT_FILE_NAME>
```

### Circle CIの環境変数に登録
Circle CIプロジェクト設定の`Environment Variables`に追加する。

例えば、こんな感じ。

`GOOGLE_SERVICES_JSON` `xxxxxxxxx...`

### circle.ymlの設定
設定した環境変数のテキストをjsonファイルにデコードする処理を書く。
```
dependencies:
  pre:
    ...
    - echo $GOOGLE_SERVICES_JSON | base64 --decode --ignore-garbage > ${HOME}/${CIRCLE_PROJECT_REPONAME}/app/google-services.json
```
[CircleCIのドキュメント](https://circleci.com/docs/1.0/google-auth/)では`${HOME}/gcloud-service-key.json`となっているが、これだと`no such file or directory`的なエラーが出るので、プロジェクト名とモジュール名（Firebaseで利用する際にはこの直下にjsonファイルを置くことになるので）も指定する必要がある。

`app`まで指定しないとデコード処理は通るが、アプリのビルド処理でファイルが見つからずコケることになる。

ちなみに、`${HOME}`は`/home/ubuntu`、`${CIRCLE_PROJECT_REPONAME}`はプロジェクト名が返る。

これで正常にビルドできるようになった。
