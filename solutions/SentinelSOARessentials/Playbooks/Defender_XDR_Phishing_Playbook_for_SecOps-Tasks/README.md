# Defender XDR Phishing タスク（日本語版）

[Azure/Azure-Sentinel](https://github.com/Azure/Azure-Sentinel/tree/master/Solutions/SentinelSOARessentials/Playbooks/Defender_XDR_Phishing_Playbook_for_SecOps-Tasks) の `Defender_XDR_Phishing_Playbook_for_SecOps-Tasks` を、統合ポータル（Microsoft Defender ポータル）で動くように修正したものです。

| 項目 | 値 |
| --- | --- |
| 積まれるタスク数 | **6** |
| 取り込み日 | 2026 年 9 月 15 日 |
| 検証 | ✅ フィッシング系インシデントで 6 タスクの作成と日本語表示を確認 |

## ファイル

| ファイル | 説明 |
| --- | --- |
| `azuredeploy.json` | **修正版**。こちらを使います |
| `azuredeploy.original.json` | 取り込み時点の本家。差分確認用 |

## 変更点

### 1. タスクが積まれない問題の修正

本家のままでは、統合ポータルで**実行は成功するのにタスクが 0 件**になります。

インシデントに紐づくアラート名に `Phish` / `ZAP` / `removed after delivery` / `URL click was detected` が含まれるかを判定してからタスクを積む作りですが、統合ポータルのインシデントは `alerts` 配列が空で、さらに Playbook が大文字の `Alerts` を読んでいるため、判定が常に false になります。

**修正内容**: top-level の `Condition` の式を常に true へ変更

これでキーワード判定を素通りし、タスクが必ず積まれます。**どのインシデントに適用するかは、オートメーション ルールの条件側で絞ってください。**

### 2. 文字化けの修正

本家の定義ファイルには、記号が壊れた文字（U+FFFD）が **48 箇所**含まれています。日本語環境の Defender ポータルでは、箇条書きの先頭が `◆` ではなく `?` のような記号で表示されます。

**修正内容**: 壊れた文字を `◆` `◇` に置き換え

### 3. 日本語化

タスク名と本文を日本語にしました。埋め込まれている Microsoft Learn へのリンクは、日本語ページがあるものは日本語版を指しています。

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
