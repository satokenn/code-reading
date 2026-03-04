# 解答：演習03 — 実装を読まずに全体像を描く

> ⚠️ **自分の回答を書いてから開いてください。**

---

## 問1 — `POST /users` の処理フロー

```
[HTTP リクエスト受信: POST /users]
        ↓
[1. app.use() で登録されたミドルウェアを順番に実行]
   （ロギング、ボディパーサー、認証など）
   各ミドルウェアは next() を呼んで次へ渡す
        ↓
[2. パスとHTTPメソッドのマッチング]
   app.post('/users', ...) にマッチするかチェック
        ↓
[3. マッチしたルートのハンドラを実行]
   handler(req, res, next) が呼ばれる
        ↓
[4. （エラーの場合）next(err) が呼ばれる]
   エラーハンドリングミドルウェアへ転送
   ※エラーなければスキップ
        ↓
[5. res.json() / res.send() でレスポンスを送信]
   HTTPレスポンスが生成されクライアントへ
```

この図は `express/lib/application.js` や `express/lib/router/index.js` を一切読まずに、シグネチャとREADMEだけで描けます。

---

## 問2 — 公開API一覧の探索

`package.json` の `"main": "index.js"` から `index.js` を開くと：

```javascript
module.exports = require('./lib/express');
```

さらに `lib/express.js` のエクスポート部分のみ確認：

```javascript
exports = module.exports = createApplication;
exports.application = proto;
exports.request = req;
exports.response = res;
exports.Route = Route;
exports.Router = Router;
exports.json = bodyParser.json;
exports.static = serveStatic;
// ...
```

外部に公開されているAPIとして確認できるもの：
- `express()` / `createApplication`（アプリ生成関数）
- `express.Router`（独立したルーター）
- `express.json()`（ボディパーサー）
- `express.static()`（静的ファイル配信）

**実装コードは一行も読まずに、エクスポート宣言だけでAPIサーフェスが把握できます。**

---

## 問3 — 抽象化の限界（例）

典型的な「限界のポイント」と対処法の例：

| 不明だった点 | 限界の内容 | 読むべき箇所の仮説 |
|---|---|---|
| エラーハンドラの識別方法 | `next(err)` を渡せばいいとはわかるが識別方法が不明 | `lib/router/layer.js` の `handle_error` 周辺 |
| ミドルウェアの内部データ構造 | 「配列に追加する」程度しか推測できない | `lib/router/index.js` の `stack` プロパティ |
| パスマッチングのアルゴリズム | 文字列比較以上のことが起きているはず | `lib/router/layer.js` の `match()` |

この「限界の言語化」が Chapter 03（抽象の漏れへの対応）の演習につながります。

---

← 問題に戻る [`ex03_trust_test.md`](../ex03_trust_test.md)  
次のChapter → [`../02_purpose_driven/`](../../02_purpose_driven/)
