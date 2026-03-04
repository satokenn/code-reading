# 演習02：モジュールの契約を読む — `flask` の `@app.route()`

## 🎯 この演習が鍛える能力

**能力Ⅰ：抽象化の信頼**  
デコレータのシグネチャとドキュメントから、内部実装を読まずに仕様を推定する。

> ⚠️ **解答は [`answers/ex02_answers.md`](./answers/ex02_answers.md) に分離されています。**  
> 問題をすべて解き終えてから開いてください。

---

## 📋 目的宣言（演習前に記入）

```
🎯 目的：
📍 仮説（どこを読めば分かると思うか）：
🚫 除外（今回は読まない領域）：
```

---

## 📚 背景知識

Flask は Python の軽量Webフレームワークです。`@app.route()` はルーティングを定義するデコレータです。

以下は Flask の `route()` のシグネチャ（ソースコードの `app.py` より抜粋）です。**実装は見ません。**

```python
def route(
    self,
    rule: str,
    **options: t.Any
) -> t.Callable[[T_route], T_route]:
    """Decorate a view function to register it with the given URL
    rule and options. Calls :meth:`add_url_rule`, which has more
    details about the implementation.

    .. code-block:: python

        @app.route("/")
        def index():
            return "Hello, World!"

    See :ref:`url-route-registrations` for full details, including
    how to handle trailing slashes.

    The URL rule string uses Werkzeug's routing syntax. In general,
    variable parts are marked as ``<variable_name>``. If a converter
    is given, it looks like ``<converter(args):variable_name>``.

    :param rule: The URL rule string.
    :param options: Extra options passed to the
        :class:`~werkzeug.routing.Rule` object.
    """
```

また、`add_url_rule()` のシグネチャも確認します（`@app.route()` が内部で呼ぶ関数）：

```python
def add_url_rule(
    self,
    rule: str,
    endpoint: str | None = None,
    view_func: ft.RouteCallable | None = None,
    provide_automatic_options: bool | None = None,
    **options: t.Any,
) -> None:
```

実際の使用例：

```python
from flask import Flask, request
app = Flask(__name__)

# 基本的なルート
@app.route("/")
def index():
    return "Hello"

# パス変数あり
@app.route("/users/<int:user_id>")
def get_user(user_id):
    return f"User {user_id}"

# HTTPメソッド指定
@app.route("/users", methods=["GET", "POST"])
def users():
    if request.method == "POST":
        return "Created", 201
    return "List"
```

---

## 🔍 演習問題

### 問1
`route()` の返り値型 `t.Callable[[T_route], T_route]` は何を意味しますか？デコレータとして機能するためにこの型が必要な理由を説明してください。

---

### 問2
`add_url_rule()` の `endpoint: str | None = None` 引数の役割を推定してください。`@app.route()` を使う場合と `add_url_rule()` を直接使う場合の違いは何だと思いますか？

---

### 問3
以下のコードを見て、`GET /users/42` というリクエストが来たとき、どの関数が呼ばれ、引数にどんな値が渡されるかを推理してください。`<int:user_id>` という記法の意味も、実装を読まずに推定してください。

```python
@app.route("/users/<int:user_id>")
def get_user(user_id):
    return f"User {user_id}"

@app.route("/users/<string:username>")
def get_user_by_name(username):
    return f"User {username}"
```

---

### 問4
`route()` が `**options` で任意の追加オプションを受け取る設計になっています。これはどのような拡張性を意図した設計ですか？具体的に3種類のオプションを推定し、その用途を説明してください。

---

## 📝 振り返り

```
① 「読まなかった」判断は正しかったか？

② 推定したことで実際と違った点や、新たに気づいたことはあるか？

③ 次回どう変えるか？
```

---

解答確認 → [`answers/ex02_answers.md`](./answers/ex02_answers.md)  
次の演習 → [`ex03_trust_test.md`](./ex03_trust_test.md)
