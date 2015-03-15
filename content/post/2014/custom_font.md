---
date: 2015-03-15T23:44:32+09:00
draft: true
title: 【Android】スクロールパフォーマンスを低下させるカスタムフォントの実装例とその回避策
---
Androidのデフォルトフォントってイケてない。  
なので好きなフォントに変更したいってケースがあるんだけど、ここで実装に気をつけないとスクロールパフォーマンスの低下を招いてしまう。  

以下がその例。  
（CustomViewとしてつくる）  

```
public class CustomFontTextView extends TextView {
    private static final String FONT_PATH_PREFIX = "fonts/";
    private static final String DEFAULT_FONT_TYPE = "mplus-1c-light.ttf";
    private Typeface mTypeface;

    public CustomFontTextView(final Context context) {
        super(context);
    }

    public CustomFontTextView(final Context context, final AttributeSet attrs) {
        super(context, attrs);
    }

    public CustomFontTextView(final Context context, final AttributeSet attrs, final int defStyle) {
        super(context, attrs, defStyle);

        TypedArray typedArray = context.obtainStyledAttributes(attrs, R.styleable.CustomFontTextView);

        try {
            String customFontType = typedArray.getString(R.styleable.CustomFontTextView_custom_font_type);

            if (StringUtils.isNotBlank(customFontType)) {
                mTypeface = Typeface.createFromAsset(context.getAssets(), FONT_PATH_PREFIX + customFontType);
            } else {
                mTypeface = Typeface.createFromAsset(context.getAssets(), FONT_PATH_PREFIX + DEFAULT_FONT_TYPE);
            }
            setTypeface(mTypeface);
        } finally {
            typedArray.recycle();
        }
    }
}
```

下記のように使う。
```
<com.example.app.view.CustomFontTextView
android:layout_width="wrap_content"
android:layout_height="wrap_content"/>
```

フォントを適用する処理は`Typeface.createFromAsset()`によって行われるわけだが、上の例ではこのメソッドがViewの生成の度に実行されてしまう。  
つまり、毎回フォントを読み込んでViewのプロパティにセットする処理が走るので、スクロールがカクついてしまうというわけだ。  

ではこれを回避するにはどうするか。  

キャッシュしてしまおう。  
一度読み込んだフォントはハッシュに追加して、次回からはそれを読み込むようにすれば良い。  

```
private static final String FONT_PATH_PREFIX = "fonts/";
private static final String DEFAULT_FONT_TYPE = "mplus-1c-light.ttf";
// 読み込んだフォントをキャッシュさせるHashtable
private static Hashtable<String, Typeface> sTypeFaces = new Hashtable<String, Typeface>();

// 略...

private void setFont(Context context, AttributeSet attrs) {
    TypedArray typedArray = context.obtainStyledAttributes(attrs, R.styleable.CustomFontTextView);
    String font = typedArray.getString(R.styleable.CustomFontTextView_custom_font_type);

    if (StringUtils.isBlank(font)) {
        font = DEFAULT_FONT_TYPE;
    }

    Typeface typeface = sTypeFaces.get(font);

    // フォントがキャッシュ（Hashtable）にない場合は追加
    if (typeface == null) {
        typeface = Typeface.createFromAsset(getContext().getAssets(), FONT_PATH_PREFIX + font);
        sTypeFaces.put(font, typeface);
    }

    setTypeface(typeface);

    typedArray.recycle();
}
```

`sTypeFaces.get(font);`の部分でハッシュテーブルから読み込んだフォントを取り出している。  
リクエストしたものが見つからなければ新たに読み込んでハッシュテーブルに追加するという仕組みだ。  

こうすることでカスタムフォントの読み込みによるスクロールパフォーマンス低下を回避することができる。  

たったこれだけなのだが、この処理に変えたら全体の操作感が格段に上がった（以前がクソすぎた）。  
こういうところ、気をつけないとアプリの質を下げかねないのでしっかりやらないとね。  

参考：[http://stackoverflow.com/questions/15338016/performance-issue-on-custom-font-textview](http://stackoverflow.com/questions/15338016/performance-issue-on-custom-font-textview)
