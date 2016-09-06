---
date: 2016-09-07T00:25:18+09:00
draft: true
title: 減衰のインタラクションアルゴリズム
---
マウスの移動量に応じて値が大きくなり、徐々に値が0に戻っていく減衰のアルゴリズムを紹介する。
コードはopenFrameworks。

`ofApp.h`
```cpp
class ofApp : public ofBaseApp{
    public:
        float velocity = 0;
        float smooth = 0.9;
        ofVec2f pMouse;
};
```

`ofApp.cpp`
```cpp
void ofApp::update(){
    ofVec2f m(mouseX, mouseY);
    ofVec2f dir = m - pMouse;

    velocity = smooth * velocity + smooth * dir.x;
    pMouse.set(mouseX, mouseY);
}

void ofApp::draw(){
    ofDrawRectangle(mouseX - velocity, 0, 30 - velocity, ofGetHeight());
}
```

![](../../../images/of/attenuation.gif)


参考：https://github.com/ofZach/funkyForms/blob/e79b4728eac983530a16e7ebae33ed2f52fb5e13/sketches/Character/src/ofApp.cpp#L11
