+++
author = "すかい"
title = "Jupyter Notebook上でCanvasを利用して画像を生成する"
date = "2017-05-14"
description = "Jupyter Notebook上でCanvasを利用して画像を生成する"
tags = [
    "Jupyter Notebook",
    "Python",
]
+++

## はじめに

Jupyter Notebook上でCanvasを使って画像を作って読み込めない？と相談を受けて調べたら

```py
from IPython.core.display import HTML
```

を使うとHTMLをセルの結果表示部分に表示させることが出来るのでトライしたメモ

成果物はgistにあげてるのでご興味があればどうぞ

- [js_canvas.ipynb](https://gist.github.com/skyblue3350/5306034f87c54e119f578bae6e03066f)

Python3でテストしましたが依存してそうなのはByteIOくらいなのでStringIOに置き直せば2系でも動くと思います

## キャンバスを生成する

先程のモジュールを使って要素を生成します
ブラウザで動かせるものならなんであれ使えるはずなのでjqueryで動作するもっと高機能なものを持ってきても良いと思います

```html
<canvas id="canvas" height="300px" width="300px" style="border: 1px solid;"></canvas>
<p><input id="variable" type="text" placeholder="Input python variable name" size="30"></p>
<p>
    <button id="clear">Clear</button>
    <button id="submit">Save image to variable</button>
</p>
```

固定の変数名に書き出す場合は不要ですがPython側の変数名を指定するインプットボックスとキャンバスをクリアするボタンを用意して置きます

## キャンバスにお絵かきできるようにする

良い感じにJSを書いてお絵かきできるようにします
この辺のデバッグは普通にHTML書いてやった方が幸せな気がします

```js
    var config = {
        "linesize": 7,
        "linecolor": "#000000"
    }

    var mouse = {
        "X": null,
        "Y": null,
    }

    var canvas = document.getElementById("canvas");
    var ctx = canvas.getContext("2d");
    canvas.addEventListener("mouseup", drawEnd, false);
    canvas.addEventListener("mouseout", drawEnd, false);
    
    canvas.addEventListener("mousemove", function(e){
        if (e.buttons === 1 || e.witch === 1) {
            var rect = e.target.getBoundingClientRect();
            var X = e.clientX - rect.left;
            var Y = e.clientY - rect.top;
            draw(X, Y);
        };
    });
 
    canvas.addEventListener("mousedown", function(e){
        if (e.button === 0) {
            var rect = e.target.getBoundingClientRect();
            var X = e.clientX - rect.left;
            var Y = e.clientY - rect.top;
            draw(X, Y);
        }
    });

    function draw(X, Y) {
        ctx.beginPath();
        if (mouse.X === null) {
            ctx.moveTo(X, Y);
        } else {
            ctx.moveTo(mouse.X, mouse.Y);
        }
        ctx.lineTo(X, Y);
        
        ctx.lineCap = "round";
        ctx.lineWidth = config.linesize * 2;
        ctx.strokeStyle = config.linecolor;
        ctx.stroke();

        mouse.X = X;
        mouse.Y = Y;
    };
 
    function drawEnd() {
        mouse.X = null;
        mouse.Y = null;
    }
```

## キャンバスのクリア

ボタン押したらclearRectを実行するだけ

```js
    var clear = document.getElementById("clear");
    clear.addEventListener("click", function(){
        ctx.clearRect(0, 0, canvas.width, canvas.height);
    });
```

## 変数への書き出し

JS -> Pythonへはとりあえずbase64を使用した
JSからPythonのコードを実行できる
例えば

```js
var kernel = IPython.notebook.kernel;
kernel.execute("hoge = 2")
```

とするとPython側のhoge変数に2が代入される
これをボタンのイベントと紐付ける

```js
submit.addEventListener("click", function(){
    kernel.execute(variable.value + " = '" + canvas.toDataURL() + "'");
}
```

## Python側でbase64をデコードする

受け取った変数から余分な部分を取り除いてデコードする

```py
base64_img = base64_img.split(",")[-1]
img = Image.open(BytesIO(base64.b64decode(base64_img)))
plt.imshow(np.asarray(img))
plt.show()
```

こんな感じでPython側で画像が利用できる
色々夢が広がる
