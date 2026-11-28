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
| スケジュール | 毎日 0:00〜5:59（`Asia/Tokyo`）。朝 6:51 の AI 巡回の前に PR が揃う |
| PR 数の上限 | 1時間あたりは無制限（`prHourlyLimit: 0`）。同時オープンはリポジトリごとに15本（`prConcurrentLimit: 15`）で、マージで空いた分を次の実行で補充する。security PR と Node.js ランタイムの patch / minor は枠の対象外（Node.js の major は他の major と同じく承認制で、枠にも数える） |
| Node.js ランタイム | mise・setup-node・Docker の `node` の patch / minor はスケジュール外でも随時 PR にする（自動マージの条件は他と同じ） |
| patch / minor | 公開から3日経過後、CI（ステータスチェック）成功時に自動マージ。`platformAutomerge` 有効 |
| GitHub Actions（digest 固定） | `pinDigest` は待機なし（`minimumReleaseAge: null`）で、CI 成功時に自動マージ |
| GitHub Actions（公式の digest 更新） | `actions/**`・`github/**` は公開から3日経過後、CI 成功時に自動マージ |
| GitHub Actions（サードパーティの digest 更新） | 自動マージしない。`third-party GitHub Actions digests` として1つの PR にまとめて手動レビュー |
| major | 自動マージしない。Dependency Dashboard でチェックしたものだけ PR にする（`dependencyDashboardApproval`）。security PR は承認不要 |
| Dependency Dashboard | 有効 |
| lockFileMaintenance | 週1回（毎週月曜 0:00〜5:59） |
| 脆弱性アラート | `osvVulnerabilityAlerts: true` で OSV データベースの脆弱性も検知。脆弱性修正 PR（`vulnerabilityAlerts`）には `security` ラベルを付与。security PR は `minimumReleaseAge` を外し（即時作成）、major を含めて CI 通過で自動マージ |
| 更新後の処理 | `postUpdateOptions: ["pnpmDedupe"]` で更新のたびに `pnpm dedupe` を実行（pnpm のリポジトリのみ） |

### 補足

- サプライチェーン攻撃対策として、自動マージ対象の更新には `minimumReleaseAge: "3 days"` と `internalChecksFilter: "strict"` を設定しています。公開から3日経つまでブランチ・PR は作られません。公開日時が取得できない更新は、`minimumReleaseAgeBehaviour: "timestamp-required"`（明示的に設定）により保留されます。
- GitHub Actions の `pinDigest` だけはこの待機を外しています。今使っているタグが指すコミットを固定するだけで新しいコードは入らないので、待機しても守れるものがありません。しかも pinDigest には公開日時が無いので、`timestamp-required` の待機を付けると `renovate/stability-days` が pending のまま止まり、自動マージされません（tv-news-with-tanstack #30 が5日止まった件）。
- サードパーティ Actions の digest 更新はタグの付け替え（tj-actions/changed-files の事件のような攻撃）を含みうるため、自動マージせず人がレビューします。
- 自動マージは CI が成功した場合のみ行われます（Renovate のデフォルト `ignoreTests: false`）。CI が無いリポジトリではステータスチェックが無いため、そのまま自動マージされる点に注意してください。
- `pnpmDedupe` は、ロックファイルの再生成で同じパッケージが別の peer 依存で二重に入り、型の不一致で CI が落ちるのを防ぐためのものです（placeastro で `drizzle-orm` が二重になった件）。
- Node.js ランタイムは OSV の脆弱性アラートの対象外で、security PR が出ません。通常のスケジュール（毎日 0〜5 時）まで待たせず、Node2Shell（CVE-2026-111497）のような修正をすぐ PR にするため、ランタイムだけスケジュールを外しています。
- `platformAutomerge` は GitHub のネイティブ自動マージを使うため、リポジトリ設定で **Allow auto-merge** を有効にし、ブランチ保護で必須ステータスチェックを設定しておく必要があります。

## 検証

```sh
npx --package renovate -- renovate-config-validator default.json
```
