# Defender XDR Ransomware タスク（日本語版）

[Azure/Azure-Sentinel](https://github.com/Azure/Azure-Sentinel/tree/a4446e65c08206da39193ef30bdf714da9ab1dcd/Solutions/SentinelSOARessentials/Playbooks/Defender_XDR_Ransomware_Playbook_for_SecOps-Tasks) の `Defender_XDR_Ransomware_Playbook_for_SecOps-Tasks` を日本語化し、文字化けと HTML を修正したものです。アラート判定ロジックは原版を維持しています。

| 項目 | 値 |
| --- | --- |
| 積まれるタスク数 | **25** |
| 日本語化 | **完了**（メタデータ / タスク名 25 件 / 本文 25 件すべて） |
| 取り込み日 | 2026 年 9 月 15 日 |
| 検証 | ⚠️ サンプルでは判定条件を満たさず。`Ransomware` を含む実アラートが紐づいたインシデントで要検証 |

## ファイル

| ファイル | 説明 |
| --- | --- |
| `azuredeploy.json` | **修正版**。こちらを使います |
| `azuredeploy.original.json` | 取り込み時点の本家。差分確認用 |

## 変更点

### 1. 原版の判定ロジックを維持

原版は、インシデントに紐づくアラート名に `Ransomware` / `ransomware` が含まれる場合にタスクを作成します。

ラボのサンプル インシデントではアラート配列が空で、タスクは 0 件でした。しかし、サンプルは実際のランサムウェア関連アラートが紐づいたインシデントと同じ入力を保証しません。今回確認した `Azure VM extension activity followed by ransomware or hands-on-keyboard attack` の実例では、サービス ソースが Microsoft Defender XDR のアラートと、Microsoft Defender for Endpoint の関連アラートが同じインシデントで相関されていました。

この日本語版では条件式を変更していません。`Ransomware` を含む実アラートが紐づいた Defender XDR インシデントで検証してから本番利用してください。


### 2. 文字化けの修正

本家の定義ファイルには、記号が壊れた文字（U+FFFD）が **70 箇所**含まれています。Defender ポータルでは、箇条書きの先頭が `?` のような記号で表示されます。

**修正内容**: `<dt>` 内は `◆`、`<dd>` 内は `◇` に置き換え

### 3. 日本語化

テンプレートのタイトル、説明、デプロイ後の手順に加え、タスク名 25 件と本文 25 件を**すべて日本語にしました**。製品名（Microsoft Sentinel / Microsoft Defender XDR / Microsoft Defender Antivirus など）、KQL、検出名（IOA 名）、URL は原文のまま残しています。MITRE の戦術ラベルは、初期アクセス、実行、防御回避、横展開、権限昇格、影響に統一しました。

あわせて、本文中の壊れた HTML タグ（`<a/>` → `</a>`、閉じ忘れの `<dt>` → `</dt>`）を修正しました。

| 本家 | 日本語版 |
| --- | --- |
| Introduction | はじめに |
| Containment - Step 1: Assess the scope of the incident | 封じ込め - ステップ 1: インシデントの影響範囲を評価する |
| Containment - Step 2: Preserve existing systems | 封じ込め - ステップ 2: 既存システムを保全する |
| Containment - Step 3.1 / 3.2: Prevent the spread | 封じ込め - ステップ 3.1 / 3.2: 拡散を防止する |
| Investigation - Assess the current situation | 調査 - 現在の状況を評価する |
| Investigation - Identify the ransomware process | 調査 - ランサムウェアのプロセスを特定する |
| Investigation - Look for exposed credentials ... | 調査 - 感染したデバイスで露出した資格情報を探す |
| Investigate - Identify the line of business (LOB) apps ... | 調査 - インシデントにより利用できなくなった基幹業務 (LOB) アプリを特定する |
| Eradication and recovery - Step 1〜9 | 根絶と復旧 - ステップ 1〜9 |
| More data about Human-operated ransomware | 人間が操作するランサムウェアに関する詳細情報 |
| Prevention - Device protection: Part 1 / Part 2 | 予防 - デバイス保護: パート 1 / パート 2 |
| Prevention - Email management | 予防 - メール管理 |
| Prevention - Identity protection | 予防 - ID 保護 |
| Prevention - Information protection | 予防 - 情報保護 |
| Prevention - Vulnerability management | 予防 - 脆弱性管理 |

## 積まれるタスク

はじめに 1 件 / 封じ込め 4 件 / 調査 4 件 / 根絶と復旧 9 件 / 人間が操作するランサムウェアに関する詳細情報 1 件 / 予防 6 件 = **25 件**

根絶と復旧の 9 ステップは次のとおりです。

1. バックアップを確認する
2. インジケーターを追加する
3. 侵害されたユーザーをリセットする
4. 攻撃者の制御ポイントを隔離する
5. マルウェアを駆除する
6. クリーンアップ済みデバイスのファイルを復旧する
7. OneDrive for Business のファイルを復旧する
8. 削除されたメールを復旧する
9. Exchange ActiveSync と OneDrive 同期を再度有効にする

## デプロイ

```bash
az deployment group create \
  --resource-group <リソース グループ> \
  --template-file azuredeploy.json \
  --parameters PlaybookName=<任意の名前>
```

デプロイ後、Logic App のシステム割り当てマネージド ID に `Microsoft Sentinel Responder` を付与します。**接続の手動認証は不要です。**

詳しい手順と注意点は[ソリューションの README](../../README.md) を参照してください。

## 元になっている手順書

[人間が操作するランサムウェア](https://learn.microsoft.com/security/ransomware/human-operated-ransomware)
