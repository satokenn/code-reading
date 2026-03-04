# 演習03：抽象化の信頼テスト — 実装を読まずに全体像を描く

## 🎯 この演習が鍛える能力

**能力Ⅰ：抽象化の信頼（総合）**  
実際のOSSリポジトリに対して、シグネチャ・README・docstringのみを手がかりに、システム全体の処理フロー図を描く。

> ⚠️ **解答は [`answers/ex03_answers.md`](./answers/ex03_answers.md) に分離されています。**  
> 問題をすべて解き終えてから開いてください。

> この演習では「正確な図」よりも「実装を読まずに推論できること」が評価されます。

---

## 📋 目的宣言（演習前に記入）

```
🎯 目的：
📍 仮説（どこを読めば分かると思うか）：
🚫 除外（今回は読まない領域）：
```

---

## 📚 背景知識（使ってよい情報）

以下のAPIリストとREADME抜粋のみを使って演習を進めてください。

**公開API一覧（シグネチャのみ）:**

```typescript
// アプリケーション
const app = express();

// ミドルウェア登録
app.use(path?, ...handlers): Application
app.get(path, ...handlers): Application
app.post(path, ...handlers): Application

// リクエストオブジェクト（主要プロパティ）
req.method: string          // "GET", "POST" など
req.path: string            // "/users/42" など
req.params: object          // { user_id: "42" } など
req.query: object           // { page: "1" } など
req.body: any               // リクエストボディ（パース済み）
req.headers: object

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

上記の情報だけを使い、`POST /users` というHTTPリクエストが Express アプリに届いてから `res.json()` でレスポンスが返るまでの処理フローを、以下のテンプレートに書き起こしてください。実装ファイルは開かないでください。

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

---

### 問2（インターフェース探索）

Express のリポジトリ（`https://github.com/expressjs/express`）の `package.json` を開いてください（実装コードは開かない）。`main` フィールドが示すファイルから、Expressが外部に公開しているAPIの一覧を、**そのファイルだけを読んで**書き出してください。

> 📁 対象ファイル: `package.json` → `main` が指すファイル（例: `index.js`）のみ

---

### 問3（抽象化の限界）

今回の演習で「インターフェースだけでは答えられなかった」または「推測が難しかった」ことを正直に書き出してください。その上で「もし実装を読むとしたら、どのファイルのどの部分を読めば解決しそうか」を推定してください。

> ⭐ この問いに「正解」はありません。自分の限界を言語化することが目的です。

---

## 📝 振り返り

```
① 実装を読まずに描いた全体像は、どの程度正確だったと思うか？

② 「読まなかった」ことで生じた不確かさは受け入れられたか？

③ 問3で挙げた「限界のポイント」は、今日のタスクにとって必要な情報だったか？
```

---

解答確認 → [`answers/ex03_answers.md`](./answers/ex03_answers.md)  

> 🎉 **Chapter 01 完了！** 次は [Chapter 02：目的駆動の探索](../02_purpose_driven/) へ。
