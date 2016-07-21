---
date: 2016-07-21T12:57:22+09:00
draft: true
title: oFからAndroidのメソッドを呼ぶ
---
oFからAndroidのメソッドを呼ぶには`ofxAndroidUtils`内のいつくかのメソッドを使う必要がある。

例えば、プロジェクトフォルダが`sampleProject`の場合、`ofApp.cpp`から`cc.openframeworks.sampleProject.OFActivity#getId`メソッドを呼ぶ処理を以下に書く。

`OFActivity.java`
```java
public class OFActivity extends cc.openframeworks.OFActivity{
	...
	// ofApp.cppからこのメソッドを呼びたい
    public int getId() {
        return 10;
    }
}
```

`ofApp.h`
```cpp
#include "ofxAndroidUtils.h"
...
```

`ofApp.cpp`
```cpp
void ofApp::setup(){
	// setup()は一度しか呼ばれないっぽい（バックボタンで戻った後に再度立ち上げても呼ばれない）
	ofLog(OF_LOG_NOTICE, "ID : " + ofToString(getId()));  // 10
}

// 2回目の起動以降はreloadTextures()が呼ばれるので、ここにも設定しておく必要がある
// resume()も２回目以降の起動で呼ばれるが、ここでofImage.load()とかすると画像が正しく設定されないので、reloadTextures()で処理するのが良い
void ofApp::reloadTextures(){
	ofLog(OF_LOG_NOTICE, "ID : " + ofToString(getId()));  // 10
}

// OFActivity#getIdを呼んで返り値のintを取得するメソッド
int ofApp::getId() {
    // Get env.
    JNIEnv *env = ofGetJNIEnv();
    if (!env) {
        ofLogError() << "Couldn't get environment using GetEnv()." << endl;
        return -1;
    }

    // Find reference for OFActivity.
    jclass localClass = env->FindClass("cc/openframeworks/sampleProject/OFActivity");
    jclass javaClass = (jclass) env->NewGlobalRef(localClass);
    if(!javaClass){
        ofLogError() << "Couldn't get java class for OFActivity." << endl;
        return -1;
    }
    jobject javaObject = ofGetOFActivityObject();
    javaObject = (jobject) env->NewGlobalRef(javaObject);
    if (!javaObject) {
        ofLogError() << "javaObject not found." << endl;
        return -1;
    }

    // Find getId method from OFActivity.
    jmethodID javaGetIdMethod = env->GetMethodID(javaClass,"getId","()I");
    if(!javaGetIdMethod){
        ofLogError() << "Couldn't get java getId from OFActivity." << endl;
        return -1;
    }

    return env->CallIntMethod(javaObject, javaGetIdMethod);
}
```

参考：[https://github.com/cpietsch/cordova-plugin-opcvExample/blob/c15d91225caff536ea9425908f61f2b9a2dc2892/of_v0.9.0_android_release/addons/ofxAndroidWebView/src/ofxAndroidWebView.cpp](https://github.com/cpietsch/cordova-plugin-opcvExample/blob/c15d91225caff536ea9425908f61f2b9a2dc2892/of_v0.9.0_android_release/addons/ofxAndroidWebView/src/ofxAndroidWebView.cpp)

[methodsignatureの種類について](http://stackoverflow.com/questions/30815284/whats-method-signature-parameter-when-calling-a-java-method-using-jni)
```
Type Signature   Java Type
Z                boolean
B                byte
C                char
S                short
I                int
J                long
F                float
D                double
L fully-qualified-class ;   fully-qualified-class
[ type           type[]
( arg-types ) ret-type method type
```
