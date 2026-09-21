# Defender XDR BEC タスク（日本語版）

[Azure/Azure-Sentinel](https://github.com/Azure/Azure-Sentinel/tree/master/Solutions/SentinelSOARessentials/Playbooks/Defender_XDR_BEC_Playbook_for_SecOps-Tasks) の `Defender_XDR_BEC_Playbook_for_SecOps-Tasks` を、統合ポータル（Microsoft Defender ポータル）で動くように修正したものです。

| 項目 | 値 |
| --- | --- |
| 積まれるタスク数 | **8** |
| 日本語化 | **完了**（メタデータ / タスク名 8 件 / 本文 8 件すべて） |
| 取り込み日 | 2026 年 9 月 15 日 |
| 検証 | ✅ 8 タスクの作成を確認済み。⚠️ 日本語表示は再デプロイ後に確認 |

## ファイル

| ファイル | 説明 |
| --- | --- |
| `azuredeploy.json` | **修正版**。こちらを使います |
| `azuredeploy.original.json` | 取り込み時点の本家。差分確認用 |

## 変更点

### 1. タスクが積まれない問題の修正

本家のままでは、統合ポータルで**実行は成功するのにタスクが 0 件**になります。

インシデントに紐づくアラート名に `BEC` が含まれるかを判定してからタスクを積む作りですが、統合ポータルのインシデントは `alerts` 配列が空で、さらに Playbook が大文字の `Alerts` を読んでいるため、判定が常に false になります。

**修正内容**: `Foreach` 内の `Scope_*` 5 個を top-level へ引き上げ

これでキーワード判定を素通りし、タスクが必ず積まれます。**どのインシデントに適用するかは、オートメーション ルールの条件側で絞ってください。**


### 2. 文字化けの修正

本家の定義ファイルには、記号が壊れた文字（U+FFFD）が **27 箇所**含まれています。Defender ポータルでは、箇条書きの先頭が `?` のような記号で表示されます。

**修正内容**: `<dt>` 内は `◆`、`<dd>` 内は `◇` に置き換え

### 3. 日本語化

テンプレートのタイトル、説明、デプロイ後の手順に加え、タスク名 8 件と本文 8 件を**すべて日本語にしました**。製品名（Microsoft Sentinel / Microsoft Defender XDR / Microsoft Defender for Office 365 / Threat Explorer など）、Advanced Hunting のテーブル名（CloudAppEvents、EmailEvents、UrlClickEvents、AuditLogs）、アクティビティ名（`New-InboxRule`、`Set-Mailbox` など）、URL は原文のまま残しています。

あわせて、本文中の閉じ忘れの `<dt>` を `</dt>` に修正しました。

| 本家 | 日本語版 |
| --- | --- |
| Introduction | はじめに |
| Contain | 封じ込め |
| Investigation - Step 1 | 調査 - ステップ 1 |
| Investigation - Step 2 | 調査 - ステップ 2 |
| Investigation - Step 3 | 調査 - ステップ 3 |
| Investigation - Step 4 | 調査 - ステップ 4 |
| Remediation | 修復 |
| Prevention | 予防 |

## 積まれるタスク

はじめに / 封じ込め / 調査 - ステップ 1〜4 / 修復 / 予防 = **8 件**

調査の 4 ステップは、ユーザー アカウントの初期侵害経路の特定、調査優先度スコアの確認、ユーザー アクティビティ（受信トレイ ルール、SMTP 転送、デバイス登録、MFA の追加など）の調査、ユーザーが送信したメールの調査という流れです。

## デプロイ

```bash
az deployment group create \
  --resource-group <リソース グループ> \
  --template-file azuredeploy.json \
  --parameters PlaybookName=<任意の名前>
```

デプロイ後、Logic App のシステム割り当てマネージド ID に `Microsoft Sentinel Responder` を付与します。**接続の手動認証は不要です。**

詳しい手順と注意点は[ソリューションの README](../../README.md) を参照してください。
