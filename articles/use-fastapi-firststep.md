---
title: "FastAPIを使ってみる" # 記事のタイトル
emoji: "💨" # アイキャッチとして使われる絵文字（1文字だけ）
type: "tech" # tech: 技術記事 / idea: アイデア記事
topics: ["python","FastAPI","Swagger"] # タグ。["markdown", "rust", "aws"]のように指定する
published: false # 公開設定（falseにすると下書き）
---

# FastAPIを使ってみる

今回プロジェクト内でFastAPIを使うことになったので、導入をする際の自分用メモ。

FastAPIの公式ドキュメントを参考にしてます↓
https://fastapi.tiangolo.com/ja/

# 環境

+ Python 3.6
+ macOS Big Sur ver 11.1

# インストール方法

```bash:terminal
$ pip3 install fastapi
```

uvicornを使用します、ちなみにuvicornは「Uvicorn is a lightning-fast ASGI server implementation, using uvloop and httptools.」とあることから、unicorn(ユニコーン)をもじってuvloopのuvを使ったのでしょうか。
ASGIとは、Web Server Gateway Interfaceの略で、WebサーバとWebアプリケーションを接続するための、標準化されたインタフェースの定義のことです。

```bash:terminal
$ pip3 install uvicorn
```

# hello worldを表示してみよう

まず、好きなエディターでmain.pyを作成します。

```python:main.py
from fastapi import FastAPI

app = FastAPI()


@app.get("/")
def read_root():
    return {"Hello": "World"}


@app.get("/items/{item_id}")
def read_item(item_id: int, q: str = None):
    return {"item_id": item_id, "q": q}
```

# 実行


```
$ uvicorn main:app --reload
```


