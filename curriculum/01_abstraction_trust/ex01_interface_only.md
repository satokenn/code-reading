# 演習01：インターフェースだけを読む — `express` のミドルウェア連鎖

## 🎯 この演習が鍛える能力

**能力Ⅰ：抽象化の信頼**  
関数のシグネチャとドキュメントだけを手がかりに、内部実装を読まずに動作を推論する。

---

## 📋 目的宣言（演習前に記入）

演習を始める前に、以下をあなたのメモに書いてください。

```
🎯 目的：
📍 仮説（どこを読めば分かると思うか）：
🚫 除外（今回は読まない領域）：
```

---

## 📚 背景知識

Express は Node.js の軽量Webフレームワークです。`app.use()` はその中核をなすAPIで、**ミドルウェア**と呼ばれる関数を登録するために使います。

以下は Express の `app.use()` のシグネチャ（TypeScript型定義より）です。**実装は見ません。**

```typescript
// Express の型定義（@types/express より抜粋）
interface Application extends EventEmitter, IRouter {
  use(path: PathParams, ...handlers: RequestHandler[]): this;
  use(path: PathParams, ...handlers: RequestHandlerParams[]): this;
  use(...handlers: RequestHandler[]): this;
  use(...handlers: RequestHandlerParams[]): this;
}

// RequestHandler の型
interface RequestHandler<
  P = ParamsDictionary,
  ResBody = any,
  ReqBody = any,
  ReqQuery = ParsedQs
> {
  (
    req: Request<P, ResBody, ReqBody, ReqQuery>,
    res: Response<ResBody>,
    next: NextFunction
  ): void;
}
```

そして、以下が実際の使用例です。

```javascript
const express = require('express');
const app = express();

// ミドルウェアを登録
app.use((req, res, next) => {
  console.log('ミドルウェア1');
  next(); // 次のミドルウェアへ
});

app.use((req, res, next) => {
  console.log('ミドルウェア2');
  next();
});

app.get('/', (req, res) => {
  res.send('Hello World');
});
```

---

## 🔍 演習問題

### 問1（ウォームアップ）
> ❓ `RequestHandler` の引数 `next: NextFunction` の役割を、型定義と使用例だけから推測してください。「next を呼ぶ」と「呼ばない」の違いは何だと思いますか？

<details>
<summary>💡 ヒント</summary>

- `next()` が呼ばれないとき、その後に登録されたミドルウェアはどうなるか？
- 使用例の「ミドルウェア1」が `next()` を呼ばなかった場合、「ミドルウェア2」は実行されるか？

</details>

<details>
<summary>✅ 解答例</summary>

`next` は「処理を次のミドルウェアに渡す」関数です。

- `next()` を **呼ぶ** → 次に登録されたミドルウェアが実行される
- `next()` を **呼ばない** → 処理がそこで止まり、以降のミドルウェアは実行されない

これは Express のミドルウェアチェーンが「連鎖的な制御フロー」を構成しているためです。内部実装を読まなくても、`next` の存在と使用例から「通過するか止めるかを選べる設計」であることが読み取れます。

</details>

---

### 問2（ウォームアップ）
> ❓ `app.use()` の返り値が `this` です。これはどのような使い方を可能にしますか？具体的なコード例を書いてください。

<details>
<summary>💡 ヒント</summary>

`this` を返すメソッドによく見られるパターンを思い浮かべてみてください。jQuery の `$('.foo').addClass('bar').show()` のような書き方はどう呼ばれますか？

</details>

<details>
<summary>✅ 解答例</summary>

返り値が `this` であることで、**メソッドチェーン（Fluent Interface）** が使えます。

```javascript
app
  .use(middlewareA)
  .use(middlewareB)
  .use(middlewareC);
```

内部実装を読まなくても、シグネチャの `this` という返り値の型だけで「連続して呼び出せる」ことがわかります。

</details>

---

### 問3（フロー追跡）
> ❓ 以下のコードを見て、リクエストが `GET /` に来たとき、どの関数がどの順番で実行されるかを書き出してください。実装コードは**読まないで**、シグネチャと使用例だけで推論してください。

```javascript
app.use((req, res, next) => {
  req.timestamp = Date.now();
  next();
});

app.use('/api', (req, res, next) => {
  req.isApi = true;
  next();
});

app.get('/', (req, res) => {
  res.json({ timestamp: req.timestamp, isApi: req.isApi });
});

app.get('/api/users', (req, res) => {
  res.json({ isApi: req.isApi });
});
```

<details>
<summary>💡 ヒント</summary>

- `app.use(handler)` と `app.use('/path', handler)` の違いは何でしょうか？
- `path` 引数がある場合、どのリクエストに対してそのミドルウェアが実行されるか？

</details>

<details>
<summary>✅ 解答例</summary>

**`GET /` の場合:**
1. `req.timestamp = Date.now(); next();` ← パス指定なし → すべてのリクエストに適用
2. `/api` のミドルウェアは**スキップ**（パスが `/api` で始まらないため）
3. `app.get('/')` のハンドラが実行される
4. `res.json({ timestamp: ..., isApi: undefined })` が返る

**`GET /api/users` の場合:**
1. `req.timestamp = Date.now(); next();` ← 全リクエスト適用
2. `req.isApi = true; next();` ← `/api` で始まるのでマッチ
3. `app.get('/api/users')` のハンドラが実行される
4. `res.json({ isApi: true })` が返る

「パス引数があるかどうか」でマッチ条件が変わることが、シグネチャの `path: PathParams` オプションから推論できます。

</details>

---

### 問4（設計意図）
> ❓ `app.use()` の設計は「登録順に実行される」という特性を持っています。これはなぜ重要だと思いますか？認証ミドルウェアとロギングミドルウェアを例に、順序が持つ意味を説明してください。実装を読まずに考えてください。

<details>
<summary>💡 ヒント</summary>

「認証が通っていないリクエストに対してロギングを行うべきか？」という問いを考えてみてください。また逆の順序ではどうなるでしょうか？

</details>

<details>
<summary>✅ 解答例</summary>

登録順が実行順を決める設計により、**ミドルウェアの責任の連鎖を明示的に制御**できます。

```javascript
// 推奨される順序
app.use(logging);       // ① まずすべてのアクセスを記録
app.use(authenticate);  // ② 認証チェック（失敗したらここで停止）
app.use(rateLimit);     // ③ 認証済みユーザーにのみレート制限
app.get('/data', ...);  // ④ 実際の処理
```

認証を①に置くと、「記録されていない不正アクセス」が生じるリスクがあります。順序を意識した設計により、セキュリティ・可観測性・パフォーマンスのトレードオフを明示的に扱えます。

これは内部実装を知らなくても、`app.use()` の「登録順に実行」という契約（インターフェース仕様）だけから引き出せる設計上の知識です。

</details>

---

## 📝 振り返り

演習が終わったら以下に答えてください。

```
① 「読まなかった」判断は正しかったか？
   （内部実装を読まずに問いに答えられたか？）

② 予想と実際に違ったことはあったか？

③ 次回どう変えるか？
```

---

> ✅ **確認**: 内部実装（`express/lib/application.js` の `app.use` の中身）は**まだ読まないでください**。  
> 次の演習 [ex02_module_contract.md](./ex02_module_contract.md) に進みましょう。
