# Get Started

それでは、Emscripten を実際に触ってみましょう。

## ダウンロードとインストール

### Emscripten SDK

Emscripten を使うには、portable SDK を利用するのが便利です。
SDK には、Emscripten のコンパイルに必要な一通りのツールや依存しているライブラリが含まれているます。

SDK は以下のページからダウンロードできます。

https://kripken.github.io/emscripten-site/docs/getting_started/downloads.html#sdk-downloads

SDK をダウンロードしたら、SDK のディレクトリに移動し、Emscripten ツールをダウンロードします。

```
# 最新の Emscripten ツール情報を取得して更新
./emsdk update
```

```
# 最新の Emscripten ツールをダウンロード
./emsdk install latest
```

```
# 最新の Emscripten ツールを有効化
./emsdk activate latest
```


Linux and Mac OS X では、以下のコマンドも実行します。
```
# 有効化したツールのパスを通す環境変数を設定
source ./emsdk_env.sh
```

## Emscriptenでコンパイルしてみる

Emscripten でコンパイルを行うには、`emcc` コマンドを使います。
まず、Emscripten のダウンロードとインストールが正常に行われているか確認します。

```
./emcc -v
```

それでは、いよいよC/C++ のコードを JavaScript に変換してみましょう。
以下のC/C++ コードを用意します。

hello_world.c
```
#include <stdio.h>

int main() {
  printf("hello, world!\n");
  return 0;
}
```

見ての通り、hello, world! を出力するだけのコードです。これを Emscripten でコンパイルします。

```
./emcc tests/hello_world.c
```

上記のコマンドを実行すると、`a.out.js` が生成されます。
このファイルを Node.js で実行することで、hello, world! の文字列が出力されます。

```
node a.out.js # hello, world!
```

ブラウザで実行したい場合は、-o オプションを利用します。-o は、出力ファイル名を指定するオプションです。
`.html` 拡張子のファイルを指定すると、Emscrpten は HTML 形式でコンパイルします。

```
./emcc tests/hello_world.c -o hello.html
```

生成された hello.html を開くことで、hello world が出力されます。

HTML 出力は、テキストの表示にとどまりません。
SDL API を利用したコードを使えば、HTML5 の canvas 要素で出力されます。

以下のコードは、SDL API を利用した C/C++ コードです。

hello_world_sdl.cpp
```
#include <stdio.h>
#include <SDL/SDL.h>

#ifdef __EMSCRIPTEN__
#include <emscripten.h>
#endif

extern "C" int main(int argc, char** argv) {
  printf("hello, world!\n");

  SDL_Init(SDL_INIT_VIDEO);
  SDL_Surface *screen = SDL_SetVideoMode(256, 256, 32, SDL_SWSURFACE);

#ifdef TEST_SDL_LOCK_OPTS
  EM_ASM("SDL.defaults.copyOnLock = false; SDL.defaults.discardOnLock = true; SDL.defaults.opaqueFrontBuffer = false;");
#endif

  if (SDL_MUSTLOCK(screen)) SDL_LockSurface(screen);
  for (int i = 0; i < 256; i++) {
    for (int j = 0; j < 256; j++) {
#ifdef TEST_SDL_LOCK_OPTS
      // Alpha behaves like in the browser, so write proper opaque pixels.
      int alpha = 255;
#else
      // To emulate native behavior with blitting to screen, alpha component is ignored. Test that it is so by outputting
      // data (and testing that it does get discarded)
      int alpha = (i+j) % 255;
#endif
      *((Uint32*)screen->pixels + i * 256 + j) = SDL_MapRGBA(screen->format, i, j, 255-i, alpha);
    }
  }
  if (SDL_MUSTLOCK(screen)) SDL_UnlockSurface(screen);
  SDL_Flip(screen); 

  printf("you should see a smoothly-colored square - no sharp lines but the square borders!\n");
  printf("and here is some text that should be HTML-friendly: amp: |&| double-quote: |\"| quote: |'| less-than, greater-than, html-like tags: |<cheez></cheez>|\nanother line.\n");

  SDL_Quit();

  return 0;
}
```

```
./emcc tests/hello_world_sdl.cpp -o hello.html
```

コンパイルすることで、HTML5 canvas 形式で、画面が表示されたことかと思います。

C/C++ コードでは、ローカルのファイルを読み込んだり、書き込んだりすることがあります。
しかし JavaScript では、ローカルのファイルの読み書きは(ユーザーの許可がない限り)できません。

このため Emscripten は独自の仮想ファイルシステムをシミュレートし、
仮想ファイルシステム内で、C/C++ コード内に記述されていたファイルの読み書きを JavaScript で行います。

以下のコードは、C/C++ 内でファイルの読み書きを行っているコードです。


The hello_world_file.cpp
```
#include <stdio.h>
int main() {
  FILE *file = fopen("tests/hello_world_file.txt", "rb");
  if (!file) {
    printf("cannot open file\n");
    return 1;
  }
  while (!feof(file)) {
    char c = fgetc(file);
    if (c != EOF) {
      putchar(c);
    }
  }
  fclose (file);
  return 0;
}
```

tests/hello_world_file.txt
```
==
This data has been read from a file.
The file is readable as if it were at the same location in the filesystem, including directories, as in the local filesystem where you compiled the source.
==
```

hello_world_file.txt を読み込み、それを標準出力するコードです。

Emscripten でコンパイルする際に、--preload-file オプションで、仮想ファイルシステムに組み込むファイルを指定します。
これにより C/C++から変換された JavaScript 内で、tests/hello_world_file.txt に、ファイル書き込みや読み込みを行うことができます。

```
./emcc tests/hello_world_file.cpp -o hello.html --preload-file tests/hello_world_file.txt
```

Note

Emscripten によって変換された JavaScriptコードから --preload-file で指定されたファイルにアクセスする際は、
XHR を利用してアクセスされます。よって Chrome や Internet Explorer から、直接ローカルの HTML を開くと XHR でアクセスできません。

その場合は、`python -m SimpleHTTPServer 8080` のように ローカルに HTTP サーバーを立てて、
アクセスしてください。
