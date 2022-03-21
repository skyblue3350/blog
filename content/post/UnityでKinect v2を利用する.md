+++
author = "すかい"
title = "UnityでKinect v2を利用する"
date = "2016-10-31"
description = "UnityでKinect v2を利用する"
tags = [
    "Csharp",
    "Unity",
]
+++

## はじめに

UnityでKinect v2を利用するメモ
画像を出したりだとかはいっぱいサンプルあるので単純に人を検出するだけのシンプルなコード

## バージョン

- Unity 5.3
- Kinect v2 2.0.1410

## 必要なライブラリ

Kinect v2のSDK（ https://www.microsoft.com/en-us/download/confirmation.aspx?id=44561 ）を入れた上で
http://download.microsoft.com/download/F/8/1/F81BC66F-7AA8-4FE4-8317-26A0991162E9/KinectForWindows_UnityPro_2.0.1410.zip
からUnity用のパッケージを落とす

Kinect.2.0.1410.19000.unitypackageをインポートして全部入れる

## ソース

### 起動と終了時の処理

最低限のコードだけならこんな感じ

`Kinect.cs`

```csharp
using UnityEngine;
using System.Collections;
using System.Collections.Generic;

using Windows.Kinect;

public class Kinect : MonoBehaviour {
    KinectSensor _Sensor;
    BodyFrameReader _Reader;
    Body[] _Data = null;

    void Start () {
        _Sensor = KinectSensor.GetDefault();
        if (_Sensor != null){
            _Reader = _Sensor.BodyFrameSource.OpenReader();
            if (!_Sensor.IsOpen){
                _Sensor.Open();
            }
        }
    }

    void OnApplicationQuit(){
        if (_Reader != null){
            _Reader.Dispose();
            _Reader = null;
        }
        if (_Sensor != null){
            if (_Sensor.IsOpen){
                _Sensor.Close();
            }
            _Sensor = null;
        }
    }
}
```

### フレームの取得

必ずDisposeメソッドを呼ぶこと
呼び忘れると初回以降AcquireLatestFrameメソッド呼ぶとひたすらnullが帰されて悲しいことになる
AcquireLatestFrameは結構null返ってくるから他の方法で実装出来るらしいからそっちの方が良いかも
体感3回に1回くらいはnullな感じする

```csharp
void Update(){
    var frame = _Reader.AcquireLatestFrame();
    if (frame == null){
        return;
    }
    frame.Dispose();
    frame = null;
}
```

### 人間の検知

```csharp
void Update(){
    var frame = _Reader.AcquireLatestFrame();
    if (frame == null){
        return;
    }
    if (_Data == null){
        _Data = new Body[_Sensor.BodyFrameSource.BodyCount];
    }
    frame.GetAndRefreshBodyData(_Data);
    frame.Dispose();
    frame = null;

    if (_Data == null){
        return;
    }

    foreach(var body in data){
        if (body == null) {
            continue;
        }
        if (body.IsTracked){
            Debug.Log("人がいるよ！");
            // RefreshBodyObject(body);
        }
    }
}
```

### ボーンの取得

色々大変

```csharp
static Vector3 GetVector3FromJoint(Windows.Kinect.Joint joint) {
    return new Vector3(joint.Position.X * 10, joint.Position.Y * 10, joint.Position.Z * 10);
}

void RefreshBodyObject(Windows.Kinect.Body body){
    for (Windows.Kinect.JointType jt = Windows.Kinect.JointType.SpineBase; jt <= Windows.Kinect.JointType.ThumbRight; jt++) {
        Windows.Kinect.Joint sourceJoint = body.Joints[jt];
        Vector3 tmp = GetVector3FromJoint(sourceJoint);
        Debug.Log("name:"+jt);
        Debug.Log("x:"+tmp[0]);
        Debug.Log("y:"+tmp[1]);
        Debug.Log("z:"+tmp[2]);
    }
}
```

あとは煮るなり焼くなりお好きに
長い
