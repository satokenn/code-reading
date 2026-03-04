# 演習02：モジュールの契約を読む — `flask` の `@app.route()`

## 🎯 この演習が鍛える能力

**能力Ⅰ：抽象化の信頼**  
デコレータのシグネチャとドキュメントから、内部実装を読まずに仕様を推定する。

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

そして、実際の使用例：

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

### 問1（ウォームアップ）
> ❓ `route()` の返り値型 `t.Callable[[T_route], T_route]` は何を意味しますか？デコレータとして機能するためにこの型が必要な理由を説明してください。

<details>
<summary>💡 ヒント</summary>

Python のデコレータは `@decorator` が `func = decorator(func)` の糖衣構文です。`route("/")` の返り値は何を受け取って何を返す必要があるでしょうか？

</details>

<details>
<summary>✅ 解答例</summary>

`t.Callable[[T_route], T_route]` は「`T_route` 型の関数を受け取り、`T_route` 型の関数を返す関数」つまり**デコレータ**そのものの型です。

```python
# @app.route("/") は以下と等価
decorator = app.route("/")   # Callable[[T_route], T_route] が返る
index = decorator(index)     # index 関数を渡すと index が返る
```

`route()` 自体はルール文字列を受け取り「デコレータを返す関数」として機能します。これを**デコレータファクトリ**と呼びます。

内部実装を読まなくても、返り値の型シグネチャだけでこのパターンが読み取れます。

</details>

---

### 問2（ウォームアップ）
> ❓ `add_url_rule()` の `endpoint: str | None = None` 引数の役割を推定してください。`@app.route()` を使う場合と `add_url_rule()` を直接使う場合の違いは何だと思いますか？

<details>
<summary>💡 ヒント</summary>

`app.route()` のdocstringに「Calls `add_url_rule`」と書いてあります。`endpoint` が `None` の場合、Flaskは何を使ってエンドポイントを識別すると思いますか？

</details>

<details>
<summary>✅ 解答例</summary>

`endpoint` はURLルールとビュー関数を紐付けるための**識別名**です。デフォルト（`None`）の場合、Flaskは関数名を自動的にエンドポイント名として使用します。

```python
# この2つは等価
@app.route("/")
def index():
    return "Hello"

# ↓ 内部的にはこう呼ばれる
app.add_url_rule("/", endpoint="index", view_func=index)
```

`url_for("index")` のようにエンドポイント名でURLを逆引きできますが、`@app.route()` を使う場合は関数名がそのまま使われます。これもシグネチャだけから推論できます。

</details>

---

### 問3（フロー追跡）
> ❓ 以下のコードを見て、`GET /users/42` というリクエストが来たとき、どの関数が呼ばれ、引数にどんな値が渡されるかを推理してください。`<int:user_id>` という記法の意味も、実装を読まずに推定してください。

```python
@app.route("/users/<int:user_id>")
def get_user(user_id):
    return f"User {user_id}"

@app.route("/users/<string:username>")
def get_user_by_name(username):
    return f"User {username}"
```

<details>
<summary>💡 ヒント</summary>

- `<int:user_id>` の `int` はどんな役割を持つと思いますか？
- `GET /users/42` と `GET /users/alice` では、どちらのルートが使われるでしょうか？
- `<converter:variable_name>` という記法はdocstringに言及されています。

</details>

<details>
<summary>✅ 解答例</summary>

`<int:user_id>` の `int` は**コンバーター**と呼ばれ、URLパラメータの型変換とルートマッチングの条件指定を行います。

**`GET /users/42` の場合:**
- `42` は数値 → `int` コンバーターにマッチ
- `get_user(user_id=42)` が呼ばれる（`int` 型として渡される）
- `get_user_by_name` はマッチしない（`string` コンバーターは数値のみのパスと競合）

**`GET /users/alice` の場合:**
- `alice` は非数値 → `int` コンバーターにマッチしない
- `get_user_by_name(username="alice")` が呼ばれる

Werkzeug のルーティング構文という記述とdocstringの例から、コンバーターが型変換と条件分岐の両機能を持つことが推定できます。

</details>

---

### 問4（設計意図）
> ❓ `route()` が `**options` で任意の追加オプションを受け取る設計になっています。これはどのような拡張性を意図した設計ですか？具体的に3種類のオプションを推定し、その用途を説明してください。

<details>
<summary>💡 ヒント</summary>

使用例で `methods=["GET", "POST"]` が `**options` として渡されていることに注目してください。他にどんな設定が必要になりそうですか？「URLのルールとして制御したいこと」を考えてみてください。

</details>

<details>
<summary>✅ 解答例</summary>

`**options` を `add_url_rule()` に渡す設計は、将来的な拡張（新しいWerkzeugのRule引数）に対してFlask本体を変更せずに対応できるようにするためです。

推定される主なオプション：

| オプション | 用途 |
|---|---|
| `methods` | 許可するHTTPメソッド（`GET`, `POST`, `PUT` など） |
| `strict_slashes` | 末尾スラッシュの有無を厳密に扱うか（`/users` と `/users/` を区別するか） |
| `host` | 特定のホスト名にのみマッチさせる（サブドメインルーティング） |

`**options` という設計は「現在知られていない将来のオプションも受け入れる」という開放性を示しており、これはインターフェース設計の重要なパターンです。

</details>

---

## 📝 振り返り

```
① 「読まなかった」判断は正しかったか？

② 推定したことで実際と違った点や、新たに気づいたことはあるか？

③ 次回どう変えるか？
```

---

> 🔗 次の演習 [ex03_trust_test.md](./ex03_trust_test.md) では、実際にOSSを自分で探索し、インターフェースだけで全体図を描く総合演習を行います。
