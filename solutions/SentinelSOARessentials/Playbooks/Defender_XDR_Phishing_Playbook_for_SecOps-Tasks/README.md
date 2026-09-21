# Defender XDR Phishing タスク（日本語版）

[Azure/Azure-Sentinel](https://github.com/Azure/Azure-Sentinel/tree/master/Solutions/SentinelSOAREssentials/Playbooks/Defender_XDR_Phishing_Playbook_for_SecOps-Tasks) の `Defender_XDR_Phishing_Playbook_for_SecOps-Tasks` を日本語化し、文字化けと HTML を修正したものです。アラート判定ロジックは原版を維持しています。

| 項目 | 値 |
| --- | --- |
| 積まれるタスク数 | **6** |
| 日本語化 | **完了**（メタデータ / タスク名 6 件 / 本文 6 件すべて） |
| 取り込み日 | 2026 年 9 月 15 日 |
| 検証 | ⚠️ MDO の実アラート例を確認済み。Playbook の条件成立と6タスク作成は要検証 |

## ファイル

| ファイル | 説明 |
| --- | --- |
| `azuredeploy.json` | **修正版**。こちらを使います |
| `azuredeploy.original.json` | 取り込み時点の本家。差分確認用 |

## 変更点

### 1. 原版の判定ロジックを維持

原版は、インシデントに紐づくアラート名に `Phish` / `ZAP` / `removed after delivery` / `URL click was detected` が含まれる場合にタスクを作成します。

ラボのサンプル インシデントではアラート配列が空で、タスクは 0 件でした。しかし、サンプルは実際の MDO アラートが紐づいたインシデントと同じ入力を保証しません。実アラート `Email reported by user as malware or phish` では、検出ソースが MDO、サービス ソースが Office 365 であることを確認しました。

この日本語版では条件式を変更していません。実際の MDO 由来インシデントで検証してから本番利用してください。

### 2. 文字化けの修正

本家の定義ファイルには、記号が壊れた文字（U+FFFD）が **48 箇所**含まれています。日本語環境の Defender ポータルでは、箇条書きの先頭が `◆` ではなく `?` のような記号で表示されます。

**修正内容**: 壊れた文字を `◆` `◇` に置き換え

### 3. 日本語化

テンプレートのタイトル、説明、デプロイ後の手順に加え、タスク名 6 件と本文 6 件を日本語にしました。埋め込まれている Microsoft Learn へのリンクは、日本語ページがあるものは日本語版を指しています。

| 本家 | 日本語版 |
| --- | --- |
| Introduction | はじめに |
| Contain | 封じ込め |
| Investigate | 調査 |
| Investigate involved users | 関係ユーザーの調査 |
| Remediate | 修復 |
| Prevent | 予防 |


## 積まれるタスク

はじめに / 封じ込め / 調査 / 関係ユーザーの調査 / 修復 / 予防

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

[インシデント対応 Playbook：フィッシング](https://learn.microsoft.com/ja-jp/security/operations/incident-response-playbook-phishing)
