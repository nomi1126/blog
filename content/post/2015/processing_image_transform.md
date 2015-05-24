---
date: 2015-05-24T23:53:06+09:00
draft: true
title: 【Processing】Image transform
---
Processingの勉強としてFORM+CODEの[TRANSCODED LANDSCAPE](http://formandcode.com/code-examples/transform-landscape)をカスタマイズしてみた。  
画像の各ピクセルを抽出してその色の値に応じてZ軸の長さを変えている。  
オリジナルからの変更点は、Z軸の長さを`sin()`で変更しているところと、カーソル移動で視点を切り替えるところぐらい。
![](../../../images/processing/image_transform.gif)
[コード](https://github.com/nomi1126/processing_work/blob/master/2015_05_17_image_transform/image_transform/image_transform.pde)
