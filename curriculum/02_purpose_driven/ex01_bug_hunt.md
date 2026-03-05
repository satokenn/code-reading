# 演習01：バグハント — エラーログから問題箇所に最短で到達する

## 🎯 この演習が鍛える能力

**能力Ⅱ：目的駆動の探索**  
工学的な目的（バグ修正）を明確に宣言し、関係ない領域を意図的に飛ばしながら最短ルートで問題箇所に到達する。

> ⚠️ **解答は [`answers/ex01_answers.md`](./answers/ex01_answers.md) に分離されています。**  
> 問題をすべて解き終えてから開いてください。

---

## 📋 目的宣言（演習前に必ず記入）

```
🎯 目的：
📍 到達すべき箇所の仮説：
🚫 除外（今回は読まない領域）：
```

---

## 📚 シナリオ

あなたはExpressで構築された既存のAPIサーバーを引き継ぎました。ある日、以下のエラーログが報告されます。

```
TypeError: Cannot read properties of undefined (reading 'id')
    at getUserPosts (/app/routes/posts.js:14:27)
    at Layer.handle [as handle_request] (/app/node_modules/express/lib/router/layer.js:95:5)
    at next (/app/node_modules/express/lib/router/route.js:149:13)
    at Route.dispatch (/app/node_modules/express/lib/router/route.js:119:3)
    at Layer.handle [as handle_request] (/app/node_modules/express/lib/router/layer.js:95:5)
    at /app/node_modules/express/lib/router/index.js:284:15
```

プロジェクトのディレクトリ構成は以下のとおりです。

```
/app/
├── server.js          # エントリポイント
├── routes/
│   ├── posts.js       # 投稿関連ルート
│   └── users.js       # ユーザー関連ルート
├── middleware/
│   ├── auth.js        # 認証ミドルウェア
│   └── validate.js    # バリデーション
├── models/
│   ├── Post.js
│   └── User.js
├── controllers/
│   ├── postController.js
│   └── userController.js
└── config/
    └── database.js
```

---

## 🔍 演習問題

### 問1（目的宣言の練習）

エラーログを読んで、以下を答えてください。コードはまだ**開かないでください**。

1. バグが発生しているファイルと行番号はどこですか？
2. 「今日の目的」を1文で宣言してください
3. 上記のディレクトリ構成を見て、**読まなくていいファイル**をすべて列挙してください

---

### 問2（最短ルートの設計）

スタックトレースとディレクトリ構成だけを使い、読む順番を設計してください。

> 「どのファイルを、どの順番で、どこまで読むか」を箇条書きにしてください。  
> 実際のコードはまだ開かないこと。

---

### 問3（読む・飛ばす の実践）

以下は `routes/posts.js` の内容です。バグを特定するために**必要な行だけ**に集中し、他は意識的に飛ばしてください。

```javascript
// routes/posts.js
const express = require('express');
const router = express.Router();
const { auth } = require('../middleware/auth');
const { validatePost } = require('../middleware/validate');
const postController = require('../controllers/postController');

// 全投稿を取得
router.get('/', postController.getAllPosts);

// ユーザーの投稿を取得
router.get('/user', auth, (req, res) => {
  const userId = req.user.id;           // line 13
  postController.getUserPosts(userId, res); // line 14
});

// 新規投稿を作成
router.post('/', auth, validatePost, postController.createPost);

// 投稿を更新
router.put('/:id', auth, validatePost, postController.updatePost);

// 投稿を削除
router.delete('/:id', auth, postController.deletePost);

module.exports = router;
```

設問：
1. スタックトレースが指す14行目は `postController.getUserPosts(userId, res)` です。しかし本当の原因は13行目にある可能性があります。13行目の `req.user` が `undefined` になる状況を推測してください
2. `auth` ミドルウェアの役割を、このファイルだけから推測してください（`middleware/auth.js` はまだ開かないこと）

---

### 問4（境界線の判断）

問3の調査で「`auth` ミドルウェアの内部実装を読む必要があるか」を判断してください。

以下のどちらに該当するか選び、理由を述べてください：

- **A**: インターフェースの理解だけで原因が特定できる → 読まなくてよい
- **B**: `auth.js` の内部実装を読まないと原因が特定できない → 読む必要がある

---

## 📝 振り返り

```
① 当初の「読まないファイル」の選択は正しかったか？

② 最短ルートの設計通りに到達できたか？外れた箇所はあったか？

③ 次回どう変えるか？（目的宣言の精度・スタックトレースの読み方など）
```

---

解答確認 → [`answers/ex01_answers.md`](./answers/ex01_answers.md)  
次の演習 → [`ex02_feature_add.md`](./ex02_feature_add.md)
