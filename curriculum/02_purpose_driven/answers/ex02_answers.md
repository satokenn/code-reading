# 解答：演習02 — 機能追加・変更ファイルの特定

> ⚠️ **自分の回答を書いてから開いてください。**

---

## 問1 — 構造の把握

**エントリポイント:**

`package.json` の `"main": "index.js"` → `index.js` → `lib/express.js` へと繋がります。

**ルーティング関連のディレクトリ:**

```
lib/
├── application.js   ← app.get() などのメソッドが定義されている
├── router/
│   ├── index.js     ← Router の実装
│   ├── layer.js     ← 個別ルートのマッチング
│   └── route.js     ← Route オブジェクト
└── express.js       ← エクスポートのハブ
```

---

## 問2 — 変更ファイルの仮説例

```
仮説:
- アプリケーション本体（app.js や server.js）に app.get('/healthcheck', ...) を追加する
- ルーターを使っている場合は該当するルーターファイルを変更する
```

Expressのフレームワーク自体のコードを変更する必要はありません。**フレームワークを使う側のコード**を変更します。

---

## 問3 — 検証とファイルの絞り込み

コマンド実行例と期待される結果：

```bash
$ ls
History.md  LICENSE  Readme.md  index.js  lib/  node_modules/  package.json  test/

$ find . -name "*.js" -not -path "*/node_modules/*" | head -20
./index.js
./lib/express.js
./lib/utils.js
./lib/application.js
./lib/router/index.js
./lib/router/layer.js
./lib/router/route.js
./lib/middleware/json.js
./lib/middleware/query.js
./lib/middleware/init.js
./lib/response.js
./lib/request.js
./lib/view.js
```

`/healthcheck` オトルートは Express のフレームワーク自体に追加するのではなく、アプリケーションコードに追加します。演習のスコープでは **アプリのエントリポイントファイル**（例: `server.js`, `app.js`）が変更対象です。

---

## 問4 — 読み飛ばし記録の例

| 項目 | 内容 |
|---|---|
| 変更が必要なファイル | `server.js`（またはアプリのエントリポイント） |
| 変更箇所の候補 | `app.get()` ルート定義が並んでいる箇所の末尾 |
| **今回読まなかった**ファイル | `lib/router/`, `lib/middleware/`, `lib/request.js`, `lib/response.js`, `test/` ディレクトリ |
| 読まなかった理由 | ルートの追加はフレームワーク内部ではなくユーザーコード側の変更のため |

---

## 問5 — 追加コードの適切な場所

```javascript
// server.js や app.js のルート定義セクション
app.get('/healthcheck', (req, res) => {
  res.json({ status: 'ok', uptime: process.uptime() });
});
```

**なぜこの場所か:**

- `app.use()` で登録するミドルウェアの後、かつ他のルートと整合する位置が適切
- ヘルスチェックは認証不要が多いため、`auth` ミドルウェアより前に置くこともある

---

← 問題に戻る [`ex02_feature_add.md`](../ex02_feature_add.md)
