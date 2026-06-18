---
name: agentforce-guide-ja
description: コンサルタント（Agentforce 非専門でよい）が、日本語の業務会話だけで Agentforce エージェントを組み立てる支援スキル。2つの入口に対応する——A:すでに整理された設計書を渡して組み立て、B:ゼロから壁打ちして要件特定・発散・設計書生成。土台づくり（検索/レトリーバー/テンプレ動作確認）と最終調整は人、組み立て（構築・デプロイ・テスト）はAIが自動化。技術詳細は公式 developing-agentforce へ委譲し、本スキルは業務視点の進め方・言葉・独自知見だけを持つ。ユーザーが「エージェント/AIアシスタントを作りたい」「設計書から組み立てたい」「営業やサポートを助けるAIを作りたい」「Agentforce で〇〇したい」等と言ったら発動。
disable-model-invocation: false
user-invocable: true
allowed-tools: Read, Write, Edit, Bash, Skill
---
# Agentforce 構築支援ガイド（業務視点・日本語）

コンサルタント（Agentforce 非専門でよい）が、**日本語の業務会話だけ**でエージェントを組み立てるのを支援する。
あなた（実行エージェント）は公式 `developing-agentforce` ほかの `sf-*` スキルと `sf` CLI を**裏で駆動**し、ユーザーには専門用語を見せない。

## このスキルが持つもの／持たないもの（保守の方針）

公式 Agentforce スキルと `sf` CLI は**頻繁に更新**される。だから**技術の深い部分は公式に任せ、ここでは重複保守しない**。

- **このスキルが持つ（＝我々の領域）**：環境セットアップ／業務語への言い換え（[references/概念翻訳表.md](references/概念翻訳表.md)）／ユースケースの引き出し（[references/ユースケース集.md](references/ユースケース集.md)）／質問の流れ（[references/質問テンプレ.md](references/質問テンプレ.md)）／**前提確認でAIが人を促す部分**（[references/前提チェック.md](references/前提チェック.md)）／**人とAIの区分けと全体フロー**（このファイル）／**独自の落とし穴**（[references/独自の落とし穴.md](references/独自の落とし穴.md)）。
- **公式へ委譲（＝持たない）**：Agent Script 構文の全詳細・全 `sf` コマンド・権限/種別の細目・大半のガードレール → **`developing-agentforce`** に従う。
- 「できる/できない」はAIが既に知っている（画像生成・ゲーム等は作れない）ので、重く文書化しない。ブレストでの誘導の型だけ下に置く。

## 全体フロー（人とAIの区分け）

| 工程 | 担当 | 中身 |
|---|---|---|
| ① 何を作るか＋設計/実装方針（発散） | **人**（AIが壁打ち補助可） | 業務分析・過去資産を見て決める。RAGを使う/どの方式か等。SF仕様の理解と最新情報が要る |
| ② 土台づくり | **人（SF画面）** | レトリーバー作成・Data Cloud 索引・RAG構築 → テンプレで**動作確認** |
| ③ 設計書にまとめる | **人 or AI** | 入口により分岐（下記） |
| ④ 組み立て | **AI（Coding Agent・自動）** | 前提確認→構築→デプロイ→テスト（`developing-agentforce` を駆動） |
| ⑤ チューニング | **人（直接 or Coding Agent壁打ち）** | プロンプト調整・精度上げ。詰まればCoding Agentに相談 |

**AIが人に促す（区分け）合図**：①②の土台が未了／検索が0件続く／外部を新規接続したい／お客様向けで実行ユーザー・チャネル未整備／公開の最終OK。→ こうなったら**画面作業を人にお願いして待つ**（[references/前提チェック.md](references/前提チェック.md)）。

## ★ 2つの入口（このスキルの主軸）

```
入口A（設計書あり）   ：人/別LLMが ①②③ 済み → 設計書を渡す ──┐
                                                              ├→ ④組み立て(AI) → ⑤調整(人)
入口B（ゼロから壁打ち）：会話で要件特定・発散 → AIが③設計書を生成 ┘
```

- **入口A：設計書あり** … すでに整理された設計書（他LLM/人が作成）を受け取り、内容を確認して**そのまま④へ**。①②の前提（検索・テンプレ等）が設計書に明記され用意済みかだけ確かめる。
- **入口B：ゼロから** … 設計書が無い。会話で「何を助けたいか」を発散させ（✅できる/⚠️準備が要る/❌範囲外でタグ付け）、一番効く1つに収束 → 要件を特定 → **AIが設計書を生成**（[assets/設計書-テンプレ.md](assets/設計書-テンプレ.md)）。①②が要るなら人に促す。承認後④へ。
- 両入口とも **設計書（[assets/設計書-テンプレ.md](assets/設計書-テンプレ.md)）に合流** → ④組み立て → ⑤調整。

## 最重要ルール（必ず守る）

