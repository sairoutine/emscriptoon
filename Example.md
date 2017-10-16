# Emscripten を使ったプロジェクト

Emscripten を使うと、どのようなことができるでしょうか。
本稿では、Emscripten を利用した既存のプロジェクトについてご紹介します。

## Ammo.js
URL:
https://github.com/kripken/ammo.js

ammo.js は Bullet と呼ばれるオープンソース C++製 物理演算エンジンを、
JavaScript に移植したプロジェクトです。

ammo.js と WebGL を利用することで、重力と地面の存在する 3D 空間にて
物体の衝突を描画できます。

とくに、3D ゲームなどを実装する際に役に立ちます。

## ocrad.js
URL:
https://github.com/antimatter15/ocrad.js

ocrad.js は GNU製のOCRであるOcradをEmscriptenを使ってJavaScriptに移植したプロジェクトです。
画像に書かれた文字を、テキストとして取得できます。

* サンプルコード
```
var string = OCRAD(image);
alert(string);
```

日本語には対応していないのが、日本人にとっては悲しいところ。

## em-dosbox
URL:
https://github.com/dreamlayers/em-dosbox

em-dosbox とは DOSBox の Emscripten 移植です。DOSBox は、 MS-DOS 環境のエミュレータです。
em-dosbox を使うことにより、MS-DOS 向けのゲームをWeb ブラウザ上に移植できます。
詳細は本書の「em-dosbox で遊んで見る」の項目でまた解説します。

## vim.js
URL:
https://github.com/coolwanglu/vim.js

vim.js は、Vim の Emscripten 移植です。ブラウザ上から Vim を利用できます。


## JSLinux
URL:
https://bellard.org/jslinux/

JSLinuxとはブラウザ上で動くLinuxです。FFmpeg や QEMU の制作者で有名な
Fabrice Bellard が作りました。

## 鬼畜王 on Web
URL:
https://kichikuou.github.io/web/

「鬼畜王 on Web」は、アリスソフトの 配布フリー宣言 に基づいて配布されている「鬼畜王ランス」等のゲームをブラウザ上で遊べるようにしたものです。アリスソフトのゲームエンジン System3 / System3.5（の、有志によるオープンソース版）を Emscripten に移植して利用しています。

## Unity

Unity の WebGL 出力では、Emscripten を利用して Unity エンジン
そのものを JavaScript として出力しています。
以下は、Unity の WebGL 出力で制作されたゲームのサンプルです。

AngryBots
https://files.unity3d.com/jonas/AngryBots/

Dead Trigger 2
https://apps.facebook.com/deadtrigger_ii/

参考
https://blogs.unity3d.com/jp/2014/04/29/on-the-future-of-web-publishing-in-unity/

