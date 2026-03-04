# 演習01：インターフェースだけを読む — `express` のミドルウェア連鎖

## 🎯 この演習が鍛える能力

**能力Ⅰ：抽象化の信頼**  
関数のシグネチャとドキュメントだけを手がかりに、内部実装を読まずに動作を推論する。

> ⚠️ **解答は [`answers/ex01_answers.md`](./answers/ex01_answers.md) に分離されています。**  
> 問題をすべて解き終えてから開いてください。

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

### 問1
`RequestHandler` の引数 `next: NextFunction` の役割を、型定義と使用例だけから推測してください。「next を呼ぶ」と「呼ばない」の違いは何だと思いますか？

---

### 問2
`app.use()` の返り値が `this` です。これはどのような使い方を可能にしますか？具体的なコード例を書いてください。

---

### 問3
以下のコードを見て、リクエストが `GET /` に来たとき、どの関数がどの順番で実行されるかを書き出してください。実装コードは**読まないで**、シグネチャと使用例だけで推論してください。

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

---

### 問4
`app.use()` の設計は「登録順に実行される」という特性を持っています。これはなぜ重要だと思いますか？認証ミドルウェアとロギングミドルウェアを例に、順序が持つ意味を説明してください。実装を読まずに考えてください。

---

## 📝 振り返り

全問解き終えたら記入してください。

```
① 「読まなかった」判断は正しかったか？
   （内部実装を読まずに問いに答えられたか？）

② 予想と実際に違ったことはあったか？

③ 次回どう変えるか？
```

---

解答確認 → [`answers/ex01_answers.md`](./answers/ex01_answers.md)  
次の演習 → [`ex02_module_contract.md`](./ex02_module_contract.md)
