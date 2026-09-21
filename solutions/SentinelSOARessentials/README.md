# Sentinel SOAR Essentials（日本語版・共通タスク）

Content Hub の [Sentinel SOAR Essentials](https://github.com/Azure/Azure-Sentinel/tree/master/Solutions/SentinelSOAREssentials) に含まれる**タスク化 Playbook 3 本**の日本語版と、すべてのインシデントで使える**共通インシデント対応タスク**を収録しています。公式 3 本は、アラートの抽出、キーワード判定、タスク作成 Scope の配置について**原版のロジックを維持**しています。

| 項目 | 値 |
| --- | --- |
| 取り込み元 | [Azure/Azure-Sentinel](https://github.com/Azure/Azure-Sentinel/tree/master/Solutions/SentinelSOARessentials) |
| パック バージョン | 3.0.8（2026 年 3 月 5 日更新） |
| 取り込み日 | 2026 年 9 月 15 日 |
| 検証環境 | Microsoft Sentinel をオンボード済みの Defender ポータル |
| 検証上の制約 | サンプル インシデントを使用。対象キーワードを含む実アラートが紐づいた Defender XDR インシデントは未検証 |

## サンプル インシデント検証の読み直し

ラボのサンプル インシデントで実行したところ、Logic App は成功しましたが、タスクは作成されませんでした。

実行履歴ではタスク作成アクションが `Skipped` でした。確認したサンプルの一部は、次のようにアラート配列が空でした。

```
providerName              : Microsoft XDR
title                     : [SAMPLE ALERT] A user phishing attempt detected ...
additionalData.alertsCount: 0
alerts                    : []
```

検証環境のインシデント 50 件はすべて `providerName` が `Microsoft XDR` で、うち 37 件は `alertsCount` が 0 でした。ただし、これは**サンプル インシデントで得た結果**です。

3 本は、それぞれ実際の製品アラートを起点に、インシデント内のアラート名を確認する設計です。

| Playbook | 主な検知元 | 判定キーワード |
| --- | --- | --- |
| Phishing | MDO（確認例のサービス ソースは Office 365） | `Phish` / `ZAP` / `removed after delivery` / `URL click was detected` |
| Ransomware | Defender XDR のランサムウェア関連アラート（確認例は Defender XDR と MDE の相関） | `Ransomware` / `ransomware` |
| BEC | Defender XDR の BEC 関連アラート（確認例のサービス ソースは Microsoft Defender for Cloud Apps） | `BEC` |

したがって、今回の結果から言えるのは、**サンプル インシデントでは原版の判定条件を満たさなかった**ことまでです。対象キーワードを含む実アラートが紐づいた Defender XDR インシデントでも動かないとは判断できません。

### ③ タスク本文の文字化け

定義ファイルには、記号が壊れた文字（U+FFFD）が含まれています。箇条書きの先頭が `?` のような記号で表示されます。

| Playbook | 文字化け箇所 |
| --- | --- |
| Phishing | 48 |
| Ransomware | 70 |
| BEC | 27 |

`<dt>` 内は `◆`、`<dd>` 内は `◇` に置き換えました。

## 変更内容

- テンプレートのタイトル、説明、デプロイ後の手順を日本語化
- タスク名と本文を日本語化
- U+FFFD に置き換わっていた箇条書き記号を修正
- 壊れた HTML タグを修正
- 原版のアラート抽出、キーワード判定、Scope 構造を維持

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
| [Phishing](Playbooks/Defender_XDR_Phishing_Playbook_for_SecOps-Tasks/) | 6 | ✅ | ✅ 48 箇所 | ⚠️ 実際の MDO 由来インシデントで要検証 |
| [Ransomware](Playbooks/Defender_XDR_Ransomware_Playbook_for_SecOps-Tasks/) | 25 | ✅ | ✅ 70 箇所 | ⚠️ `Ransomware` を含む実アラートが紐づいたインシデントで要検証 |
| [BEC](Playbooks/Defender_XDR_BEC_Playbook_for_SecOps-Tasks/) | 8 | ✅ | ✅ 27 箇所 | ⚠️ `BEC` を含む実アラートが紐づいたインシデントで要検証 |
| [共通インシデント対応](Playbooks/Defender-XDR-Generic-Incident-Tasks-JA/) | 6 | 日本語で新規作成 | — | ⚠️ デプロイ・手動実行は未検証 |

条件を一時的に迂回した検証では、タスク作成アクション自体が 6 / 25 / 8 件を作成できることを確認しました。ただし、これは**原版の判定ロジックが実際の製品由来インシデントで成立することの確認ではありません**。

## 共通インシデント対応タスク

`Defender-XDR-Generic-Incident-Tasks-JA` は、Defender ポータルのインシデント画面から必要なときに手動実行するコミュニティ Playbook です。

- `[共通] はじめに`
- `[共通] トリアージ`
- `[共通] 調査と分析`
- `[共通] 封じ込め`
- `[共通] 回復`
- `[共通] 解決と文書化`

Phishing / Ransomware / BEC の詳細タスクを置き換えるものではありません。共通タスクを対応の土台とし、該当する脅威には公式 3 本の日本語版を追加します。

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

本家の README は「条件を `Incident provider` = `Microsoft Defender XDR` にする」と指示していますが、**統合ポータルではこのプロパティが表示されません**。必要に応じて、環境で利用できる次の条件を追加してください。

| 条件 | 例 |
| --- | --- |
| タイトル | `Contains` → `Phish` |
| タグ | 事前にタグを付けておく |
| 分析ルール名 | Sentinel 由来のインシデントに限定したい場合 |

Playbook 内部にもアラート名による判定がありますが、不要な Logic Apps 実行を減らすため、オートメーション ルール側でも対象を絞ることを推奨します。

## 検証で分かったこと

- Logic App の実行が `Succeeded` でも、**中のアクションが `Skipped`** のことがあります。導入後は実行履歴をアクション単位で確認してください
- サンプル インシデントは、実際の製品アラートが紐づいた Defender XDR インシデントと同じアラート情報を持つとは限りません
- 原版ロジックの可否は、対象製品から実際に生成されたインシデントで確認してください
- Sentinel のインシデント番号と Defender ポータルの番号は**一致しません**。`properties.additionalData.providerIncidentUrl` に正しい URL が入っています
- `Redirected` ラベルが付いたインシデントは、ポータルで別インシデントへ転送されます
