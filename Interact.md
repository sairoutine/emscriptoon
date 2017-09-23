# C/C++とJavaScript間でのコード呼び出し

Emscripten 実行環境では、C/C++ コードに直接 JavaScript のコードを書けます。
あるいは、コンパイルされたC/C++の関数を JavaScript から呼び出したりできます。

## ccall/cwrap
ccall() や cwrap() を利用することで、JavaScript からコンパイルされたCの関数を利用できます。

ccall() は、コンパイルされた C/C++ の関数を引数を指定して呼び出し、そして返り値を返します。cwrap() は、コンパイルされた C/C++の関数を JavaScript の関数にして返します。

それでは、ccall() と cwrap() を試してみます。

tests/hello_function.cpp
```
#include <math.h>

extern "C" {

int int_sqrt(int x) {
  return sqrt(x);
}

}
```

以下のコマンドでコンパイルします。
```
./emcc tests/hello_function.cpp -o function.html -s EXPORTED_FUNCTIONS="['_int_sqrt']"
```

```
TODO: `EXPORTED_FUNCTIONS` とは？
`_` がついていることに注意。
```

cwrap() を使うことで JavaScript の関数として利用できます。

```
int_sqrt = Module.cwrap('int_sqrt', 'number', ['number'])
int_sqrt(13)
int_sqrt(28)
```

cwrap の第一引数は、JavaScript で利用したいC/C++での関数名です。
第二引数は、返り値の型です。第三引数は、引数の型の配列です。

型には`number` `string` `array` が使用できます。これはそれぞれC/C++ における `integer`, `char*`, Cの配列と同様です。


ccall() を使うことで、JavaScript の関数として呼び出すことができます。

```
// result is 5
var result = Module.ccall('int_sqrt', // name of C function
  'number', // return type
  ['number'], // argument types
  [28]); // arguments

```

<!--
var em_module = require('./api_example.js');

em_module._sayHi(); // direct calling works

To call the method directly, you will need to use the full name as it appears in the generated code. This will be the same as the original C function, but with a leading _.
-->



## C/C++ のコード内でJavaScript を記述する
`emscripten_run_script()` を使うことで、C/C++コード内にJavaScript を記述できます。

```
emscripten_run_script("alert('hi')");
```

または`EM_ASM()` マクロを使うことでも記述できます。
```
#include <emscripten.h>

int main() {
  EM_ASM(
    alert('hello world!');
    throw 'all done';
  );
  return 0;
}
```


`EM_ASM_` を利用することで、C/C++コード側の値を JavaScript に渡すこともできます。
($0, $1 ... が引数になります)
```
// This will show I received: 100.
EM_ASM_({
  Module.print('I received: ' + $0);
}, 100);
```
EM_ASM_INT あるいは EM_ASM_DOUBLE を使うことで、返り値を得ることもできます。

```
int x = EM_ASM_INT({
  Module.print('I received: ' + $0);
  return $0 + 1;
}, 100);
printf("%d\n", x);
```

