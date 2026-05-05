# mori-rag

森崇氏の米国株レポート RAG（`insights/`）と X（Twitter）リサーチを組み合わせ、**長文 X 記事** または **YouTube 台本（30分＋Shorts3本）** を生成する [Claude Code](https://claude.ai/claude-code) 用プロジェクトです。

| 用途 | スキル定義（正本） |
|------|---------------------|
| X「記事」向け長文 | `.claude/skills/mori-x-post/SKILL.md` |
| YouTube 台本 | `.claude/skills/mori-yt-script/SKILL.md` |

手順・プロンプト・パスは各 `SKILL.md` を優先してください。こちらの README は概要と X API セットアップです。

---

## 機能（ワークフロー概要）

### X 記事（`mori-x-post`）

1. テーマ受け取り → X リサーチ ON/OFF、X API の有無を確認  
2. X でリサーチ（任意）→ WebSearch → Nitter → X API のフォールバック  
3. 記事構成の整理（リサーチ OFF ならスキップ可）  
4. **RAG** … `insights/` 配下の `.md` ファイル一覧から、テーマに関連するレポートを 5〜8 件選んで Read。足りなければ `insights/*.md` を Grep で補強  
5. 長文 **1 本**を生成 → 確定後、`articles/` に保存  

### YouTube 台本（`mori-yt-script`）

1〜3 は記事スキルと同様の流れ（台本構成ステップあり）。  
4. RAG は `insights/` から同様にレポートを集め、本編に反映する具体事例は **2〜3 件**、Shorts はその事例の別角度での使い回し。  
5. `insights/video_prompt/Louis Gleeson氏プロンプト.md`（旧 `260428_yamamoto-rag-x-main/insights/video_prompt/`）を読み込み、台本を生成 → **`scripts/` に通常版**、続けて **`scripts_tts/` に TTS 用**（読み上げクレンジング済み）を自動保存（各 `SKILL.md` ステップ⑤・⑥）。  

X 記事の途中から **「YouTube台本も作る」** で続けると、リサーチ・RAG を引き継いで台本ステップに進めます（各 SKILL の説明どおり）。

各ステップではスキル内の承認 UI（選択肢）を想定しています。

---

## 生成物の仕様（概要）

| 項目 | X 記事 | YouTube 台本 |
|------|--------|----------------|
| 形式 | X「記事」用 Markdown | Markdown（本編 9,000〜10,000 字目安＋Shorts3本） |
| 分量・構成 | 約 2,000〜3,000 文字、見出し 7〜12 | 12 セクション構成（詳細はプロンプト） |
| 文体 | タメ口・独り言調。ハッシュタグ・絵文字なし | 話し言葉（プロンプトの BRAND DNA に従う） |
| 保存先 | `articles/` | `scripts/`（通常）＋ `scripts_tts/`（TTS 用・自動） |

---

## プロジェクト構成（現状）

```
mori-rag/
├── README.md
├── articles/                    ← 生成した X 記事
├── scripts/                     ← 生成した YouTube 台本（通常版）
├── scripts_tts/                 ← TTS 用クレンジング済み台本（スキルが自動出力）
├── insights/                   ← 森崇氏の米国株レポート（RAG ソース）
│   ├── *.md                     ← 正本（Read 対象）
│   └── *.pdf                    ← 元データ（参考。スキルは md を優先）
└── .claude/skills/
    ├── mori-x-post/SKILL.md
    └── mori-yt-script/SKILL.md
```

YouTube 台本プロンプトファイル（`Louis Gleeson氏プロンプト.md`）は旧プロジェクト（`260428_yamamoto-rag-x-main/insights/video_prompt/`）を参照しています。台本の作り方は変更しない方針です。

---

## データ運用の原則

| 観点 | ルール |
|------|--------|
| 取得 | 1 テーマあたり **広めに 5〜10 ファイル** 読んでもよい（`insights/*.md`） |
| 反映 | X 記事: **1〜2 件** / 台本本編: **2〜3 件**（Shorts は本編事例の使い回し） |
| 元データ | `insights/` のファイルは移動・削除しない |
| 再利用 | 同一レポートファイルの頻度・切り口はユーザーの指示に従う（リポ側にクールダウンログは持たない） |

新しいレポートを `insights/` に追加すれば、そのまま RAG 候補に含まれます。

---

## ローカルパスについて

`SKILL.md` 内の **`insights` 等のパスは絶対パス**で書かれています。**クローン先やマシンが違う場合は、自分の環境のパスに合わせて編集**してください。

---

## 必要な環境

- [Claude Code](https://claude.ai/claude-code) がインストール済みであること  
- インターネット接続（X リサーチ用）  

---

## セットアップ

### 1. リポジトリをクローン

```bash
git clone <repo-url>
cd 260505_mori-rag-main
```

（フォルダ名は任意です。合わせて `.claude/skills/*/SKILL.md` のパスも更新してください。）

### 2. X API の設定（任意）

X API がなくてもスキルは動作します（WebSearch + Nitter 等）。  
設定すると検索精度が上がります。

---

## X API セットアップ手順

### 料金

| プラン | 料金 | 備考 |
|--------|------|------|
| Pay-Per-Use（従量課金） | 最低 **$5（約750円）** から | 2026年2月〜 |
| ~~Free（無料）~~ | ~~$0~~ | 2026年2月に廃止。503 等の原因になりやすい |

### 前提条件

- 有効な X(Twitter) アカウント  
- クレジットカード  

### STEP 1: Developer Portal でアカウント作成 & Bearer Token 取得

1. **https://developer.x.com** にサインイン  
2. アカウント名・利用目的（**250文字以上**）を入力して送信  
3. **「Projects & Apps」** → 自分の App → **「Keys and tokens」** → **Bearer Token** を Generate してコピー  

> Bearer Token は再表示されないことが多いです。失ったら Regenerate してください。

### STEP 2: Developer Console で Pay-Per-Use に切り替え

1. **https://console.x.com** を開く（STEP 1 と別ページ）  
2. **「アプリ」** で App が **Free** の場合は **Pay-Per-Use** に変更  

> Free のままだと検索 API で `403` などになりやすいです。

### STEP 3: クレジット購入 & 支出上限

1. **「請求書作成」→「クレジット」** で $5 などをチャージ  
2. **支出上限**を設定（無制限のままにしない）  

### STEP 4: 環境変数

```bash
echo 'export X_API_BEARER_TOKEN="ここにBearer Tokenを貼り付け"' >> ~/.bashrc
source ~/.bashrc
```

macOS（zsh）の場合は `~/.zshrc` が一般的です。

### STEP 5: 動作確認

```bash
echo $X_API_BEARER_TOKEN

curl -s -H "Authorization: Bearer ${X_API_BEARER_TOKEN}" \
  "https://api.x.com/2/tweets/search/recent?query=test&max_results=10" \
  | python3 -m json.tool | head -20
```

`"data"` が返れば成功のことが多いです。

---

## トラブルシューティング

| エラー | 原因の例 | 対応 |
|--------|-----------|------|
| `403 Client Forbidden` | App が Free のまま | Pay-Per-Use に切り替え |
| `503 Service Unavailable` | 旧 Free キー等 | Pay-Per-Use へ移行・キー再確認 |
| `401 Unauthorized` | Token 不正・期限 | Regenerate して環境変数を更新 |

---

## 使い方（例）

Claude Code で、テーマと「X 記事を書いて」「YouTube 台本を」などと依頼するとスキルが起動します。

```
アップルの決算速報をテーマにXの長文を書いて
```

---

## 参考リンク

- [X Developer Portal](https://developer.x.com)  
- [X Developer Console](https://console.x.com)  
- [X API セットアップ詳細ガイド](https://www.and-and.co.jp/gas-lab/x-twitter-api-start/)  
- [503 解決（Pay-Per-Use）](https://www.and-and.co.jp/gas-lab/x-twitter-api-503-error/)  
- [X API 料金](https://docs.x.com/x-api/getting-started/pricing)  

---

## ライセンス

Private - All Rights Reserved
