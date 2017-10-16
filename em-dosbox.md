# はじめに

[f:id:sairoutine:20170731020004p:plain]

Emscriptenとは C/C++言語からLLVMを生成し、それをJavaScriptに変換するコンパイラのことです。  
DOSBoxは、PC/AT互換機のMS-DOS環境を再現するエミュレータです。  

# 結果

[https://twitter.com/sairoutine/status/891677841976303617:embed]

上記ツイート内動画のように、DOS向け Bad Apple!! をブラウザで再生することができました。  


# 再生
皆様のブラウザでも、以下のURLから再生することができます。  
  
https://sairoutine.github.io/BadAppleJS/

起動すると「Choose sound device」と聞かれるので「3」を押して Enter してください。ロードに成功すると、「Press enter to start」と言われるので、Enter を押してください。再生が始まります。

※「Exception thrown, see JavaScript console」と表示されたらリロードしてみてください  
※音ズレが発生しているのは勘弁  

# 過程
em-dosbox という DOSBox の emscripten 移植が存在するので、それを利用しています。

https://github.com/dreamlayers/em-dosbox

DOS 向け Bad Apple!! は以下のサイトからダウンロードできます。

http://abaduaber.ru/Prog.htm

(ロシア語でわかりづらいですが、「Bad Apple, ДОС - версия」という項目がソレです。)

事前にダウンロードして解凍し、のちのち git clone してくる `./em-dosbox/src` 配下に移動させておきます。


コンパイル手順は基本的に em-dosbox レポジトリの Readme に書いてある通りです。環境は、MacOS X EI Captain 10.11.6 です。

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

# ちょっと解説

```
emconfigure ./configure --without-sdl2
```

em-dosbox はデフォルトで、SDL2 を利用してコンパイルするが、
私のMac には SDL1 しか入ってなかったので、オプション指定して SDL1 を利用しています。

SDL2 が入っていないと、以下のようなエラーになる。

[f:id:sairoutine:20170731020014p:plain]


```
./packager.py badapple badapple BADAPPLE.EXE
```

packager の第一引数にはパッケージ後のファイル名を、第二引数にはパッケージ対象のディレクトリを、第三引数には、DOSBox をブラウザで開いた際に、最初に起動する実行ファイル名を指定します。

なお、実行ファイルが1つしか存在しない場合、以下のように、第二引数に直接実行ファイル名の指定もできます。

```
./packager.py badapple BADAPPLE.EXE
```

# 終わり
描画が追いついておらず、音ズレが発生しているのは、WebAssembly を有効にすれば
解決するかもしれない。次回やってみる。

Bad Apple !!だけでなく、DOSBox では例えば Win3.1 や Windows 98 も起動できるみたいなので、うまくやれば、ブラウザ上でそれらも動かせるかもしれない。
