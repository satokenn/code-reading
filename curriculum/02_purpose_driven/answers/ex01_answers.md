# 解答：演習01 — バグハント

> ⚠️ **自分の回答を書いてから開いてください。**

---

## 問1 — 目的宣言の練習

**1. バグが発生しているファイルと行番号:**

```
ファイル: /app/routes/posts.js
行番号: 14（直接エラーが出た行）、原因は 13 行目の可能性あり
```

**2. 目的の宣言例:**

```
🎯 目的：routes/posts.js の14行目付近で req.user が undefined になる原因を特定する
```

**3. 読まなくていいファイル（全部）:**

スタックトレースが `routes/posts.js:14` を指しているため、以下はすべて今回の目的に関係ない：

- `routes/users.js` — 投稿と無関係
- `models/Post.js`, `models/User.js` — モデル定義はバグ箇所ではない
- `controllers/postController.js` — スタックトレースの14行目は routes/ 側が原因
- `controllers/userController.js` — 同上
- `config/database.js` — DB接続はバグと無関係
- `middleware/validate.js` — バリデーションはルートに到達した後の処理

今回読む必要があるのは **`routes/posts.js`** と、疑わしければ **`middleware/auth.js`** のみ。

---

## 問2 — 最短ルートの設計例

```
1. routes/posts.js を開く
   → 14行目周辺だけを読む（他のルートは飛ばす）

2. 13行目: req.user.id の req.user が undefined かどうかを確認
   → req.user はどこで付与されるかを推測する

3. auth ミドルウェアが req.user を付与しているという仮説を立てる
   → auth.js を「必要なら」開く（まずは仮説だけで判断を試みる）
```

---

## 問3 — 原因の推測と auth の役割

**13行目 `req.user` が undefined になる状況:**

`req.user` はExpressの標準プロパティではなく、ミドルウェアが付与するカスタムプロパティです。  
`router.get('/user', auth, ...)` で `auth` が先に実行されますが、`auth` が `req.user` を正しく設定しなかった、または `next()` を呼ばずにリクエストを終了させた場合に undefined になります。

典型的な原因：
- JWTトークンが無効 → `auth` が `401` を返して `next()` を呼ばないはずなのに、`next()` を呼んでしまっている
- `auth` ミドルウェアに条件分岐のバグがあり、トークンなしでも `next()` が呼ばれる

**`auth` の推測される役割（このファイルだけから）:**

`auth` が認証専用のミドルウェア名であること、そして `req.user.id` を参照していることから、`auth` は「JWTやセッションからユーザー情報を取得して `req.user` に設定する」ミドルウェアだと推測できます。

---

## 問4 — 内部実装を読む必要があるか

**判断: B（読む必要がある）**

**理由:**

インターフェースの理解（`auth` は認証ミドルウェア）だけでは、「なぜ `req.user` が undefined になったか」の根本原因は特定できません。

`auth.js` で `next()` が不適切に呼ばれているか、`req.user` の設定が条件分岐で漏れているかを確認するには、実装を読む必要があります。ただし読むべき範囲は `auth.js` の `req.user` 設定と `next()` 呼び出し箇所**のみ**に限定します。

これが **能力Ⅲ（抽象の漏れ）** の入口です。Chapter 03 でより深く訓練します。

---

← 問題に戻る [`ex01_bug_hunt.md`](../ex01_bug_hunt.md)
