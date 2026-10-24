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
| GitHub Actions（digest 固定） | `pinDigest` は公開から3日経過後、CI 成功時に自動マージ |
| GitHub Actions（公式の digest 更新） | `actions/**`・`github/**` は公開から3日経過後、CI 成功時に自動マージ |
| GitHub Actions（サードパーティの digest 更新） | 自動マージしない。`third-party GitHub Actions digests` として1つの PR にまとめて手動レビュー |
| major | 自動マージしない（PR を手動でレビュー・マージ） |
| Dependency Dashboard | 有効 |
| lockFileMaintenance | 週1回（上記スケジュールと同じ枠） |
| 更新後の処理 | `postUpdateOptions: ["pnpmDedupe"]` で更新のたびに `pnpm dedupe` を実行（pnpm のリポジトリのみ） |

### 補足

- サプライチェーン攻撃対策として、自動マージ対象の更新には `minimumReleaseAge: "3 days"` と `internalChecksFilter: "strict"` を設定しています。公開から3日経つまでブランチ・PR は作られません。公開日時が取得できない更新は、`minimumReleaseAgeBehaviour: "timestamp-required"`（明示的に設定）により保留されます。
- サードパーティ Actions の digest 更新はタグの付け替え（tj-actions/changed-files の事件のような攻撃）を含みうるため、自動マージせず人がレビューします。
- 自動マージは CI が成功した場合のみ行われます（Renovate のデフォルト `ignoreTests: false`）。CI が無いリポジトリではステータスチェックが無いため、そのまま自動マージされる点に注意してください。
- `pnpmDedupe` は、ロックファイルの再生成で同じパッケージが別の peer 依存で二重に入り、型の不一致で CI が落ちるのを防ぐためのものです（placeastro で `drizzle-orm` が二重になった件）。
- `platformAutomerge` は GitHub のネイティブ自動マージを使うため、リポジトリ設定で **Allow auto-merge** を有効にし、ブランチ保護で必須ステータスチェックを設定しておく必要があります。

## 検証

```sh
npx --package renovate -- renovate-config-validator default.json
```
