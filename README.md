# sentinel-content-ja

Microsoft Sentinel のコンテンツパックを、原版の判定ロジックを保ったまま**日本語化**し、文字化けや壊れた HTML を修正したものを置いています。

日本語化した JSON は、原版との構造比較、HTML、保護用語を機械検査しています。ラボで使用したサンプル インシデントではアラート配列が空だったため、対象キーワードを含む実アラートが紐づいた Defender XDR インシデントに対する原版の判定ロジックは未検証です。

## なぜこのリポジトリがあるのか

Content Hub のコンテンツパックには、日本語環境で使う前に確認したいものがあります。

- 本文が英語のまま
- 日本語環境で箇条書きが文字化けする
- README の手順が、現在の画面に存在しない設定を指している
- サンプル インシデントと実際の製品由来インシデントで入力データが異なる

ここでは、原版の処理条件を変更せずに日本語化と表示上の修正を加えたものと、公開されている Microsoft Learn をもとにしたコミュニティ Playbook を公開しています。

## 収録内容

| ソリューション | 状態 | 内容 |
| --- | --- | --- |
| [SentinelSOARessentials](solutions/SentinelSOARessentials/) | ⚠️ 実インシデントで要検証 | タスク化 Playbook 3 本（Phishing / Ransomware / BEC）の日本語版と、手動実行用の**共通インシデント対応タスク** |
| [Defender XDR 共通インシデント対応](playbooks/defender-xdr-generic-incident-tasks-ja/) | ✅ 実機確認済み | インシデント画面からオンデマンド実行し、共通タスク 6 件を追加するコミュニティ Playbook |

## 構成

本家 [Azure/Azure-Sentinel](https://github.com/Azure/Azure-Sentinel) と**同じディレクトリ構成**にしています。元ファイルとの差分が追いやすく、必要なら本家へ PR を出せる形です。

```
solutions/
└── <ソリューション名>/
    ├── README.md                  ← 修正の一覧と検証結果
    └── Playbooks/
        └── <Playbook 名>/
            ├── azuredeploy.json          ← デプロイするテンプレート
            ├── azuredeploy.original.json ← 本家由来の場合のみ。取り込み時点の比較用
            └── README.md                 ← 変更点とデプロイ手順

playbooks/
└── <コミュニティ Playbook 名>/
    ├── azuredeploy.json
    └── README.md
```

本家由来の Playbook では、更新時に差分を取れるよう `azuredeploy.original.json` を残しています。コミュニティ Playbook には原版ファイルはありません。

## 使い方

1. 使いたい Playbook のフォルダを開く
2. `README.md` で変更点を確認する
3. `azuredeploy.json` をダウンロードして `az deployment group create` でデプロイする
4. Logic App の**システム割り当てマネージド ID** に必要なロールを付与する

詳しい手順は各 Playbook の README に書いています。

## 本家との関係

- 元は [Azure/Azure-Sentinel](https://github.com/Azure/Azure-Sentinel)（MIT ライセンス）です
- 各ファイルの取り込み日とバージョンは、ソリューションごとの README に記載しています
- **本家の更新には自動で追随しません。** 取り込み時点の内容に対する日本語化です

## ライセンス

[MIT License](LICENSE)。元のコンテンツの著作権は Microsoft Corporation に帰属します。改変した箇所は各 README に明記しています。

## 免責

本リポジトリは**個人の検証に基づくもの**であり、所属する組織の公式見解や公式サポートを示すものではありません。利用にあたっては、必ずご自身の環境で動作を確認してください。

製品仕様は変更されるため、最新情報は [Microsoft Learn](https://learn.microsoft.com/azure/sentinel/) でご確認ください。

## 関連記事

- [検知の次を自動化する Microsoft Sentinel「SOAR Essentials」を整理してみた](https://zenn.dev/naonao71/articles/sentinel-soar-essentials-solution)
