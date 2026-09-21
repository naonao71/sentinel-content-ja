# Sentinel SOAR Essentials（日本語版・共通タスク）

次の 2 種類の Microsoft Sentinel Playbook を収録しています。

| 種類 | 内容 |
| --- | --- |
| **公式 Playbook の日本語版** | Sentinel SOAR Essentials に含まれる Phishing / Ransomware / BEC のタスク化 Playbook を日本語化 |
| **コミュニティ Playbook** | すべてのインシデントで使える共通インシデント対応タスクを日本語で新規作成 |

## 公式 Playbook 3 本の日本語化

Content Hub の [Sentinel SOAR Essentials](https://github.com/Azure/Azure-Sentinel/tree/a4446e65c08206da39193ef30bdf714da9ab1dcd/Solutions/SentinelSOARessentials) に含まれるタスク化 Playbook 3 本を日本語化しました。

| 項目 | 値 |
| --- | --- |
| 取り込み元 | [Azure/Azure-Sentinel（固定コミット）](https://github.com/Azure/Azure-Sentinel/tree/a4446e65c08206da39193ef30bdf714da9ab1dcd/Solutions/SentinelSOARessentials) |
| パック バージョン | 3.0.8（2026 年 3 月 5 日更新） |
| 取り込み日 | 2026 年 9 月 15 日 |

アラートの抽出、キーワード判定、タスク作成 Scope の配置は**原版のロジックを維持**しています。

### 変更内容

- テンプレートのタイトル、説明、デプロイ後の手順を日本語化
- タスク名 39 件と本文 39 件を日本語化
- U+FFFD に置き換わっていた箇条書き記号を修正
- 壊れた HTML タグを修正

| Playbook | タスク名 | 本文 | 文字化け修正 |
| --- | ---: | ---: | ---: |
| Phishing | 6 件 | 6 件 | 48 箇所 |
| Ransomware | 25 件 | 25 件 | 70 箇所 |
| BEC | 8 件 | 8 件 | 27 箇所 |

`<dt>` 内は `◆`、`<dd>` 内は `◇` に置き換えました。

### 用語

| 本家 | 日本語版 |
| --- | --- |
| Introduction | はじめに |
| Contain / Containment | 封じ込め |
| Investigate / Investigation | 調査 |
| Eradication and recovery | 根絶と復旧 |
| Remediate / Remediation | 修復 |
| Prevent / Prevention | 予防 |

製品名、KQL、Advanced Hunting のテーブル名、検出名、アクティビティ名、URL は原文のまま残しています。MITRE の戦術ラベルは日本語化しました。

## 共通インシデント対応タスク

[`Defender-XDR-Generic-Incident-Tasks-JA`](https://github.com/naonao71/sentinel-content-ja/tree/main/playbooks/defender-xdr-generic-incident-tasks-ja) は、Defender ポータルのインシデント画面から必要なときにオンデマンド実行するコミュニティ Playbook です。

Microsoft Learn のインシデント対応ワークフローをもとに、次の 6 件を追加します。

1. `[共通] はじめに`
2. `[共通] トリアージ`
3. `[共通] 調査と分析`
4. `[共通] 封じ込め`
5. `[共通] 回復`
6. `[共通] 解決と文書化`

Phishing / Ransomware / BEC の詳細タスクを置き換えるものではありません。共通タスクを対応の土台とし、該当する脅威には公式 3 本の日本語版を追加します。

## 収録 Playbook

| Playbook | 種類 | タスク数 | 状態 |
| --- | --- | ---: | --- |
| [Phishing](https://github.com/naonao71/sentinel-content-ja/tree/main/solutions/SentinelSOARessentials/Playbooks/Defender_XDR_Phishing_Playbook_for_SecOps-Tasks) | 公式日本語版 | 6 | ⚠️ 対象アラートが紐づいた実インシデントで要確認 |
| [Ransomware](https://github.com/naonao71/sentinel-content-ja/tree/main/solutions/SentinelSOARessentials/Playbooks/Defender_XDR_Ransomware_Playbook_for_SecOps-Tasks) | 公式日本語版 | 25 | ⚠️ 対象アラートが紐づいた実インシデントで要確認 |
| [BEC](https://github.com/naonao71/sentinel-content-ja/tree/main/solutions/SentinelSOARessentials/Playbooks/Defender_XDR_BEC_Playbook_for_SecOps-Tasks) | 公式日本語版 | 8 | ⚠️ 対象アラートが紐づいた実インシデントで要確認 |
| [共通インシデント対応](https://github.com/naonao71/sentinel-content-ja/tree/main/playbooks/defender-xdr-generic-incident-tasks-ja) | コミュニティ | 6 | ✅ インシデント画面からのオンデマンド実行を確認 |

## デプロイ

各 Playbook のフォルダーで実行します。

```bash
az deployment group create \
  --resource-group <リソース グループ> \
  --template-file azuredeploy.json \
  --parameters PlaybookName=<Playbook 名>
```

テンプレートは Microsoft Sentinel コネクタをマネージド ID 前提で作成するため、API 接続の手動認証は不要です。

### Logic App のマネージド ID

Logic App のシステム割り当てマネージド ID に `Microsoft Sentinel Responder` を付与します。

```bash
OID=$(az resource show \
  --resource-group <リソース グループ> \
  --name <Playbook 名> \
  --resource-type "Microsoft.Logic/workflows" \
  --query "identity.principalId" \
  --output tsv)

az role assignment create \
  --assignee-object-id "$OID" \
  --assignee-principal-type ServicePrincipal \
  --role "Microsoft Sentinel Responder" \
  --scope "/subscriptions/<サブスクリプション ID>/resourceGroups/<Microsoft Sentinel のリソース グループ>"
```

### インシデント画面からオンデマンド実行する場合

共通インシデント対応 Playbook の実行には、次の権限も必要です。

| 対象 | ロール | スコープ |
| --- | --- | --- |
| 実行者 | Microsoft Sentinel Responder | Microsoft Sentinel のリソース グループ |
| 実行者 | Microsoft Sentinel Playbook Operator | Playbook のリソース グループ |
| Microsoft Sentinel | Microsoft Sentinel Automation Contributor | Playbook のリソース グループ |

Defender ポータルのインシデント画面で、Playbook 一覧の右端にある **[プレイブックを実行する]** を選択します。左側の Playbook 名は Logic Apps の設定画面を開くリンクであり、対象インシデントに対する実行ではありません。

## 公式 3 本を自動実行する場合

本家の README は、オートメーション ルールの条件として `Incident provider` = `Microsoft Defender XDR` を指定していますが、統合ポータルではこのプロパティが表示されません。

Playbook 内部にもアラート名による判定がありますが、不要な Logic Apps 実行を減らすため、環境で利用できるタイトル、タグ、分析ルール名などの条件で対象を絞ってください。

## 利用上の注意

- Logic App の実行が `Succeeded` でも、内部アクションが `Skipped` の場合があります。期待したタスクが作成されたことまで確認してください
- 公式 3 本は、対象アラートが紐づいた実インシデントで動作を確認してください
- 共通インシデント対応 Playbook を同じインシデントで再実行すると、同じタスクが再度作成されます
- Sentinel のインシデント番号と Defender ポータルの番号は一致しない場合があります
