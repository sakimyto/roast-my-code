# 🔥 roast-my-code

[English](README.md)

**コードレビュー、されるなら面白い方がいい。**

[Claude Code](https://docs.anthropic.com/en/docs/claude-code) 用スキル。毒舌で的確なコードレビューを、具体的な改善案つきで届けます。APIキー不要。依存ゼロ。マークダウンだけ。

## デモ

```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
  🔥 ROAST MY CODE — REPORT CARD 🔥
  Level: spicy | Target: ./
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

📊 OVERALL: B (74/100)

┌─────────────────────────────────────┐
│ Category          Score    Grade    │
├─────────────────────────────────────┤
│ Security          80/100     A     │
│ Architecture      70/100     B     │
│ Complexity        80/100     A     │
│ TDD               60/100     C     │
│ Type Safety       70/100     B     │
│ Error Handling    60/100     C     │
│ Naming            76/100     B     │
│ Dead Code         76/100     B     │
│ Performance       90/100     S     │
│ Dependencies      90/100     S     │
│ Frontend          76/100     B     │
│ Git Hygiene       88/100     A     │
└─────────────────────────────────────┘

━━━ 🔥 FINDINGS ━━━━━━━━━━━━━━━━━━━━━━

## TDD (60/100)

### 🚨 テスト比率が低い [error]
📍 apps/web/src/
> ソースファイル180個に対してテスト46個（26%）

💬 "なんだろう、テスト書くの後回しにしてるんですけど、
    その『後で』は永遠に来ないですよね。"

🔧 修正: ビジネスロジックから優先してテスト追加。
   まずはクリティカルパスから1ファイル1テストを目指す。

---

## Error Handling (60/100)

### 🚨 空のcatchブロック [critical]
📍 src/lib/api-client.ts:42
> } catch (e) {}

💬 "catchして何もしない。それってエラー処理じゃなくて
    エラー隠蔽ですよね。見えないから大丈夫って思ってません？"

🔧 修正: 最低限ログ出力。理想はコンテキスト付きで
   rethrowするか、失敗ケースを個別にハンドリング。

━━━ 📋 SUMMARY ━━━━━━━━━━━━━━━━━━━━━━━

🏆 よくできている点:
  1. Performance — N+1クエリなし、非同期I/O使用
  2. Dependencies — lockfileあり、ワイルドカードなし
  3. Git Hygiene — 意味のあるコミットメッセージ

💀 優先的に直すべき点:
  1. 空のcatchブロック (src/lib/api-client.ts)
  2. ビジネスロジックのテストカバレッジ
  3. 8箇所の明示的な `any` 型宣言

📈 すぐできる改善 (各5分以内):
  - tsconfig.json に `strict: true` を追加
  - console.log を構造化ロガーに置換
  - src/utils/ の未使用import 3件を削除

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
  Roasted with ❤️ by roast-my-code
  ⭐ github.com/sakimyto/roast-my-code
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
```

## インストール

3ステップ。npm不要。設定不要。コピーするだけ。

### 1. クローン

```bash
git clone https://github.com/sakimyto/roast-my-code.git
```

### 2. スキルをコピー

**プロジェクト単位（推奨）:**

```bash
cp -r roast-my-code/roast-my-code your-project/.claude/skills/
```

**グローバル（全プロジェクトで使用）:**

```bash
cp -r roast-my-code/roast-my-code ~/.claude/skills/
```

### 3. 焼く

Claude Code でプロジェクトを開いて:

```
/roast
```

以上。

## 使い方

```bash
/roast                              # cwdをspicyレベルでレビュー
/roast src/                         # 特定ディレクトリを指定
/roast --level=savage               # 最大火力
/roast src/ --level=sensei          # TDD師匠モード
/roast --diff                       # ステージ済み変更だけレビュー
/roast --quick                      # 高速スキャン（主要3カテゴリ）
/roast --lang=en                    # 英語で出力を強制
```

## 煽りレベル

| レベル | トーン | 例 |
|--------|--------|---|
| `gentle` | 優しいメンター | "いい感じです。ここだけ直すともっと安心です。" |
| **`spicy`** | ひろゆき風（デフォルト） | "なんだろう、テストないのやめてもらっていいですか。" |
| `savage` | 容赦ないシニア | "これ人間が書いたんですか？純粋な疑問として。" |
| `sensei` | TDD師匠（t_wada式） | "書かれなかったテストは、自らデプロイ日を選んだバグです。" |

## チェック内容

13カテゴリ、140以上のチェック項目:

| カテゴリ | 内容 | 重み |
|----------|------|------|
| **Security** | ハードコード秘密鍵、SQL/XSSインジェクション | 1.5x |
| **Architecture** | 神ファイル、循環依存、密結合 | 1.2x |
| **Complexity** | 長い関数、深いネスト、循環的複雑度 | 1.0x |
| **TDD** | テストカバレッジ、アサーション品質 | 1.0x* |
| **Type Safety** | `any` 乱用、strictモード、型アサーション | 1.0x |
| **Error Handling** | 空catch、エラー握りつぶし、string throw | 1.0x |
| **Naming** | 汎用名、ケース不統一、省略形 | 1.0x |
| **Dead Code** | コメントアウト、未使用import、放置TODO | 1.0x |
| **Performance** | N+1クエリ、同期I/O、メモリリーク | 1.0x |
| **Dependencies** | lockfileなし、ワイルドカードバージョン | 1.0x |
| **API Design** | 入力バリデーション、レスポンス形式 | 1.0x |
| **Frontend** | アクセシビリティ、prop drilling | 1.0x |
| **Git Hygiene** | .gitignoreなし、巨大ファイル、コミットメッセージ | 1.0x |

*\*senseiモードではTDDの重みが1.5xに上昇*

条件付きチェッカー（API Design, Frontend, Git Hygiene, Dependencies）は対応するフレームワークやツールが検出された場合のみ有効化されます。

## バイリンガル対応

入力言語を自動検出して出力を切り替えます:

- **英語**: Tech Twitter映えするドライなユーモア
- **日本語**: ひろゆき風の丁寧な毒舌 — 「それって〜ですよね」

`--lang=ja` / `--lang=en` で手動切り替えも可能。

## 対応言語

チェック定義は **JavaScript / TypeScript** に最適化されています。多くのチェック（Security, Architecture, Complexity, TDD, Naming, Dead Code, Git Hygiene）は言語を問わず動作します。フレームワーク固有のチェック（Dependencies, Frontend, API Design）は現在Node.jsエコシステム中心です。

## 成績表

| 評価 | スコア | 意味 |
|------|--------|------|
| **S** | 90-100 | 完璧 — 今すぐシップ |
| **A** | 80-89 | 堅実 — 本番投入OK |
| **B** | 70-79 | まあまあ — コードレビュー通過 |
| **C** | 60-69 | 要改善 — もうひと頑張り |
| **D** | 40-59 | 厳しい — 課題多数 |
| **F** | 0-39 | 炎上 — 消防車を呼べ |

## 必要なもの

- [Claude Code](https://docs.anthropic.com/en/docs/claude-code) CLI がインストール済み
- 以上。APIキー不要 — Claude Code自体がLLMです。

## 仕組み

roast-my-codeはマークダウンだけで構成されたClaude Codeスキルです。Claudeの組み込みツール（Read, Glob, Grep, Bash）でコードベースを分析し、結果をroast形式で出力します。`references/` 内の参照ファイルが検出パターン、重大度、roast例を定義しています。

## コントリビュート

1. リポジトリをフォーク
2. `references/*.md` にチェック項目を追加・改善
3. 改善効果がわかる出力例を添えてPR

roast例のルール:

- 事実に基づいている（実際のコードパターンから）
- 面白い（それが全て）
- 改善可能（必ず修正方法とセット）

## ライセンス

[MIT](LICENSE)

---

*AI時代のために生まれた。マークダウンで動く。皮肉で走る。*
