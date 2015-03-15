---
date: 2015-03-15T19:26:42+09:00
draft: true
title: 【Android】SurfaceViewで複数パーティクルを描画
---
Androidでパーティクルを描画する方法を調べてたら、[SurfaceView](http://developer.android.com/reference/android/view/SurfaceView.html)を使う方法を見つけたのでその実装方法をメモしておく。  
### SurfaceViewとは  
すでにいろんな記事で書かれていることではあるが、`SurfaceView`とは`View`のサブクラスで、ゲームのアニメーションとか負荷の高い処理に向いているそう。  

### 使い方  
`SurfaceView`を継承したサブクラスを用意して、必要なメソッドをimplementsする。  
```
public class AnimationSurfaceView extends SurfaceView implements Runnable, SurfaceHolder.Callback {
    public AnimationSurfaceView(Context context) {
        // 
    }

    @Override
    public void run() {
        // 毎回実行
        // ここでパーティクルの描画を行う。
    }

    @Override
    public void surfaceCreated(SurfaceHolder holder) {
        // Surfaceが構築された時
    }

    @Override
    public void surfaceChanged(SurfaceHolder holder, int format, int width, int height) {
        // Surfaceが変化したとき
    }

    @Override
    public void surfaceDestroyed(SurfaceHolder holder) {
        // Surfaceが破棄された時
    }
}
```

### パーティクルを描く  
パーティクルの描画には、`canvas.drawCircle()`を使う。  
第一引数から順に、パーティクルのx座標, パーティクルのy座標, 半径, paintインスタンスとなる。  
これを`run()`の中で実行すれば画面に描画することができる。  

```
@Override
public void run() {
    Canvas canvas = null;
    Paint paint   = new Paint();

    paint.setStyle(Style.FILL);
    paint.setColor(Color.BLACK);

    while(thread != null){
        try{
            canvas = surfaceHolder.lockCanvas();
            canvas.drawCircle(100, 100, 50, paint);
            surfaceHolder.unlockCanvasAndPost(canvas);
        }
        catch(Exception e){}
    }
}
```

### 複数のパーティクルの描画  
描画するパーティクルを配列に格納し、`run()`中のループの中で描画していく。  
ここでは画面端で跳ね返る処理と跳ね返る度に色が変わる処理も追加してある。  

```
public class AnimationSurfaceView extends SurfaceView implements Runnable, SurfaceHolder.Callback {

    SurfaceHolder surfaceHolder;
    Thread thread;
    int screen_width,
    screen_height;

    ArrayList<Ball> ballList = new ArrayList<Ball>();

    public AnimationSurfaceView(Context context) {
        super(context);
        surfaceHolder = getHolder();
        surfaceHolder.addCallback(this);
    }

    @Override
    public void run() {
        Canvas canvas = null;
        Paint paint   = new Paint();
        Paint bgPaint = new Paint();

        // Background
        bgPaint.setStyle(Style.FILL);
        bgPaint.setColor(Color.WHITE);

        // Ball
        paint.setStyle(Style.FILL);

        while(thread != null){

        try{
            canvas = surfaceHolder.lockCanvas();

            // canvasを塗りつぶす（リセット効果）
            canvas.drawRect(0, 0, screen_width, screen_height, bgPaint);

            for (int i = 0, len = ballList.size(); i < len; i++) {
                Ball tempBall = ballList.get(i);

                int new_cx = tempBall.posX + tempBall.velocity * tempBall.dirX;
                int new_cy = tempBall.posY + tempBall.velocity * tempBall.dirY;

                // 跳ね返りの計算
                if (new_cx - tempBall.radius < 0 || new_cx + tempBall.radius > screen_width) {
                    tempBall.updateDirX();
                    tempBall.setColor();
                }
                if (new_cy - tempBall.radius < 0 || new_cy + tempBall.radius > screen_height) {
                    tempBall.updateDirY();
                    tempBall.setColor();
                }

                // 色を設定
                paint.setColor(tempBall.color);

                // ボールの位置を更新
                tempBall.setPosX(tempBall.velocity * tempBall.dirX);
                tempBall.setPosY(tempBall.velocity * tempBall.dirY);

                // 描画
                canvas.drawCircle(tempBall.posX, tempBall.posY, tempBall.radius, paint);
            }

            surfaceHolder.unlockCanvasAndPost(canvas);
        }
        catch(Exception e){}
        }

    }

    @Override
    public void surfaceChanged(SurfaceHolder holder, int format, int width, int height) {
        screen_width  = width;
        screen_height = height;
    }

    @Override
    public void surfaceCreated(SurfaceHolder holder) {
        Ball ball = new Ball(50, 200, 300, 30, 1, 1);
        ballList.add(ball);
        thread = new Thread(this);
        thread.start();
    }

    @Override
    public void surfaceDestroyed(SurfaceHolder holder) {
        thread = null;
    }

    // パーティクルの追加
    public void addBall(int radius, int x, int y, int velocity) {
        int dirArray[] = {1, -1};
        Random random  = new Random();
        Ball ball      = new Ball(radius, x, y, velocity, dirArray[random.nextInt(2)], dirArray[random.nextInt(2)]);
        ballList.add(ball);
    }
}
```

画面のタッチイベントで`addBall()`を実行すれば、タップするごとにパーティクルが増えていく。  

![](https://31.media.tumblr.com/4b3eeabfec1aaf1c9761d2d9cb7d5969/tumblr_inline_n92pkxqLOl1qcnxf5.png)  
コードは[こちら](https://github.com/nomi1126/SurfaceViewTest)


### まとめ  
SurfaceViewを使って複数のパーティクルを描画してみた。  
実機でもかなりサクサク動いたので満足している。  
因に、Android wearでもちゃんと動作するようだ。  
（タッチイベントは効かないのでパーティクルは追加されない）  
今度はwear上で複数パーティクルの描画をやってみようと思う。
