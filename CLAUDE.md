# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## プロジェクト概要

地域コミュニティセンターの月刊おたよりに掲載するURLからスマホでアクセスできる、知的ミニゲームサイト。GitHub Pages でホスティングする静的サイト。

## 開発・確認方法

ビルド不要。HTMLファイルをブラウザで直接開くか、ローカルサーバーで確認する。

```
# Python でローカルサーバーを立てる場合（ルートから）
python -m http.server 8080
# → http://localhost:8080 でトップページを確認
```

Google Sheets API を使うゲームはローカルサーバー経由でないと CORS エラーになるため注意。

## アーキテクチャ

**依存関係なし・フレームワーク不使用**。各ゲームは完全に独立した単一HTMLファイル。

```
index.html              ← トップページ（ゲーム一覧）
config.js               ← Google Sheets APIキー・スプレッドシートID（全ゲーム共通）
quiz/index.html         ← 4択クイズ
crossword/index.html    ← クロスワードパズル
memory-game/index.html  ← 絵合わせ（神経衰弱）
memory-game/images/cards.js ← 絵合わせのカードデータ
```

### 画面遷移パターン（全ゲーム共通）

各ゲームは `.screen` クラスを持つ複数のdivで構成され、`showScreen(id)` でクラス `active` を付け替えることで画面を切り替える。ページ遷移なし、シングルページ。

- `screen-loading` → `screen-start` → `screen-question/game` → `screen-result`

### データソース

`config.js` の `CONFIG` オブジェクトを `<script src="../config.js">` で読み込む（各ゲームHTMLから相対パス）。

| ゲーム | シート名 | 列構成 |
|--------|----------|--------|
| クイズ | `CONFIG.SHEET_NAME`（クイズ） | A:番号, B:問題文, C:正解, D-F:不正解3択, G:解説, H:カテゴリ |
| クロスワード | `CONFIG.CROSSWORD_SHEET`（クロスワード） | A:横単語, B:縦単語, C:ヒント（`/`区切りで横・縦） |
| 絵合わせ | `CONFIG.MEMORY_SHEET`（絵合わせ） | スプレッドシートから取得（または `cards.js` のフォールバック） |

クロスワードは横・縦の単語に共通文字があることが必須。共通文字がない行は自動スキップされる。

### クロスワードのグリッド構造

`buildCrossItem()` が1問分の十字形グリッドを構築する。`cols = yoko.length`、`rows = tate.length` のグリッドで、交差位置 `grid[tateIdx][yokoIdx]` だけが答えマス（オレンジ枠）になる。

文字入力は画面下からスライドアップするダイアログで行う。カ・サ・タ・ハ行は濁音・半濁音を同一画面に一覧表示する2ステップ方式（行選択 → 文字選択）。

## デザイン規約

- テーマカラー：オレンジ `#e07b39`（hover: `#c96a28`）、ベージュ背景 `#fdf6ee`
- カード背景：`#fffaf4`、影：`box-shadow: 0 4px 16px rgba(180, 120, 60, 0.15)`
- テキスト：ダークブラウン `#4a3728`、サブテキスト `#8c6b4a`
- `max-width: 480px`（スマホ縦持ち想定）、`border-radius: 16〜20px`
- ボタン：`.btn-primary`（オレンジ）/ `.btn-secondary`（ベージュ）の2種
- 正解色：緑系 `#d4edda` / `#4cae4c`、不正解色：赤系 `#f8d7da` / `#d9534f`
- フォント：`'Hiragino Kaku Gothic ProN', 'Hiragino Sans', 'Meiryo', sans-serif`

新しいゲームを追加する場合は既存ゲームのHTML構造・CSS変数・ボタンクラスを踏襲すること。

## コンテンツの更新（問題・カードデータ）

ゲームのコンテンツはすべて Google スプレッドシートで管理。スプレッドシートを保存するだけでサイトに反映される（ファイル編集不要）。

**クイズ**（「クイズ」シート）: A:問題番号, B:問題文, C:**正解**, D-F:不正解3択, G:解説, H:カテゴリ

**クロスワード**（「クロスワード」シート）: A:横単語(カタカナ), B:縦単語(カタカナ), C:ヒント(`/`区切りで横・縦の順)
→ A列とB列に共通文字がない行は自動スキップされる

**出題数の変更**: [config.js](config.js) の `MAX_QUESTIONS` を編集する

## サイト（HTMLファイル等）の更新・公開

ファイルを編集後、git で push するだけで GitHub Pages に自動反映される（1〜2分）。

```powershell
git add .
git commit -m "変更内容のメモ"
git push
```

リポジトリ: https://github.com/arums-jp/community-games
