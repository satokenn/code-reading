# コードリーディング演習教材

> **目標**: OSSリポジトリなど複数人が参画するプロジェクトのソースコードを、認知資源を効率的に使いながら読み解くスキルを习得する。

---

## この教材で身につける4つの能力

本教材は、以下の4つの能力を段階的に訓練することを目的としています。

| 能力 | 内容 |
|---|---|
| **Ⅰ. 抽象化の信頼** | インターフェースと責務だけを信じ、内部実装をブラックボックスとして扱う |
| **Ⅱ. 目的駆動の探索** | 工学的目的を宣言し、不要な範囲を意図的に飛ばして最短で到達する |
| **Ⅲ. 抽象の漏れへの対応** | 「内部を読む必要があるか」を的確に判断し、必要な箇所だけ深く潜る |
| **Ⅳ. 視点の切り替え** | 俯瞰↔詳細を状況に応じて柔軟に切り替えるメタスキル |

---

## カリキュラム構成

```
curriculum/
├── 00_foundation/       # はじめに — 認知モデル・使い方・失敗パターン
├── 01_abstraction_trust/  # 能力Ⅰ：抽象化を信じる演習
├── 02_purpose_driven/     # 能力Ⅱ：目的駆動の探索演習
├── 03_leaky_abstraction/  # 能力Ⅲ：抽象の漏れ演習
└── 04_meta_switching/     # 能力Ⅳ：視点切り替え統合演習

reference/
├── cheatsheet.md   # grep / git / fd コマンド集
├── tools.md        # 活用ツール一覧
└── glossary.md     # 設計用語集
```

---

## 学習の進め方

1. **まず `curriculum/00_foundation/` を読む**
   - `cognitive_model.md` で4つの能力の全体像を理解する
   - `how_to_use.md` で演習の取り組み方を確認する
   - `anti_patterns.md` でよくある失敗を事前に知る

2. **Chapter 01 から順に演習を進める**
   - 各演習は「目的宣言 → 探索 → 振り返り」の3ステップで行う
   - 答えを見る前に自分の考えを必ず書きとめること

3. **Reference は辞書として使う**
   - 知らないコマンドや用語が出たら都度参照する

---

## 演習で使うOSSリポジトリ

| 難易度 | リポジトリ | 言語 |
|---|---|---|
| ⭐ 入門 | [expressjs/express](https://github.com/expressjs/express) | JavaScript |
| ⭐⭐ 中級 | [pallets/flask](https://github.com/pallets/flask) | Python |
| ⭐⭐⭐ 上級 | [django/django](https://github.com/django/django) | Python |

---

## 推奨環境

- **エディタ**: VS Code（`Go to Definition`, `Find All References` を活用）
- **CLIツール**: `git`, `grep`, `fd`, `gh`（GitHub CLI）
- **ブラウザ**: GitHub の `Code Search`, `Blame` ビューを積極的に活用

---

*このリポジトリはソースコードを書く場所ではなく、ソースコードを**読む力**を鍛える場所です。*
