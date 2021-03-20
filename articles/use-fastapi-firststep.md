---
title: "FastAPIを使ってWebアプリを作ってみる:ver1" # 記事のタイトル
emoji: "💨" # アイキャッチとして使われる絵文字（1文字だけ）
type: "tech" # tech: 技術記事 / idea: アイデア記事
topics: ["python","FastAPI","Swagger"] # タグ。["markdown", "rust", "aws"]のように指定する
published: true # 公開設定（falseにすると下書き）
---

# FastAPIを使ってWebアプリを作ってみる:ver1

FastAPIを使うことになったので、導入をする際の自分用メモ。
最終的には、掲示板アプリを作るところまでやってみたいと思っています。

今回は、基本的にはFastAPIの公式ドキュメントを参考にしてます[^1]。

# FastAPIについて

とにかく名前の通り、速いということが特徴でしょうか。レスポンス面でもコーディング面でも速さを重視しているフレームワークであるといえそうです。
FastAPI は、高速にレスポンス可能な WebAPI を構築することに優れています[^3]。
また、公式ドキュメントには、高速なコーディング: 開発速度を約 200%~300%向上させます[^1]。とあり、大変魅力的なフレームワークであると思います。

# 環境

+ Python 3.6
+ macOS Big Sur ver 11.1

# インストール方法

```bash:terminal
$ pip3 install fastapi
```

今回は、uvicornを使用します、ちなみにuvicornは「Uvicorn is a lightning-fast ASGI server implementation, using uvloop and httptools.」[^1]とあることから、unicorn(ユニコーン)をもじってuvloopのuvを使ったのでしょうか。
※この辺の由来はちょっとググっても出てこなかったです...どなたかご存知の方おられたらご教示ください。

ちなみに、ASGIとは、Web Server Gateway Interfaceの略で、WebサーバとWebアプリケーションを接続するための、標準化されたインタフェースの定義のことです。[^2]

```bash:terminal
$ pip3 install uvicorn
```

# Hello World !を表示してみよう

お決まりのHello World !を表示してみたいと思います。
まず、好きなエディターでmain.pyを作成します。

```python:main.py
from fastapi import FastAPI
from fastapi.responses import HTMLResponse

app = FastAPI()

@app.get("/",response_class=HTMLResponse)
def read_root():
    return """
        <html>
            <head>
                <title>tutorial</title>
            </head>
            <body>
                <h1>Hello World !</h1>
            </body>
        </html>
        """
```

まず、fastapiライブラリからFastAPIをインポートします。
また、HTMLをレンダーしたいと思うので、fastapi.resposesライブラリからHTMLResposeをインポートします。

ルーティングの書き方はとてもシンプルで、@appよりメソッドチェーンを通してgetリクエストを受け付けるインスタンスを生成し、パスを記述しルートのリクエストを受け付けます。
直下にあるサブルーチンで処理を書きます。今回は「Hello World !」を表示するのみです。

# 実行

```bash:terminal
$ uvicorn main:app --reload
```

この--reloadというのは、ファイル更新等をした際にも再読み込みしてね、と明示するための引数である。

以下は実行した様子↓
![](https://storage.googleapis.com/zenn-user-upload/75laqcay25o0tu9moqy796btywyf)
![](https://storage.googleapis.com/zenn-user-upload/d7vs452ydic88mkeijkndqxgd4xl)

以上でhello World !の表示ができるところまでできた。
次回は、getリクエスト以外のリクエストも書いてみます。

[^1]: FastAPI https://fastapi.tiangolo.com/ja/
[^2]: Web Server Gateway Interface https://ja.wikipedia.org/wiki/Web_Server_Gateway_Interface
[^3]: 【第1回】FastAPIチュートリアル: ToDoアプリを作ってみよう【環境構築編】https://rightcode.co.jp/blog/information-technology/fastapi-tutorial-todo-apps-environment




