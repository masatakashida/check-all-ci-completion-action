# Check All CI Completion (GitHub Actions)

[英語(Eng)](./README.md) | [日本語(JP)](./README_jp.md)

このリポジトリは、**複数 CI を利用時に Auto Merge を使いやすくするためのGitHub Actions** を公開します。

## コンセプト

複数の CI が Pull Request に存在しそれぞれが特定の条件でのみ動く場合、特定の CI の成功を強制して実行する GitHub Auto-Merge は使用できません。

例えば以下の図では、Bazel、Spanner のジョブが実行され Kubernetes Manifest のジョブは実行されていません。

![multi-job](images/multi-jobs.png)

このとき、どのジョブの完了を必須にしても、更新内容によっては Auto-Merge が使えません。

Check All CI Completion では、すべての起動した CI が成功しているかどうかをチェックします。そのため **Check All CI Completion ジョブが成功することを強制する** ことで GitHub Auto-merge を利用しやすくします。

![check-job](images/check-job.png)

## 使い方

```yaml
- uses: lirlia/check-all-ci-completion-action@v1.0.0
  with:
    # Default: env.GITHUB_SHA
    # check suites を取得するためのコミットハッシュ
    commit-hash: 'put any git commit hash'

    # Default: false
    # true の場合、ジョブに失敗しても exit 0 で正常終了します
    disable-errexit: 'true' or 'false'

    # Default: 000000000000000 (dummy value)
    # 無視する check suite id を指定(複数の場合は,で区切る)
    ignore-check-suite-ids: '11111111,22222222'

    # Default: -1 (タイムアウトまでループします)
    # ジョブ内で何回ループを行うか
    loop-count: '10'

    # Default: 300 seconds
    # 他のジョブの状態をチェックするまでの待機秒数
    sleep-seconds: '300'

    # Default: 1800 seconds
    # このジョブを維持する秒数(このジョブは他のジョブの終了を待機します)
    timeout-seconds: '1800'
```

### Outputs

| name   | value                    | description                                                                            |
| ------ | ------------------------ | -------------------------------------------------------------------------------------- |
| result | success / fail           | 別の CI ジョブの結果（すべてのジョブが success または neutralの場合、結果は success です |
| status | completed / in-progress | 別の CI ジョブのステータスです                                                           |

### Full yaml (for main branch)

```yaml
name: check-all-ci-result
on:
  push:
    branches:
      - '**'        # すべてのブランチにマッチする
      - '!main'     # main ブランチは除く
jobs:
  check-another-ci:
    name: check
    runs-on: ubuntu-20.04
    steps:
      - uses: lirlia/check-all-ci-completion-action@latest
```

### GitHub 設定

- [Setting] → [Branches] → [Branch protection rule] → [Edit]
- "Require status checks to pass before merging" の有効化
- "Require branches to be up to date before merging" の有効化
- "Status checks that are required." で "check" を設定してください

## :warning:注意:warning:

このジョブは他のジョブの完了を待っている間は待機し続けるため GitHub Actions の課金対象になります。ユースケースによっては予期しない額の課金が発生する可能性があります。

## License

[License](./LICENSE)
