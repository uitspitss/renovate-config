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
| ベース | `config:recommended` を継承 |
| スケジュール | 毎週月曜 0:00〜5:59（`Asia/Tokyo`） |
| patch / minor | CI（ステータスチェック）成功時に自動マージ。`platformAutomerge` 有効 |
| major | 自動マージしない（PR を手動でレビュー・マージ） |
| Dependency Dashboard | 有効 |
| lockFileMaintenance | 週1回（上記スケジュールと同じ枠） |

### 補足

- 自動マージは CI が成功した場合のみ行われます（Renovate のデフォルト `ignoreTests: false`）。CI が無いリポジトリではステータスチェックが無いため、そのまま自動マージされる点に注意してください。
- `platformAutomerge` は GitHub のネイティブ自動マージを使うため、リポジトリ設定で **Allow auto-merge** を有効にし、ブランチ保護で必須ステータスチェックを設定しておく必要があります。

## 検証

```sh
npx --package renovate -- renovate-config-validator default.json
```
