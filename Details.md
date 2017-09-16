# C/C++コードの移植に当たっての気をつけポイント

Emscpriten を利用することで、C/C++ コードを JavaScript に移植できます。

しかし、一部移植不可能なコードや、移植可能だが実行速度が極端に遅いコードが存在します。
それらのコードは別のコードに書き換える必要があります。

## コンパイルできないコード

以下のようなコードは Emscpriten を使って移植しても動かないため、C/C++ 側のコードを書き換える必要があります。

 - マルチスレッド機能を使っており、かつスレッド間でメモリを共有している

JavaScript には、Web Workers を使ったマルチスレッド機能があります。しかし Web Workers 間でのメモリ共有ができないため、
Emscripten ではスレッド間でメモリを共有するコードを移植できません。

 - ビッグエンディアンに依存したコード

JavaScript の TypedArray (Uint8Array 等) は実行環境のコンピュータのバイトオーダーに従います。
ほとんどのコンピュータのバイトオーダーはリトルエンディアンであるため、ビッグエンディアンに依存したコードは動きません。

 - x86 のメモリアラインメントの挙動に依存したコード

x86 (私達のPCでよく使われているCPU の種類) では、アラインメントされていないメモリの読み書きが可能です。しかし、ARM などの他のCPUアーキテクチャでは、それらは不可能です。そのため、Emscripten によって生成されたコードでは、これらの挙動に依存したコードの挙動は未定義になります。SAFE_HEAP=1 オプションを Emscripten でのコンパイル時に使用することで、これらの挙動をエラーにできます(デバッグ時に便利)。

 - 一部の低レベルな機能を使ったコード

例えば、setjmp や longjmp を利用したコールスタックを操作するコードです。

 - レジスタの内容やコールスタックを読みとるコード

レジスタの内容やコールスタックは、Emscripten が生成した JavaScript ではローカル変数となるため、
コードから読み取ることができません。

 - asm() などでアセンブリを利用したコード

Emscripten を利用したコードでは、アセンブリやCPUのエミュレートを行うわけではないので、アセンブリを利用したコードは使用できません。

## コンパイルできるがパフォーマンスが極端に悪いコード

 - 64bit int の変数

 JavaScript には 64bit int 型がありません。そのため Emscripten によって変換される際に、64bit int 型をエミュレートするコードに変換されます。しかし、あくまでエミュレートなので、四則演算(+, -, *, /) がとても遅くなります。

 - C++ の例外

 JavaScript エンジンのコード最適化が無効になるため、遅くなります。

## API Limitations
http://localhost:2000/docs/porting/guidelines/api_limitations.html

## Function Pointer Issues
http://localhost:2000/docs/porting/guidelines/function_pointer_issues.html

## Specific Browser Limitations
http://localhost:2000/docs/porting/guidelines/browser_limitations.html