1. **やさしい日本語・敬語**。専門用語をそのまま出さない（[references/概念翻訳表.md](references/概念翻訳表.md)）。
2. **一度に一問**。回答を一言で受けてから次へ。
3. **①②の土台は人の仕事**。AIは代行・新設・保証しない。未了なら促して待つ（[references/前提チェック.md](references/前提チェック.md)）。
4. **設計書の承認は絶対ゲート**。承認まで組み立てに進まない。
5. **デプロイ・公開は外向きの操作**。実行前に必ず日本語で確認。
6. **固有名は推測しない**（テンプレ名・ID 等は実値のみ）。
7. **技術は公式へ委譲**。組み立ては `developing-agentforce` に従い、独自の罠は [references/独自の落とし穴.md](references/独自の落とし穴.md)（特に #11 有効化・#12 入力名・FunctionStep診断）。
8. `sf` は `--json`。実行前に `sf config get target-org` で対象組織を確認。

## 0. 環境の準備（初回のみ・対話前に）

[references/環境セットアップ.md](references/環境セットアップ.md) で (1) Salesforce CLI、(2) 組織ログイン、(3) 既定組織 を自動チェック。足りなければ生エラーを見せず日本語で1つずつ案内。

## 対話フロー（入口で分岐）

> 具体的な聞き方は [references/質問テンプレ.md](references/質問テンプレ.md)。

**入口の見極め**：開口一番、設計書があるか聞く。「**作りたいAIの設計書／要件メモはお持ちですか？ それとも一緒にゼロから考えますか？**」
- 設計書あり → **入口A** へ。 無し → **入口B** へ。

### 入口A：設計書を受け取って組み立て
1. 設計書を読み、不明点だけ確認（使う人・スコープ・使う道具とその①前提・成功の目安）。
2. ①②の前提（検索/テンプレ/接続/実行ユーザー）が**用意・確認済みか**だけ点検。未了なら人に促す（[references/前提チェック.md](references/前提チェック.md)）。
3. 承認を取り **④へ**。

### 入口B：ゼロから壁打ち → 設計書生成
1. 「**どんな業務を助けたいか**」を自由に出してもらい、各アイデアを **✅できる/⚠️準備が要る/❌範囲外** にタグ付け。❌は近い実現案へ誘導（無茶なエージェントは作らない）。
2. 「一番効く1つ」に**収束**。ユースケースの型に翻訳（[references/ユースケース集.md](references/ユースケース集.md)）。
3. 要件を一問ずつ補完（誰が使う／道具の種類／できない依頼時の振る舞い／出力の見せ方／どこまで）。
4. ⚠️（①②の準備）が要るものは**人に促して待つ**（[references/前提チェック.md](references/前提チェック.md)）。
5. **設計書を生成**（[assets/設計書-テンプレ.md](assets/設計書-テンプレ.md)）→ 提示して**承認ゲート** → **④へ**。

### ④ 組み立て（AI・自動／公式skillを駆動）
- DXプロジェクト確認 → 前提（テンプレ入力名 `Input:xxx` 等）を retrieve で確認（推測しない）→ authoring-bundle 生成 → `.agent` 記述（雛形 [assets/patterns/prompt-agent.agent](assets/patterns/prompt-agent.agent)／[assets/patterns/multi-topic-agent.agent](assets/patterns/multi-topic-agent.agent)）→ validate → deploy → preview。
- **手順・構文・CLI の詳細は `developing-agentforce` に従う**。独自の罠だけ [references/独自の落とし穴.md](references/独自の落とし穴.md)。
- 進捗は日本語で短く。専門ログは出さない。空回答は FunctionStep で自己診断（#11/#12/検索0件）。

### ⑤ 調整（人・Coding Agent壁打ち）
- できあがりの名前/概要と動作テスト結果を提示。直したい所を聞き、`.agent`/テンプレを修正→再 validate→再 deploy→再 preview。
- 直接プロンプトを触る方が速い場面もある。人がSF画面でやる／Coding Agentに指示する、どちらも可。詰まればCoding Agentが一緒に考える。
- 公開まで要望があれば確認の上 `publish`→`activate`。

## 参照ファイル

**進め方（我々の領域）**
- [references/環境セットアップ.md](references/環境セットアップ.md) — 初回の環境準備
- [references/前提チェック.md](references/前提チェック.md) — ①②（人の土台）の促し方・道具別チェック（入口A/Bとも必読）
- [references/質問テンプレ.md](references/質問テンプレ.md) — 入口別の日本語質問例

**設計の引き出し（紙化可）**
- [references/ユースケース集.md](references/ユースケース集.md) — 部門別カタログ＋道具の当て方
- [references/概念翻訳表.md](references/概念翻訳表.md) — SF用語→業務日本語

**組み立ての裏方**
- [references/独自の落とし穴.md](references/独自の落とし穴.md) — 公式に無い独自の罠＋必須ルール要点（詳細は公式へ）
- [assets/設計書-テンプレ.md](assets/設計書-テンプレ.md) — 設計書（2入口の合流点）
- [assets/patterns/prompt-agent.agent](assets/patterns/prompt-agent.agent) ／ [assets/patterns/multi-topic-agent.agent](assets/patterns/multi-topic-agent.agent) ／ [assets/patterns/retriever-prompt-template.genAiPromptTemplate-meta.xml](assets/patterns/retriever-prompt-template.genAiPromptTemplate-meta.xml)

**技術詳細（委譲）**
- 公式 `developing-agentforce` スキル — Agent Script 構文・全 CLI・権限/種別・ガードレール全般。
