---
date: 2017-07-29T16:30:44+09:00
draft: true
title: glsl-livecoderを正常に使えるようにするまで
---

[glsl-livecoder](https://github.com/fand/glsl-livecoder)はglslでライブコーディングを行うためのAtomパッケージである。  
正常に動作できるようになるまでにやや手こずったのでメモしておく。

実行環境は、`macOS 10.12.5`。

# glsl-livecoderのインストール
```
$ apm install glsl-livecoder
```
だが、Atomの設定で`glslangValidator`のパスを指定しないと正常に動かない。  
自分は`glslangValidator`をインストールしてなかったので、次のステップで方法を紹介。  
なかなかめんどくさい。

# glslangValidatorのインストール
基本的にリポジトリに記載されている[ビルドステップ](https://github.com/KhronosGroup/glslang#build-steps)通りにやれば良いわけだが、わかりづらい。  
なので、以下に自分がやったステップを紹介する。

### 1. [KhronosGroup/glslang](https://github.com/KhronosGroup/glslang)リポジトリをクローン

### 2. [CMake](https://cmake.org/)のインストール
CMakeはクロスプラットフォームのビルドツール。  
自分は、[ダウンロードページ](https://cmake.org/download/)から`cmake-3.9.0-Darwin-x86_64.dmg`をインストールした。

### 3. CMakeをコマンドラインから使えるよう設定
```
$ sudo "/Applications/CMake.app/Contents/bin/cmake-gui" --install
```
これで`$ cmake`が使えるようになる。  
この設定コマンドは、`CMake.app`の`Tools/How to Install For Command Line Use`に記載されている。

### 4. `glslang`リポジトリの[ビルドステップ](https://github.com/KhronosGroup/glslang#build-steps)に沿って実行
[1) Check-Out this project](https://github.com/KhronosGroup/glslang#1-check-out-this-project)  
すでにクローンしてるのでスキップ。

[2) Check-Out External Projects](https://github.com/KhronosGroup/glslang#2-check-out-external-projects)  
必要なステップだったか正直わからないが、一応1)で作ったディレクトリ内にクローンしておいた。

[3) Configure](https://github.com/KhronosGroup/glslang#3-configure)  
Mac上での作業だったが、wnidow向けとして書かれていたステップで実行。  
glslangをクローンしたディレクトリ上で下記コマンドを実行。

```
$ cmake ./ -DCMAKE_INSTALL_PREFIX=`pwd`/install
```

[4) Build and Install](https://github.com/KhronosGroup/glslang#4-build-and-install)  
こちらも`for Windows`と書かれている方で実行。

```
$ cmake --build . --config {Release|Debug|MinSizeRel|RelWithDebInfo} \ --target install
```

ただ、ドキュメントに記載されてた上記コマンドだと諸々のコマンドが見つからない旨のエラーが出た。

```
-bash: Debug: command not found
-bash: MinSizeRel: command not found
-bash: RelWithDebInfo}: command not found
make[2]: *** [hlsl/CMakeFiles/HLSL.dir/depend] Error 141
make[1]: *** [hlsl/CMakeFiles/HLSL.dir/all] Error 2
make: *** [all] Error 2
```

なので、congif指定を外して実行。

```
$ cmake --build . --target install
```

```
...
-- Installing: /Users/r21nomi/workspace/shader/glslang/install/bin/glslangValidator
...
```

`glslangValidator`がインストールされたので、これで準備完了。

# glsl-livecoderでglslangValidatorのパスを設定
Atomの`Preferences -> glsl-livecoderのSettings`から`glslangValidator path`欄にインストールした`glslangValidator`のパスを設定。  
自分の場合は`/Users/r21nomi/workspace/shader/glslang/install/bin/glslangValidator`。

これでシェーダーファイルを開いて`ctrl-enter`とかやるとシェーダープログラムがコンパイルされて、背景に描画されるようになる。
