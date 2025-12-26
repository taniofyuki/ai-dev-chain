# AI制御・実行モデル設計

本章では、AI開発自動化システムにおけるAIの制御方式、実行モデル、役割分担、および運用ルールを定義する。

---

## 1. 実行ライフサイクル

### 1.1 フェーズ実行の全体像

```
フェーズ開始
   ↓
Planner による計画生成
   ↓
Doc Writer / Code Writer による成果物生成
   ↓
（オプション）Reviewer による多角的レビュー
   ↓
Checker による整合性チェック
   ↓
判定により分岐
   ├─ WARN/INFOのみ → PR作成 → レビュー
   └─ BLOCKあり → 自動修正ループ
                    ├─ 修正
                    ├─ 再チェック
                    └─ 3回失敗 → Blocked＋通知
```

### 1.2 自動修正ループの詳細

- BLOCK の原因箇所のみを対象に修正を行う
- 1フェーズあたり最大3回まで再試行
- 各修正内容・判断は必ず report に記録

---

## 2. AI入力管理モデル

### 2.1 論理入力スコープ
AIは常に以下を論理的に保持する。

- /docs/spec（正本）
- /docs/reports（履歴）
- フェーズテンプレ
- 直前差分
- ユーザー指示

### 2.2 実装入力戦略

- spec：構造要約＋必要部分全文
- reports：要約
- 差分：全文
- テンプレ：全文
- 必要時オンデマンド取得

---

## 3. AIロール設計

### 3.1 標準ロール
- Planner：計画作成
- Doc Writer：ドキュメント生成
- Code Writer：実装生成
- Checker：整合性チェック

### 3.2 オプションロール
- Reviewer：多角的レビュー


### 3.3 ロール詳細

#### Planner

- 目的：作業計画と判断材料の構造化
- 出力：対象ファイル、作業手順、未決事項、ゴール定義

#### Doc Writer

- 目的：仕様・設計・レポートの生成
- 出力：/docs 配下の成果物更新案

#### Code Writer

- 目的：アプリケーション実装
- 出力：/src /config の更新案

#### Reviewer（任意）

- 目的：多角的観点による妥当性評価
- 出力：WARN/INFO 指摘

#### Checker

- 目的：品質ゲート
- 出力：BLOCK/WARN/INFO 判定と修正指示

---

### 3.4 実行順
Planner → Writer → Reviewer → Checker → 修正ループ

---

## 4. 作業パケット

### 4.1 パケット構造

#### 基本項目

- phase_id（例: 07）
- phase_name_ja（例: 詳細設計）
- goal（このフェーズで達成したいこと）

#### inputs

- spec_summary
- recent_reports_summary
- templates（該当テンプレ全文）
- diff_context（直前差分）
- user_instructions

#### plan（Plannerが作成）

- target_files（更新対象ファイル）
- tasks（作業手順）
- open_questions（要判断＝WARN候補）

#### changes（Writerが作成）

- file_updates[]（パス＋変更内容）

#### check_result（Checkerが作成）

- items[]（BLOCK / WARN / INFO、理由、対象、修正方針）

### 4.2 パケット格納場所

```
.workflow/runs/<run_id>/
  packet.json
  logs/
```

- packetはGitに含めない

---

## 5. run_id 管理

- 採番：```<project>-<phase>-<YYYYMMDD-HHMMSS>```
- 実行単位で一意

---

## 6. 成果物反映方式

- 作業ブランチで直接反映
- 成功後PR作成

---


## 7. フェーズ開始トリガ

- デフォルト：手動
- 設定で連続実行に切替可能

---

## 8. プロンプト管理

### 8.1 管理対象

- システムプロンプトのみ

### 8.2 管理方式

```
/prompts/{role}/system.md
/prompts/{role}/user.md
```

### 8.3 再現性保証

- 使用プロンプトはGit管理し、ハッシュを packet.json に記録して再現性を保証する
