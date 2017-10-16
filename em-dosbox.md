# em-dosbox で遊んで見る


## em-dosbox とは

em-dosbox とは DOSBox の Emscripten 移植です。DOSBox は、 MS-DOS 環境のエミュレータです。
em-dosbox を使うことにより、MS-DOS 向けのゲームをWeb ブラウザ上に移植できます。

Internet Archive が、2015年ごろに、2400タイトルものMS-DOSのゲームを Web 上で公開しました。
それらの公開されたゲームでは、エミュレータとして em-dosbox が利用されています。

参考URL: https://archive.org/details/softwarelibrary_msdos_games/v2

## Bad Apple!! をブラウザ上で再生してみる。

それでは、em-dosbox を使って、MS-DOS用のソフトウェアをブラウザ上で動かしてみましょう。
ニコニコ動画で有名な動画「Bad Apple!!」は、様々な環境に移植されており、DOS向けの移植が存在します。
今回は、このDOS向けの「Bad Apple!!」をブラウザ上で再生してみることにします。

## 結果

![](./em-dosbox.png)

皆様のブラウザでも、以下のURLから再生できます。

https://sairoutine.github.io/BadAppleJS/

起動すると「Choose sound device」と聞かれるので「3」を押して Enter してください。
ロードに成功すると、「Press enter to start」と言われるので、Enter を押してください。再生が始まります。

※「Exception thrown, see JavaScript console」と表示されたらリロードしてみてください。

## 過程
em-dosbox のコードは以下のリポジトリに存在します。

https://github.com/dreamlayers/em-dosbox

DOS 向け Bad Apple!! は以下のサイトからダウンロードできます。

http://abaduaber.ru/Prog.htm

(ロシア語でわかりづらいですが、「Bad Apple, ДОС - версия」という項目がソレです。)

事前にダウンロードして解凍し、のちのち `git clone` してくる `./em-dosbox/src` 配下に移動させておきます。


コンパイル手順は基本的に em-dosbox リポジトリの Readme に書いてある通りです。

なお、emsdk を利用しています。

```
# emsdk のセット
git clone git@github.com:juj/emsdk.git
cd emsdk-portable
./emsdk install latest
./emsdk activate latest
source ./emsdk_env.sh

# em-dosbox のダウンロード
cd ../
git clone git@github.com:dreamlayers/em-dosbox.git

cd em-dosbox

# configure ファイルの生成
./autogen.sh

# emscripten 用 Makefile の生成
emconfigure ./configure --without-sdl2

# コンパイル
make

# DOS 向け Bad Apple!! の実行ファイルを emscripten 向けにパッケージング
./packager.py badapple badapple BADAPPLE.EXE

# http-server を起動
python -m SimpleHTTPServer 8000

# ブラウザでアクセス
open http://localhost:8000/badapple.html
```

以上です。

## ちょっと解説

```
emconfigure ./configure --without-sdl2
```

em-dosbox はデフォルトで、SDL2 を利用してコンパイルしますが、
SDL1 しか入ってない環境の場合、オプション指定して SDL1 を利用できます
(上記の手順でもSDL1を利用しています)


```
./packager.py badapple badapple BADAPPLE.EXE
```

packager の第一引数にはパッケージ後のファイル名を、第二引数にはパッケージ対象のディレクトリを、第三引数には DOSBox をブラウザで開いた際に最初に起動する実行ファイル名を指定します。

なお、実行ファイルが1つしか存在しない場合、以下のように、第二引数に直接実行ファイル名の指定もできます。

```
./packager.py badapple BADAPPLE.EXE
```
