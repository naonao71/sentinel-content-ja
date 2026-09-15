# sentinel-content-ja

Microsoft Sentinel のコンテンツパックを、**統合ポータル（Microsoft Defender ポータル）で動くように修正**し、必要に応じて**日本語化**したものを置いています。

すべて **Microsoft Sentinel をオンボード済みのラボ環境で実機検証**してから公開しています。

## なぜこのリポジトリがあるのか

Content Hub のコンテンツパックには、**Azure ポータル時代に作られたまま更新されていないもの**があります。統合ポータルへ移行した環境でそのまま入れると、次のような問題が起きます。

- **デプロイも実行も成功するのに、何も起きない**
- README の手順が、現在の画面に存在しない設定を指している
- 日本語環境では本文が文字化けしている

ここでは、そうした箇所を実機で特定し、**最小限の変更で動くようにしたもの**を置いています。

## 収録内容

| ソリューション | 状態 | 内容 |
| --- | --- | --- |
| [SentinelSOARessentials](solutions/SentinelSOARessentials/) | ✅ 検証済み | タスク化 Playbook 3 本の修正版（Phishing は日本語化済み） |

## 構成

本家 [Azure/Azure-Sentinel](https://github.com/Azure/Azure-Sentinel) と**同じディレクトリ構成**にしています。元ファイルとの差分が追いやすく、必要なら本家へ PR を出せる形です。

```
solutions/
└── <ソリューション名>/
    ├── README.md                  ← 修正の一覧と検証結果
    └── Playbooks/
        └── <Playbook 名>/
            ├── azuredeploy.json          ← 修正版（これを使う）
            ├── azuredeploy.original.json ← 取り込み時点の本家（比較用）
            └── README.md                 ← 変更点とデプロイ手順
```

`azuredeploy.original.json` を残しているのは、**本家が更新されたときに差分を取れるようにする**ためです。

## 使い方

1. 使いたい Playbook のフォルダを開く
2. `README.md` で変更点を確認する
3. `azuredeploy.json` をダウンロードして `az deployment group create` でデプロイする
4. Logic App の**システム割り当てマネージド ID** に必要なロールを付与する

詳しい手順は各 Playbook の README に書いています。

## 本家との関係

- 元は [Azure/Azure-Sentinel](https://github.com/Azure/Azure-Sentinel)（MIT ライセンス）です
- 各ファイルの取り込み日とバージョンは、ソリューションごとの README に記載しています
- **本家の更新には自動で追随しません。** 取り込み時点の内容に対する修正です

## ライセンス

[MIT License](LICENSE)。元のコンテンツの著作権は Microsoft Corporation に帰属します。改変した箇所は各 README に明記しています。

## 免責

本リポジトリは**個人の検証に基づくもの**であり、所属する組織の公式見解や公式サポートを示すものではありません。利用にあたっては、必ずご自身の環境で動作を確認してください。

製品仕様は変更されるため、最新情報は [Microsoft Learn](https://learn.microsoft.com/azure/sentinel/) でご確認ください。

## 関連記事

- [検知の次を自動化する Microsoft Sentinel「SOAR Essentials」を整理してみた](https://zenn.dev/naonao71)
