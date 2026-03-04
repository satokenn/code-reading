# 演習03：抽象化の信頼テスト — 実装を読まずに全体像を描く

## 🎯 この演習が鍛える能力

**能力Ⅰ：抽象化の信頼（総合）**  
実際のOSSリポジトリに対して、シグネチャ・README・docstringのみを手がかりに、システム全体の処理フロー図を描く。

> ⚠️ この演習では「正確な図」よりも「実装を読まずに推論できること」が評価されます。

---

## 📋 目的宣言（演習前に記入）

```
🎯 目的：
📍 仮説（どこを読めば分かると思うか）：
🚫 除外（今回は読まない領域）：
```

---

## 📚 背景知識

### Express のリクエスト処理の全体像（手がかり資料）

以下の情報のみを使って演習を進めてください。

**公開API一覧（シグネチャのみ）:**

```typescript
// アプリケーション
const app = express();

// ミドルウェア登録
app.use(path?, ...handlers): Application
app.get(path, ...handlers): Application
app.post(path, ...handlers): Application
// ... (他のHTTPメソッドも同様)

// リクエストオブジェクト（読み取り専用な主要プロパティ）
req.method: string          // "GET", "POST" など
req.path: string            // "/users/42" など
req.params: object          // { user_id: "42" } など
req.query: object           // { page: "1" } など
req.body: any               // リクエストボディ（パース済み）
req.headers: object         // HTTPヘッダー

// レスポンスオブジェクト（主要メソッド）
res.status(code): Response
res.json(body): void
res.send(body): void
res.redirect(url): void
res.end(): void

// NextFunction
next(err?: any): void
```

**Express の README から抜粋:**

> Express is a minimal and flexible Node.js web application framework that provides a robust set of features for web and mobile applications.
>
> The `app` object has methods for routing HTTP requests; configuring middleware; rendering HTML views; registering a template engine; and modifying application settings that control how the application behaves.

---

## 🔍 演習問題

### 問1（フロー追跡）

> ❓ 上記の情報だけを使い、`POST /users` というHTTPリクエストが Express アプリに届いてから `res.json()` でレスポンスが返るまでの処理フローを、以下のテンプレートに書き起こしてください。実装ファイルは開かないでください。

**フロー図テンプレート（このまま使ってください）:**

```
[HTTP リクエスト受信]
        ↓
[1. ________________]
        ↓
[2. ________________]
        ↓
[3. ________________]
        ↓
        ...
        ↓
[N. res.json() でレスポンス送信]
```

<details>
<summary>💡 ヒント</summary>

- `app.use()` で登録されたミドルウェアはいつ実行されるか？
- `app.post('/users', handler)` はいつマッチするか？
- `next()` が呼ばれない場合とエラーが渡された場合（`next(err)`）の違いは？

</details>

<details>
<summary>✅ 解答例</summary>

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

重要な点：この図は `express/lib/application.js` や `express/lib/router/index.js` を一切読まずに、シグネチャとREADMEだけで描けます。

</details>

---

### 問2（インターフェース探索）

> ❓ Express のリポジトリ（`https://github.com/expressjs/express`）の `package.json` を開いてください（実装コードは開かない）。`main` フィールドが示すファイルから、Expressが外部に公開しているAPIの一覧を、**そのファイルだけを読んで**書き出してください。

> 📁 対象ファイル: `package.json` → `main` が指すファイル（例: `index.js`）のみ

<details>
<summary>💡 ヒント</summary>

`package.json` の `main` フィールドはnpmがモジュールをインポートしたときに最初に読み込むファイルです。そのファイルが `module.exports = ...` で何をエクスポートしているかを確認してください。

</details>

<details>
<summary>✅ 解答例</summary>

`package.json` の `"main": "index.js"` から `index.js` を開くと：

```javascript
// index.js（実装は読まない、エクスポートだけ確認）
module.exports = require('./lib/express');
```

さらに `lib/express.js` のエクスポート部分（**実装部分は読まない**）：

```javascript
exports = module.exports = createApplication;
exports.application = proto;
exports.request = req;
exports.response = res;
exports.Route = Route;
exports.Router = Router;
// ... ミドルウェア一覧
exports.json = bodyParser.json;
exports.static = serveStatic;
// ...
```

外部に公開されているAPIとして確認できるもの：
- `express()` / `createApplication` （アプリ生成関数）
- `express.Router` （独立したルーター）
- `express.json()` （ボディパーサー）
- `express.static()` （静的ファイル配信）

**実装コードは一行も読まずに、エクスポート宣言だけでAPIサーフェスが把握できます。**

</details>

---

### 問3（抽象化の限界）

> ❓ 今回の演習で「インターフェースだけでは答えられなかった」または「推測が難しかった」ことを正直に書き出してください。その上で「もし実装を読むとしたら、どのファイルのどの部分を読めば解決しそうか」を推定してください。

> ⭐ この問いに「正解」はありません。自分の限界を言語化することが目的です。

<details>
<summary>💡 ヒント</summary>

たとえば以下のような点が疑問になったかもしれません：
- `next(err)` が呼ばれたとき、Expressはエラーハンドラをどうやって識別するの？
- `app.use()` と `app.get()` の優先順位関係は？
- 同じパスに複数のミドルウェアを登録したらどうなる？

</details>

<details>
<summary>✅ 解答例</summary>

典型的な「限界のポイント」と対処法：

| 不明だった点 | 推測の限界 | 読むべき箇所の仮説 |
|---|---|---|
| エラーハンドラの識別方法 | `next(err)` を渡せばいいとはわかるが、Expressがどう識別するか不明 | `lib/router/layer.js` の `handle_error` 周辺か |
| ミドルウェアの内部データ構造 | 「配列に追加する」程度しか推測できない | `lib/router/index.js` の `stack` プロパティ |
| パスマッチングのアルゴリズム | 文字列比較以上のことが起きているはず | `lib/router/layer.js` の `match()` |

この「限界の言語化」が Chapter 03（抽象の漏れへの対応）の演習につながります。

</details>

---

## 📝 振り返り

```
① 実装を読まずに描いた全体像は、どの程度正確だったと思うか？

② 「読まなかった」ことで生じた不確かさは受け入れられたか？

③ 問3で挙げた「限界のポイント」は、今日のタスクにとって必要な情報だったか？
```

---

> 🎉 **Chapter 01 完了！**  
> `ex01` → `ex02` → `ex03` を通じて、「インターフェースを信頼して内部を読まない」スキルの基礎を練習しました。
>
> 次は [Chapter 02：目的駆動の探索](../02_purpose_driven/) に進みましょう。
