---
date: 2015-03-01T04:52:35+09:00
draft: true
title: google-code-prettifyを使ってコードブロックをかっこいいスタイルに整形する
---
このライブラリ以外にも[SyntaxHighlighter](http://alexgorbatchev.com/SyntaxHighlighter)とか強力なものもあるが、複数のスクリプトを読む必要があったり（結合すればいいんだが）でややめんどくさい。  
その点、google-code-prettifyは１スクリプトの読み込みだけでOKだし、シンプルなのでこれで十分だと思ってる。  
### 準備
下記スクリプトを追加しておく。  

```
  <script src="https://google-code-prettify.googlecode.com/svn/loader/run_prettify.js"></script>
```


### 利用  
表示させるコードをタグで囲う。  

```
<pre class="prettyprint linenums">
    function say(str) {
        console.log("hello " + str);
    }
    say("world");
</pre>

```

表示は次のようになる。  

```
function say(str) {
    console.log("hello " + str);
}
say("world");
```

スクリプトがクラス`prettyprint`を見つけ出してスタイルをあてている。  
クラス`linenums`は行番号の表示を意味する。  

### カスタマイズ  
エディタで毎回コードを`<pre class="prettyprint linenums"></pre>`で囲うのは面倒なので、`<pre></pre>`を見つけたら動的に`prettyprint linenums`クラスを付加するようにする。  

```
<script type="text/javascript">
    $("pre").addClass("prettyprint linenums");
</script>
```

### まとめ  
google-code-prettifyを用いてコードブロックをかっこよく整形した。  
外部エディタを用いてMarkdownで書いたものをtumblrに投稿するとエディタによって展開のされ方が違ったりしてやや手こずった。  
今度はHTML/CSS/Javascriptをセットにしたモジュールとか表示させてみようかな。
