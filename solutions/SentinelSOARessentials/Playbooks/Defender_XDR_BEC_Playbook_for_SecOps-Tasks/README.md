# Defender XDR BEC タスク（統合ポータル対応版）

[Azure/Azure-Sentinel](https://github.com/Azure/Azure-Sentinel/tree/master/Solutions/SentinelSOARessentials/Playbooks/Defender_XDR_BEC_Playbook_for_SecOps-Tasks) の `Defender_XDR_BEC_Playbook_for_SecOps-Tasks` を、統合ポータル（Microsoft Defender ポータル）で動くように修正したものです。

| 項目 | 値 |
| --- | --- |
| 積まれるタスク数 | **8** |
| 取り込み日 | 2026 年 9 月 15 日 |
| 検証 | ✅ 8 タスクの作成を確認 |

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

## 積まれるタスク

Introduction / Contain / Investigation Step 1〜4 / Remediation / Prevention

## デプロイ

```bash
az deployment group create \
  --resource-group <リソース グループ> \
  --template-file azuredeploy.json \
  --parameters PlaybookName=<任意の名前>
```

デプロイ後、Logic App のシステム割り当てマネージド ID に `Microsoft Sentinel Responder` を付与します。**接続の手動認証は不要です。**

詳しい手順と注意点は[ソリューションの README](../../README.md) を参照してください。
