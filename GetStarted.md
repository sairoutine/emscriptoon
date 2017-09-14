# Get Started

それでは、Emscripten を実際に触ってみましょう。

## ダウンロードとインストール

### Emscripten SDK

Emscripten を使うには、portable SDK を利用するのが便利です。
SDK には、Emscripten のコンパイルに必要な一通りのツールや依存しているライブラリが含まれているます。

SDK は以下のページからダウンロードできます。

```
TODO: URL
```

また、Mac であれば homebrew を利用してダウンロードすることもできます。

```
TODO: 手順
```

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


