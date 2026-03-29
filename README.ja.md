# hugo-upgrade

[English](README.md)

**Hugo インストール / アップグレードツール**

[Hugo](https://gohugo.io/) 静的サイトジェネレーターを [GitHub リリース](https://github.com/gohugoio/hugo/releases)からインストール・アップグレードするツールです。

## 特徴

- Hugo の新規インストールおよびアップグレードに対応（標準版・拡張版）
- エディション不一致（standard ↔ extended）を検出し、上書き前に警告
- インストール前に SHA256 チェックサムを検証
- アトミックインストール（途中で中断しても壊れたバイナリが残らない）
- 実際にインストールせず確認だけできるドライランおよびアップデートチェックに対応
- 外部依存なし — Python 3 標準ライブラリのみ使用

## 動作要件

| | |
|---|---|
| OS | RHEL / Rocky / AlmaLinux 8.x\*、9.x、10.x · Debian 11+ · Ubuntu 20.04+ |
| Python | 3.8 以上 |
| 権限 | インストール時に `sudo` が必要 |

\* RHEL/Rocky/AlmaLinux 8.x のデフォルト Python は 3.6 です。先に Python 3.8 をインストールしてください:
```bash
sudo dnf install python3.8
```

## インストール

```bash
sudo curl -fsSL https://raw.githubusercontent.com/yamk/hugo-upgrade/main/hugo-upgrade \
    -o /usr/local/bin/hugo-upgrade
sudo chmod 755 /usr/local/bin/hugo-upgrade
```

### `sudo hugo-upgrade` をフルパスなしで実行できるようにする

デフォルトでは `sudo` は `/usr/local/bin` を検索しません。secure_path に追加してください:

```bash
sudo visudo -f /etc/sudoers.d/hugo-upgrade
```

以下の行を追加します:

```
Defaults secure_path = /sbin:/bin:/usr/sbin:/usr/bin:/usr/local/bin
```

## 使い方

```
hugo-upgrade 0.1  Hugo install / upgrade tool

usage: hugo-upgrade [-h] [-e] [-d DIR] [-V VERSION] [-a ARCH] [-u] [-f] [-n] [-c] [-q] [-v]

options:
  -e, --extended        Hugo 拡張版を使用（SCSS/Sass サポートを含む）
  -d DIR, --dir DIR     インストール先ディレクトリ（デフォルト: /usr/local/bin）
  -V VERSION, --hugo-version VERSION
                        インストールする Hugo のバージョンを指定（例: 0.145.0）
  -a ARCH, --arch ARCH  対象アーキテクチャ: amd64 または arm64（デフォルト: 自動検出）
  -u, --upgrade         インストール / アップグレードを実行（変更を行うために必須）
  -f, --force           すでに対象バージョンでも強制的に再インストール
  -n, --dry-run         実際の変更を行わず、実行内容をプレビュー
  -c, --check           アップデートの確認のみ行う（アップデートがあれば終了コード 1）
  -q, --quiet           情報出力を抑制（エラーは引き続き表示）
  -v, --version         スクリプトのバージョンを表示して終了
```

### 使用例

```bash
# 標準版をインストール / アップグレード
sudo hugo-upgrade -u

# 拡張版をインストール / アップグレード
sudo hugo-upgrade -u -e

# アップデートの確認のみ（インストールしない）
hugo-upgrade -c -e

# 実行内容のプレビュー（ドライラン）
hugo-upgrade -n -e

# バージョンを指定してインストール
sudo hugo-upgrade -u -e -V 0.145.0

# 強制的に再インストール（同じバージョン）
sudo hugo-upgrade -u -e -f

# インストール先ディレクトリを指定
sudo hugo-upgrade -u -e -d /opt/bin
```

### 実行例

**初回インストール:**

```
$ sudo hugo-upgrade -u -e
INFO     Installed : not found at /usr/local/bin/hugo
INFO     Checking  : GitHub for latest Hugo release...
INFO     Available : Hugo 0.159.1  (extended, amd64, published 2026-03-26)
INFO     Action    : installing Hugo 0.159.1 (extended)...
INFO     Download  : hugo_extended_0.159.1_linux-amd64.tar.gz
  hugo_extended_0.159.1_linux-amd64.tar.gz  [============================] 100%   18.3 / 18.3 MB
INFO     Verify    : SHA256 checksum...
INFO     Verify    : OK
INFO     Install   : /usr/local/bin/hugo
INFO     Done      : Hugo 0.159.1 (extended) installed successfully.

$ hugo version
hugo v0.159.1+extended linux/amd64 BuildDate=2026-03-26T09:54:15Z VendorInfo=gohugoio
```

**すでに最新の場合:**

```
$ hugo-upgrade -c -e
INFO     Installed : Hugo 0.159.1 (extended)  (/usr/local/bin/hugo)
INFO     Checking  : GitHub for latest Hugo release...
INFO     Available : Hugo 0.159.1  (extended, amd64, published 2026-03-26)
INFO     Status    : already up to date.
```

**エディション不一致（`-e` を忘れた場合）:**

```
$ sudo hugo-upgrade -u
INFO     Installed : Hugo 0.159.1 (extended)  (/usr/local/bin/hugo)
INFO     Checking  : GitHub for latest Hugo release...
INFO     Available : Hugo 0.159.1  (standard, amd64, published 2026-03-26)
WARNING  Warning   : installed edition is extended, but standard was requested.
WARNING             Use -f to force the edition change.
```

**ドライラン:**

```
$ hugo-upgrade -n -e
INFO     Installed : Hugo 0.159.1 (extended)  (/usr/local/bin/hugo)
INFO     Checking  : GitHub for latest Hugo release...
INFO     Available : Hugo 0.159.1  (extended, amd64, published 2026-03-26)
INFO     Status    : already up to date.
```

### 終了コード

| コード | 意味 |
|--------|------|
| 0 | 成功（最新状態、またはインストール完了） |
| 1 | アップデートあり / 未インストール（`--check`）、または一般エラー |
| 2 | GitHub API レート制限超過 |
| 3 | チェックサム不一致（セキュリティ警告） |
| 130 | ユーザーによる中断 |

## ライセンス

MIT
