---
date: 2015-03-15T23:55:39+09:00
draft: true
title: 【Android】ValueAnimatorを使ったアイテム削除アニメーション
---
アイテムを削除するときは、ユーザーに処理の完了を認知させるためにアニメーションを入れるのが良い。  
わかりやすいアニメーションとしては、

1. フェードアウト  
2. 両サイドのアイテムの間隔をつめる。  
3. アイテム削除  

が良いと思われる。  

今回は、この処理をValueAnimatorを使って実装する。  

### ValueAnimatorとは  
[ValueAnimator](http://developer.android.com/reference/android/animation/ValueAnimator.html)はオブジェクトのアニメーションクラスである。  
時間内の各フレームにおいてリスナー（`Animator#addUpdateListener`）を設定できるので、一つのValueAnimatorの中で複数のViewをアニメーションさせることができる。  

### 簡単なアニメーション  
500msかけてフェードアウトするアニメーションをつくる。  

```
ValueAnimator animator = ValueAnimator.ofFloat(1.0f, 0f).setDuration(500);
animator.addUpdateListener(new ValueAnimator.AnimatorUpdateListener() {
    @Override
    public void onAnimationUpdate(ValueAnimator animation) {
        imgView.setAlpha((Float) animation.getAnimatedValue());
    }
});
animator.addListener(new AnimatorListenerAdapter() {
    @Override
    public void onAnimationEnd(Animator animation) {
        super.onAnimationEnd(animation);
        Toast.makeText(getApplicationContext(), "Animated", Toast.LENGTH_SHORT).show();
    }
});
animator.start();
```

`upDateListener` の分だけやや冗長にはなるが、`ObjectAnimator`のように`onAnimationStart`, `onAnimationEnd`, `onAnimationCancel`, `onAnimationRepeat`と大量のリスナーを必要とせず、`onAnimationEnd`のみセットできるのが良い。  

### フェードアウトしながら削除するアニメーション  

本題は下記。  
ValueAnimatorを２つ用意し、  

1. フェードアウト  
2. widthを0にする  
3. アニメーション完了したら削除処理  

という流れである。  

```
ValueAnimator widthAnimator = ValueAnimator.ofInt(imgView.getWidth(), 1).setDuration(150);
// アニメーションしながらwidthを0にする
widthAnimator.addUpdateListener(new ValueAnimator.AnimatorUpdateListener(){
    @Override
    public void onAnimationUpdate(ValueAnimator animation) {
        ViewGroup.LayoutParams layoutParams = imgView.getLayoutParams();
        layoutParams.width                  = (Integer) animation.getAnimatedValue();
        imgView.setLayoutParams(layoutParams);
    }
});
// アニメ−ション完了後にアイテムを入れ替える
widthAnimator.addListener(new AnimatorListenerAdapter() {
    @Override
    public void onAnimationEnd(Animator animation) {
        super.onAnimationEnd(animation);
        // 削除処理
    }
});

ValueAnimator alphaAnimator = ValueAnimator.ofFloat(0f, 1.0f).setDuration(500);
alphaAnimator.addUpdateListener(new ValueAnimator.AnimatorUpdateListener() {
    @Override
    public void onAnimationUpdate(ValueAnimator animation) {
        imgView.setAlpha((Float) animation.getAnimatedValue());
    }
});
alphaAnimator.addListener(new AnimatorListenerAdapter() {
    @Override
    public void onAnimationEnd(Animator animation) {
        super.onAnimationEnd(animation);
        widthAnimator.start();
    }
});
alphaAnimator.start();
```

簡単ではあるが以上  

アニメーションはどうしてもネストが深くなりコードが汚くなりがちなので、[gradle-retrolambda](https://github.com/evant/gradle-retrolambda)なんか使うとラムダ式でネストがすっきりして良いかも。
