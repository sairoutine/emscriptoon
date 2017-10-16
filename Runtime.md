# Emscripten 実行環境について
`Emscripten` による実行環境は、C/C++ アプリケーションと異なる部分があります。

## Input/output

`Emscripten` は、SDL (Simple DirectMedia Layer API) をブラウザ環境用に再実装しています。
このため、キーボードやマウスの入力や、音声や画像の描画を利用したC/C++アプリケーションを動かすことができます。

## ファイルシステム
多くのC/C++コードでは、`libc` ライブラリなどを利用して、同期的にPCのローカルファイルにアクセスできます。
しかし、ブラウザ環境では、PCのローカルファイルに直接アクセスできません。

`Emscripten` では、`libc` を再実装しています。仮想ファイルシステムを通して、
PCのローカルファイルにアクセスするコードを実行できます。
これにより、開発者は必要なファイルを `Emscripten` によるビルド時に指定しておくことで、
ファイルを仮想ファイルシステムに事前に組み込んでおくことができます。

デフォルトの仮想ファイルシステムは `MEMFS` と呼ばれるファイルシステムで、ファイルは全てブラウザのメモリに置かれます。
そのため、ファイルへの変更はブラウザをリロードすると消えてしまいます。

もし、ファイルの変更を保存したい場合、`IDBFS` と呼ばれる仮想ファイルシステムを使う必要があります。
もしコードをブラウザではなく Node.js で利用する場合、`NODEFS` という仮想ファイルシステムを使うことで、
PCのローカルファイルを直接読み書きすることが可能です。

これらの仮想ファイルシステムの種類については、「ファイルシステムについて」の項目でまた説明します。


## ブラウザのメインループ

C/C++ で GUI アプリケーションを実装する際は、無限ループを実装し、その中で描画の実装を行います。
しかし、ブラウザ上ではこれらのコードは、ブラウザの他のイベントをブロックします。
そのため描画が行われず、あたかもブラウザが停止してしまったかのように見えてしまいます。

このため、描画の実装の際のループには、`emscripten_set_main_loop()` を利用します。
このコードはC/C++ では動かないため、`Emscripten` によるコンパイルかどうかでコードを分岐させる必要があります。
分岐には、`#ifdef __EMSCRIPTEN__` が使えます。

以下はサンプルコードです。
```
int main() {
...
#ifdef __EMSCRIPTEN__
  // void emscripten_set_main_loop(em_callback_func func, int fps, int simulate_infinite_loop);
  emscripten_set_main_loop(one_iter, 60, 1);
#else
  while (1) {
    one_iter();
    // Delay to keep frame rate constant (using SDL)
    SDL_Delay(time_to_next_frame());
  }
#endif
}

// The "main loop" function.
void one_iter() {
  // process input
  // render to screen
}
```


## 実行サイクル
`Emscripten` によって生成された js が実行される際、まず preloading フェーズがあります。
ここで、`emcc --preload-file` でプリロードした仮想ファイルシステム用のファイルや、`FS.createPreloadedFile()` 関数で指定したファイルが読み込まれます。

`addRunDependency()` 関数を実行しておくことで、このフェーズにさらに何かを実行することも可能です。

アプリケーションの実行に必要な全ての依存関係が解決できたら、`Emscripten` は`run()` 関数を呼びます。`run()`関数を通して、`main()`が呼ばれます。
`main()` はアプリケーションの実行に必要な初期化を行い、その後 `emscripten_set_main_loop()` を呼ぶコードにすべきです。
これによりアプリケーションのメインループが開始します。

いくつかの方法で、メインループを制御できます。 `emscripten_push_main_loop_blocker()` を実行しておくと、この関数が完了するまで、メインループが開始しません。これは例えばゲームを作る際に、新しいレベルをロードするなど、読み込みのタスクを`emscripten_push_main_loop_blocker()`にて行い、その間、メインループを止めるのに役立ちます。

`emscripten_pause_main_loop()` が呼ばれると、メインループの実行が中断されます。
`emscripten_resume_main_loop()`を呼ぶことで再開できます。


