---
title: "FastAPIを使ってWebアプリを作ってみる:ver1" # 記事のタイトル
emoji: "💨" # アイキャッチとして使われる絵文字（1文字だけ）
type: "tech" # tech: 技術記事 / idea: アイデア記事
topics: ["python","FastAPI","Swagger"] # タグ。["markdown", "rust", "aws"]のように指定する
published: false # 公開設定（falseにすると下書き）
---

# FastAPIを使ってWebアプリを作ってみる:ver1

FastAPIを使うことになったので、導入をする際の自分用メモ。
基本的にはFastAPIの公式ドキュメントを参考にしてます[^1]。

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

# hello world!を表示してみよう

お決まりのhello world!を表示してみたいと思います。
まず、好きなエディターでmain.pyを作成します。

```python:main.py
from fastapi import FastAPI

app = FastAPI()


@app.get("/")
def read_root():
    return {"Hello": "World !"}
```

# 実行

```bash:terminal
$ uvicorn main:app --reload
```

この--reloadというのは、ファイル更新等をした際にも再読み込みしてね、と明示するための引数である。


テスト投稿


[^1]: FastAPI https://fastapi.tiangolo.com/ja/
[^2]: Web Server Gateway Interface https://ja.wikipedia.org/wiki/Web_Server_Gateway_Interface
[^3]: 【第1回】FastAPIチュートリアル: ToDoアプリを作ってみよう【環境構築編】https://rightcode.co.jp/blog/information-technology/fastapi-tutorial-todo-apps-environment




