# release-experimental

GitHub Actions の `release` のイベントについて調べます。

`release` のイベントは対象の Repository の Release の作成・変更・削除などによってトリガーされます。

## Release イベントの Activity type

以下を参考に、Release の状態変化とそれにトリガーされるイベントをまとめる。

https://docs.github.com/ja/actions/writing-workflows/choosing-when-your-workflow-runs/events-that-trigger-workflows#release

```mermaid
stateDiagram-v2
    Draft: draft<br>release or pre-release
    PreRelease: pre-release
    Release: release

    [*] --> Draft: none
    [*] --> PreRelease: prereleased<br>created<br>published
    [*] --> Release: released<br>created<br>published
    Draft --> PreRelease: published
    Draft --> Release: published
    PreRelease --> Release: released<br>(uncheck "Set as a pre-release")
    Release --> PreRelease: prereleased<br>(check "Set as a pre-release")
    PreRelease --> [*]: deleted
    Release --> [*]: deleted

    state PreRelease {
        [*] --> [*]: edited<br>(edit description or title)
        [*] --> [*]: none<br>(change tag)
    }
    state Release {
        [*] --> [*]: edited<br>(edit description or title)
        [*] --> [*]: none<br>(change tag)
        [*] --> [*]: edited<br>(check "Set as the latest release")
    }
```

### Latest release の変更について

２つ以上の Release がある状態で、現在 `Latest Release` ではない方の Release の `Set as the latest release` にチェックして更新すると、 `release.edit` のワークフローがトリガーされる。

**その際の `${{ github.event.action }}` は、これから `Latest` となる方のブランチ（タグ）になる。**

もともと `Latest` で、これからそうでなくなる方からはワークフローはトリガーされない。

### unpublish について

`release` には `unpublish` という Activity type (action) が定義されているが、 GitHub の UI 上の操作からは発生させる方法を見つけられなかった。