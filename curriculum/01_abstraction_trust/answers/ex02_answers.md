# 解答：演習02 — `flask` の `@app.route()` デコレータ

> ⚠️ **自分の回答を書いてから開いてください。**

---

## 問1 — 返り値型 `t.Callable[[T_route], T_route]` の意味

`t.Callable[[T_route], T_route]` は「`T_route` 型の関数を受け取り、`T_route` 型の関数を返す関数」つまり**デコレータ**そのものの型です。

```python
# @app.route("/") は以下と等価
decorator = app.route("/")   # Callable[[T_route], T_route] が返る
index = decorator(index)     # index 関数を渡すと index が返る
```

`route()` 自体はルール文字列を受け取り「デコレータを返す関数」として機能します。これを**デコレータファクトリ**と呼びます。返り値の型シグネチャだけでこのパターンが読み取れます。

---

## 問2 — `endpoint: str | None = None` の役割

`endpoint` はURLルールとビュー関数を紐付けるための**識別名**です。デフォルト（`None`）の場合、Flaskは関数名を自動的にエンドポイント名として使用します。

```python
# この2つは等価
@app.route("/")
def index():
    return "Hello"

# 内部的にはこう呼ばれる
app.add_url_rule("/", endpoint="index", view_func=index)
```

`url_for("index")` でURLを逆引きする際に使うエンドポイント名は、`@app.route()` を使う場合は関数名がそのまま使われます。

---

## 問3 — `<int:user_id>` の意味とルートマッチング

`<int:user_id>` の `int` は**コンバーター**と呼ばれ、URLパラメータの型変換とルートマッチングの条件指定を行います。

**`GET /users/42` の場合:**
- `42` は数値 → `int` コンバーターにマッチ
- `get_user(user_id=42)` が呼ばれる（`int` 型として渡される）
- `get_user_by_name` はマッチしない

**`GET /users/alice` の場合:**
- `alice` は非数値 → `int` コンバーターにマッチしない
- `get_user_by_name(username="alice")` が呼ばれる

docstringの「`<converter:variable_name>`」という記述から、コンバーターが型変換と条件分岐の両機能を持つことが推定できます。

---

## 問4 — `**options` が意図する拡張性

`**options` を `add_url_rule()` に渡す設計は、将来的な拡張（新しいWerkzeugのRule引数）に対してFlask本体を変更せずに対応できるようにするためです。

推定される主なオプション：

| オプション | 用途 |
|---|---|
| `methods` | 許可するHTTPメソッド（`GET`, `POST`, `PUT` など） |
| `strict_slashes` | 末尾スラッシュの有無を厳密に扱うか |
| `host` | 特定のホスト名にのみマッチさせる（サブドメインルーティング） |

`**options` という設計は「現在知られていない将来のオプションも受け入れる」という開放性を示します。これはインターフェース設計における重要なパターンです。

---

← 問題に戻る [`ex02_module_contract.md`](../ex02_module_contract.md)
