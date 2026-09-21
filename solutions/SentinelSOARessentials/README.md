# Sentinel SOAR Essentials（日本語版）

Content Hub の [Sentinel SOAR Essentials](https://github.com/Azure/Azure-Sentinel/tree/master/Solutions/SentinelSOARessentials) に含まれる**タスク化 Playbook 3 本**を、統合ポータル（Microsoft Defender ポータル）で動くように修正し、**3 本とも日本語化**したものです。

| 項目 | 値 |
| --- | --- |
| 取り込み元 | [Azure/Azure-Sentinel](https://github.com/Azure/Azure-Sentinel/tree/master/Solutions/SentinelSOARessentials) |
| パック バージョン | 3.0.8（2026 年 3 月 5 日更新） |
| 取り込み日 | 2026 年 9 月 15 日 |
| 検証環境 | Microsoft Sentinel をオンボード済みの Defender ポータル |

## 何が問題だったのか

3 本とも、**デプロイも実行も成功するのに、タスクが 1 つも作られません。**

実行履歴を開くと、すべてのアクションが `Skipped` になっています。原因は 2 つです。

### ① トリガー本文の `alerts` が空

Playbook は、インシデントに紐づくアラート名にキーワード（`Phish` など）が含まれるかを調べてからタスクを積みます。ところが統合ポータルのインシデントでは、この配列が空でした。

```
providerName              : Microsoft XDR
title                     : [SAMPLE ALERT] A user phishing attempt detected ...
additionalData.alertsCount: 0
alerts                    : []
```

検証環境のインシデント 50 件はすべて `providerName` が `Microsoft XDR` で、うち 37 件は `alertsCount` が 0 でした。**タイトルにフィッシングと書かれていても、アラート配列は空**です。

### ② プロパティ名の大文字小文字が合っていない

トリガー本文の実際のキーは小文字の `alerts` ですが、Playbook が読んでいるのは大文字の `Alerts` です。

```
@triggerBody()?['object']?['properties']?['Alerts']
```

Logic Apps の `?[...]` は存在しないキーを `null` として扱うため、**エラーにならず静かに素通り**します。結果として「実行は成功、タスクは 0 件」という、最も気づきにくい壊れ方をします。

### ③ タスク本文の文字化け

定義ファイルには、記号が壊れた文字（U+FFFD）が含まれています。箇条書きの先頭が `?` のような記号で表示されます。

| Playbook | 文字化け箇所 |
| --- | --- |
| Phishing | 48 |
| Ransomware | 70 |
| BEC | 27 |

`<dt>` 内は `◆`、`<dd>` 内は `◇` に置き換えました。

## 修正内容

**キーワード判定を通さず、常にタスクを積む**ようにしました。どのインシデントに適用するかは、**オートメーション ルールの条件側で絞ります**。

構造が 3 本で違うため、修正方法も 2 通りあります。

| Playbook | 構造 | 修正 |
| --- | --- | --- |
| Phishing | `If` > `Scope_*` | `If` の式を常に true へ |
| Ransomware | `If` > `Scope_*` | 同上 |
| **BEC** | `Foreach` > `If` > `Scope_*` | **`Scope_*` を top-level へ引き上げ** |

BEC だけタスクが `Foreach` の内側にあるため、式を変えるだけでは動きません。`Foreach` が 1 度も回らないからです。

## 日本語化

**3 本ともタスク名と本文をすべて日本語にしました。** Phishing はポータル上の日本語表示を確認済みです。Ransomware と BEC は JSON 構造・HTML・保護用語の機械検査まで完了しており、ポータルへの再デプロイ後に表示を確認します。

| Playbook | タスク名 | 本文 |
| --- | --- | --- |
| Phishing | 6 件 | 6 件 |
| Ransomware | 25 件 | 25 件 |
| BEC | 8 件 | 8 件 |

タスク名は 3 本で用語をそろえています。

| 本家 | 日本語版 |
| --- | --- |
| Introduction | はじめに |
| Contain / Containment | 封じ込め |
| Investigate / Investigation | 調査 |
| Eradication and recovery | 根絶と復旧 |
| Remediate / Remediation | 修復 |
| Prevent / Prevention | 予防 |

製品名（Microsoft Sentinel、Microsoft Defender XDR、Microsoft Defender for Endpoint など）、KQL、Advanced Hunting のテーブル名、検出名（IOA 名）、URL は原文のまま残しています。MITRE の戦術ラベルは日本語化しました。あわせて、本文に含まれていた壊れた HTML タグ（`<a/>`、閉じ忘れの `<dt>`）も修正しました。

## 収録 Playbook

| Playbook | タスク数 | 日本語化 | 文字化け修正 | 検証 |
| --- | --- | --- | --- | --- |
| [Phishing](Playbooks/Defender_XDR_Phishing_Playbook_for_SecOps-Tasks/) | 6 | ✅ | ✅ 48 箇所 | ✅ 6 件作成を確認 |
| [Ransomware](Playbooks/Defender_XDR_Ransomware_Playbook_for_SecOps-Tasks/) | 25 | ✅ | ✅ 70 箇所 | ⚠️ タスク作成25件は確認済み。日本語表示は再検証待ち |
| [BEC](Playbooks/Defender_XDR_BEC_Playbook_for_SecOps-Tasks/) | 8 | ✅ | ✅ 27 箇所 | ⚠️ タスク作成8件は確認済み。日本語表示は再検証待ち |

3 本とも**実機でタスクが作られるところまで確認**しています。

## 共通のデプロイ手順

```bash
az deployment group create \
  --resource-group <リソース グループ> \
  --template-file azuredeploy.json \
  --parameters PlaybookName=<任意の名前>
```

デプロイ後、Logic App の**システム割り当てマネージド ID** に `Microsoft Sentinel Responder` を付与します。

```bash
OID=$(az resource show -g <リソース グループ> -n <Playbook 名> \
  --resource-type "Microsoft.Logic/workflows" --query "identity.principalId" -o tsv)

az role assignment create --assignee-object-id $OID \
  --assignee-principal-type ServicePrincipal \
  --role "Microsoft Sentinel Responder" \
  --scope "/subscriptions/<サブスクリプション ID>/resourceGroups/<リソース グループ>"
```

**接続（API コネクション）の手動認証は不要です。** このテンプレートは Sentinel コネクタをマネージド ID 前提で定義しているため、デプロイした時点で認証が通ります。

## オートメーション ルールでの絞り込み

本家の README は「条件を `Incident provider` = `Microsoft Defender XDR` にする」と指示していますが、**統合ポータルではこのプロパティが削除されています**。代わりに次を使ってください。

| 条件 | 例 |
| --- | --- |
| タイトル | `Contains` → `Phish` |
| タグ | 事前にタグを付けておく |
| 分析ルール名 | Sentinel 由来のインシデントに限定したい場合 |

**条件を付けないと全インシデントにタスクが積まれます。** Logic Apps の課金は実行回数に比例するため、必ず絞ってください。

## 検証で分かったこと

- Logic App の実行が `Succeeded` でも、**中のアクションが `Skipped`** のことがあります。導入後は実行履歴をアクション単位で確認してください
- Sentinel のインシデント番号と Defender ポータルの番号は**一致しません**。`properties.additionalData.providerIncidentUrl` に正しい URL が入っています
- `Redirected` ラベルが付いたインシデントは、ポータルで別インシデントへ転送されます
