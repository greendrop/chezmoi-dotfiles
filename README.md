# dotfiles

[chezmoi](https://www.chezmoi.io/) で管理する個人用 dotfiles リポジトリです。macOS を対象としています。

## 管理対象

| リポジトリパス | 適用先 | 内容 |
|---|---|---|
| `dot_config/ghostty/config` | `~/.config/ghostty/config` | Ghostty ターミナル設定 |

> **chezmoi の命名規則**
> `dot_` 接頭辞が付いたファイル・ディレクトリは、`chezmoi apply` 時にホームディレクトリへ `dot_` を `.` に置き換えた名前で配置されます。

## 前提条件

| ツール | 用途 | インストール |
|---|---|---|
| [chezmoi](https://www.chezmoi.io/install/) | dotfiles 管理 | `brew install chezmoi` |
| [mise](https://mise.jdx.dev/) | 開発ツールのバージョン管理（任意） | `brew install mise` |

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

### ツールをインストールする

[mise](https://mise.jdx.dev/) を使ってリポジトリで管理するツール（gh など）をインストールする:

```bash
# リポジトリのルートで実行
mise install
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
