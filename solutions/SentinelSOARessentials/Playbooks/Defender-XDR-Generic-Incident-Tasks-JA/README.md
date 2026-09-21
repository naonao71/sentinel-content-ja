# Defender XDR 共通インシデント対応タスク（日本語）

Microsoft Defender ポータルのインシデント画面から手動実行し、脅威の種類を問わず利用できる共通チェックリストを追加する Microsoft Sentinel Playbook です。

| 項目 | 値 |
| --- | --- |
| Logic App 名 | `Defender-XDR-Generic-Incident-Tasks-JA` |
| トリガー | Microsoft Sentinel インシデント |
| 想定する実行方法 | Defender ポータルのインシデント画面から手動実行 |
| 作成するタスク | **6 件** |
| 実行順序 | 逐次実行（同時実行数 1） |
| 認証 | システム割り当てマネージド ID |

## 位置づけ

公式の Phishing / Ransomware / BEC Playbook は、インシデント内のアラート名を判定し、該当する脅威の詳細手順を追加します。

この Playbook は、その 3 シナリオに該当しないインシデントも含めて利用できる**共通の対応骨格**です。Microsoft Learn の以下の公開手順を、インシデント タスクの形へ読み替えています。

- [Microsoft Defender ポータルでインシデント対応ワークフローを計画する](https://learn.microsoft.com/ja-jp/defender-xdr/plan-incident-response)
- [Microsoft Defender でインシデントを管理する](https://learn.microsoft.com/ja-jp/defender-xdr/manage-incidents)
- [Microsoft Defender ポータルでインシデントを調査する](https://learn.microsoft.com/ja-jp/defender-xdr/investigate-incidents)

⚠️ Microsoft Learn はインシデント対応の進め方を示すもので、タスク化を指示しているわけではありません。本 Playbook はコミュニティ実装です。

## 作成するタスク

| # | タスク名 | 内容 |
| --- | --- | --- |
| 1 | `[共通] はじめに` | 対応の流れと、脅威別タスクとの関係を確認。作成後に自動完了 |
| 2 | `[共通] トリアージ` | 優先度、担当者、状態、タグを確認 |
| 3 | `[共通] 調査と分析` | 攻撃ストーリー、アラート、資産、自動調査、証拠を確認 |
| 4 | `[共通] 封じ込め` | ユーザー無効化、デバイス分離、IP ブロック、自動攻撃の中断を確認 |
| 5 | `[共通] 回復` | 資産・構成・業務を段階的に復旧 |
| 6 | `[共通] 解決と文書化` | 分類、解決メモ、コメント、再発防止への反映 |

タスク本文は、公式のタスク化 Playbook と同じ `<dt><b>◆ 問い</b></dt>` / `<dd>◇ 補足</dd>` 形式です。

## デプロイ

```bash
az deployment group create \
  --resource-group <リソース グループ> \
  --template-file azuredeploy.json \
  --parameters PlaybookName=Defender-XDR-Generic-Incident-Tasks-JA
```

## 必要な権限

### Logic App のマネージド ID

タスクを作成するため、Logic App のシステム割り当てマネージド ID に `Microsoft Sentinel Responder` を付与します。

```bash
OID=$(az resource show \
  --resource-group <リソース グループ> \
  --name Defender-XDR-Generic-Incident-Tasks-JA \
  --resource-type "Microsoft.Logic/workflows" \
  --query "identity.principalId" \
  --output tsv)

SCOPE=$(az group show \
  --name <Microsoft Sentinel ワークスペースのリソース グループ> \
  --query id \
  --output tsv)

az role assignment create \
  --assignee-object-id "$OID" \
  --assignee-principal-type ServicePrincipal \
  --role "Microsoft Sentinel Responder" \
  --scope "$SCOPE"
```

### Microsoft Sentinel と実行者

- Microsoft Sentinel のサービス アカウント: Playbook のリソース グループに `Microsoft Sentinel Automation Contributor`
- 実行者: インシデントへの `Microsoft Sentinel Responder`
- 実行者: Playbook のリソース グループへの `Microsoft Sentinel Playbook Operator`

詳細は [Microsoft Sentinel の Playbook を使用して脅威対応を自動化する](https://learn.microsoft.com/ja-jp/azure/sentinel/automate-responses-with-playbooks#prerequisites) を参照してください。

## インシデント画面から実行する

1. Microsoft Defender ポータルで **[インシデントとアラート] > [インシデント]** を開きます。
2. 対象インシデントを選択します。
3. 詳細ペインから **[Playbook の実行]** を選択します。
4. `Defender-XDR-Generic-Incident-Tasks-JA` を選択して実行します。
5. インシデントのタスク パネルで、6 件が作成されたことを確認します。

## 注意事項

### 再実行するとタスクが重複する

この Playbook は既存タスクの有無を確認しません。同じインシデントで再実行すると、同じ 6 件がもう一度作成されます。

運用では次のいずれかを採用してください。

- インシデントのコメントへ実行記録を残す
- 実行済みタグを付ける
- 再実行前にタスク パネルを確認する

### 脅威別 Playbook との併用

Phishing / Ransomware / BEC に該当する場合、共通 6 件に加えて脅威別タスクも作成できます。共通タスクを土台、脅威別タスクを詳細手順として扱います。

### 自動実行も可能

インシデント トリガーを使用しているため、オートメーション ルールからも実行できます。ただし、すべてのインシデントへ自動適用すると Logic Apps の実行回数とタスク数が増えるため、本版は手動実行を推奨します。
