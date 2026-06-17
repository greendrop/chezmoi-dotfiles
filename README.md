# dotfiles

[chezmoi](https://www.chezmoi.io/) で管理する個人用 dotfiles リポジトリです。macOS / Linux に対応しています。

## セットアップ

### 新規環境にセットアップする

リポジトリの取得と dotfiles の適用を一括で行う:

```bash
chezmoi init --apply git@github.com:greendrop/chezmoi-dotfiles.git
```

適用前に差分を確認したい場合は分割して実行する:

```bash
# リポジトリをクローン
chezmoi init git@github.com:greendrop/chezmoi-dotfiles.git

# 適用内容を確認
chezmoi diff

# 問題がなければ適用
chezmoi apply -v
```

## 使い方

### 管理ファイルを編集する

`chezmoi edit` でソースの該当ファイルをエディタで開き、保存後に `apply` で反映する:

```bash
chezmoi edit ~/.config/ghostty/config
chezmoi apply
```

### 新しいファイルを管理対象に追加する

既存ファイルを chezmoi の管理下に置く:

```bash
chezmoi add ~/.config/foo/bar
```

### リポジトリから最新を取得して適用する

リモートの変更を pull して apply を一括で行う:

```bash
chezmoi update -v
```

差分を確認してから適用したい場合は分割して実行する:

```bash
# 最新を取得（rebase）
chezmoi git pull -- --rebase

# 差分を確認
chezmoi diff

# 適用
chezmoi apply -v
```

### 管理状態を確認する

```bash
# 未適用の変更がないか確認
chezmoi status

# 管理対象のファイル一覧を表示
chezmoi managed
```

### 差分を確認する

ソースとホームディレクトリの差分を確認する:

```bash
chezmoi diff
```

### 管理対象から外す

ファイルを chezmoi の管理から外す（ソースのみ削除、実ファイルは残る）:

```bash
chezmoi forget ~/.config/foo/bar
```

### ソースディレクトリへ移動する

chezmoi が管理するソースディレクトリ（このリポジトリのクローン先）へ移動する:

```bash
chezmoi cd
```

## コミット・プッシュの自動化

`chezmoi edit` / `chezmoi add` での変更を自動でコミット & プッシュするには、
`~/.config/chezmoi/chezmoi.toml` に以下を追記する:

```toml
[git]
    autoCommit = true
    autoPush = true
```

> このリポジトリには `.chezmoi.toml.tmpl` が含まれており、`chezmoi init` 時に
> `~/.config/chezmoi/chezmoi.toml` へ自動生成されます。新規環境でも上記設定が有効になります。

## 自動で最新を取り込む（定期 pull）

chezmoi には自動 pull の仕組みはないため、OS のスケジューラで `chezmoi update`
（`git pull` + `apply`）を定期実行することで自動化できる。macOS は launchd、
Linux は systemd ユーザータイマーを使う。

> いずれもバックグラウンド実行のため、git 認証が非対話で通る必要がある
> （SSH 鍵のパスフレーズなし、または ssh-agent に登録済みであること）。

### macOS（launchd）

以下の内容を `~/Library/LaunchAgents/io.chezmoi.update.plist` に保存する
（chezmoi のパスは `which chezmoi` で確認して合わせる）:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN"
    "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
<plist version="1.0">
<dict>
    <key>Label</key>
    <string>io.chezmoi.update</string>
    <key>ProgramArguments</key>
    <array>
        <string>/opt/homebrew/bin/chezmoi</string>
        <string>update</string>
    </array>
    <key>StartInterval</key>
    <integer>3600</integer>
    <key>RunAtLoad</key>
    <true/>
    <key>StandardOutPath</key>
    <string>/tmp/chezmoi-update.log</string>
    <key>StandardErrorPath</key>
    <string>/tmp/chezmoi-update.log</string>
</dict>
</plist>
```

> `StartInterval` は秒単位（3600 = 1時間ごと）。特定の時刻に実行したい場合は
> `StartCalendarInterval` を使う。

有効化・無効化:

```bash
# 有効化（ログイン中から即時開始）
launchctl load -w ~/Library/LaunchAgents/io.chezmoi.update.plist

# 無効化
launchctl unload -w ~/Library/LaunchAgents/io.chezmoi.update.plist
```

失敗時は `/tmp/chezmoi-update.log` を確認する。

### Linux（systemd ユーザータイマー）

`~/.config/systemd/user/chezmoi-update.service` を作成する
（chezmoi のパスは `which chezmoi` で確認して合わせる）:

```ini
[Unit]
Description=Update dotfiles with chezmoi

[Service]
Type=oneshot
ExecStart=/usr/bin/chezmoi update
```

`~/.config/systemd/user/chezmoi-update.timer` を作成する:

```ini
[Unit]
Description=Run chezmoi update periodically

[Timer]
OnBootSec=5min
OnUnitActiveSec=1h
Persistent=true

[Install]
WantedBy=timers.target
```

> `OnUnitActiveSec` が実行間隔（1h = 1時間ごと）。`Persistent=true` により、
> 電源オフ/スリープ中に逃した実行を起動後に補完する。

有効化・無効化:

```bash
# 有効化（即時開始）
systemctl --user daemon-reload
systemctl --user enable --now chezmoi-update.timer

# 無効化
systemctl --user disable --now chezmoi-update.timer
```

実行ログは `journalctl --user -u chezmoi-update.service` で確認する。
