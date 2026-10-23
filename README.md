# renovate-config

uitspitss のリポジトリで共有する [Renovate](https://docs.renovatebot.com/) プリセットです。

## 使い方

各リポジトリのルートに `renovate.json` を置き、このプリセットを継承します。

```json
{
  "$schema": "https://docs.renovatebot.com/renovate-schema.json",
  "extends": ["github>uitspitss/renovate-config"]
}
```

リポジトリ固有の設定が必要な場合は、`extends` の後に追記すると上書きできます。

## プリセットの内容（`default.json`）

| 項目 | 設定 |
| --- | --- |
| ベース | `config:recommended` と `helpers:pinGitHubActionDigests` を継承 |
| スケジュール | 毎週月曜 0:00〜5:59（`Asia/Tokyo`） |
| patch / minor | 公開から3日経過後、CI（ステータスチェック）成功時に自動マージ。`platformAutomerge` 有効 |
| GitHub Actions | digest に固定。digest 更新も公開から3日経過後、CI 成功時に自動マージ |
| major | 自動マージしない（PR を手動でレビュー・マージ） |
| Dependency Dashboard | 有効 |
| lockFileMaintenance | 週1回（上記スケジュールと同じ枠） |

### 補足

- サプライチェーン攻撃対策として、自動マージ対象の更新には `minimumReleaseAge: "3 days"` と `internalChecksFilter: "strict"` を設定しています。公開から3日経つまでブランチ・PR は作られません。公開日時が取得できない更新は、Renovate のデフォルト（`minimumReleaseAgeBehaviour: "timestamp-required"`）により保留されます。
- 自動マージは CI が成功した場合のみ行われます（Renovate のデフォルト `ignoreTests: false`）。CI が無いリポジトリではステータスチェックが無いため、そのまま自動マージされる点に注意してください。
- `platformAutomerge` は GitHub のネイティブ自動マージを使うため、リポジトリ設定で **Allow auto-merge** を有効にし、ブランチ保護で必須ステータスチェックを設定しておく必要があります。

## 検証

```sh
npx --package renovate -- renovate-config-validator default.json
```
