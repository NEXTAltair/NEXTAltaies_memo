# Git でのディレクトリ構造変更の注意点とベストプラクティス

# 発生した問題

main ブランチのプロジェクトルートにあった`config.ini`を`add-network-error-handling`ブランチで`config/config.ini`に移動した時に起きた問題をまとめる。

## やってしまった操作

1. main ブランチから`add-network-error-handling`ブランチを作成
2. エクスプローラーで D&D で`config.ini`を移動
3. 新しい場所でコミット
4. その後 main ブランチに戻ったら`config.ini`が消えていた

## 問題が起きた技術的な理由

1. D&D での移動は Git が追跡できない

   - Git からすると「古い場所のファイル削除」+「新しい場所に新規ファイル作成」
   - 結果として履歴が途切れる
   - ブランチ間での整合性が失われる

2. 設定ファイルの扱いの問題
   - `config.ini`は通常`.gitignore`で除外する類のファイル
   - 未追跡状態だとブランチ切り替え時に予期せぬ動作が起きやすい

# 解決方法

## 緊急対処

設定が消えてしまった場合：

```bash
# mainブランチの状態を保存
git checkout main
cp config.ini config.ini.backup

# 新ブランチで適切な場所に移動
git checkout add-network-error-handling
# Windows PowerShell の場合
New-Item -ItemType Directory -Force -Path config

# Unix系の場合
mkdir -p config
mv config.ini.backup config/config.ini
```

## 正しいやり方での移動手順

```bash
# mainブランチから開始
git checkout main

# 構造変更用の新しいブランチを作成
git checkout -b fix-directory-structure

# Git管理下でファイルを移動
git mv config.ini config/config.ini

# 変更をコミット
git commit -m "Move config.ini to config directory"

# 開発ブランチにマージ
git checkout add-network-error-handling
git merge fix-directory-structure
```

# 今後の予防策

## 1. 設定ファイルの管理方法の改善

```bash
# テンプレートとして管理
Copy-Item config.ini config/config.ini.template
Add-Content -Path .gitignore -Value "config/config.ini"
Add-Content -Path .gitignore -Value "config.ini"

# READMEに手順を追加
echo "# Setup
1. Copy config.ini.template to config/config.ini
2. Edit config/config.ini with your settings" >> README.md
```

## 2. セットアップスクリプトの作成

```python
# branch_config_helper.py
from pathlib import Path
import subprocess

def get_current_branch() -> str:
    """現在のGitブランチ名を取得"""
    result = subprocess.run(['git', 'branch', '--show-current'],
                          capture_output=True, text=True)
    return result.stdout.strip()

def manage_config():
    """ブランチに応じて設定ファイルを移動"""
    current_branch = get_current_branch()
    root_config = Path('config.ini')
    config_dir = Path('config')
    dir_config = config_dir / 'config.ini'

    # 設定ファイルのバックアップを作成（存在する方の設定を保存）
    if root_config.exists():
        backup_data = root_config.read_bytes()
        source = "root"
    elif dir_config.exists():
        backup_data = dir_config.read_bytes()
        source = "dir"
    else:
        print("設定ファイルが見つかりません")
        return

    # ブランチに応じて適切な場所に設定を配置
    if current_branch in ['main', 'dev']:
        if source == "dir":
            # configディレクトリの設定をルートに移動
            root_config.write_bytes(backup_data)
            dir_config.unlink()  # 元のファイルを削除
            # 空のconfigディレクトリを削除
            if not any(config_dir.iterdir()):
                config_dir.rmdir()
    elif current_branch == 'add-network-error-handling':
        if source == "root":
            # ルートの設定をconfigディレクトリに移動
            config_dir.mkdir(exist_ok=True)
            dir_config.write_bytes(backup_data)
            root_config.unlink()  # 元のファイルを削除

if __name__ == '__main__':
    manage_config()
```

# ベストプラクティス

## 1. ディレクトリ構造変更のルール

- 必ず`git mv`コマンドを使う
- 大きな変更は専用のブランチを作る
- 変更前に必ずバックアップを取る
- エクスプローラーでの直接操作は避ける

## 2. 設定ファイルの扱い方

- テンプレートファイルは Git で管理する
- 実際の設定ファイルは`.gitignore`で除外する
- ファイルの場所は全ブランチで統一する
- 環境依存の設定は環境変数を使う

## 3. ドキュメント管理

- README.md に設定手順を書く
- ディレクトリ構造の変更履歴を残す
- セットアップ手順を明確に書く

# 推奨するディレクトリ構造

```
project-root/
├── config/
│   ├── config.ini.template
│   └── .gitkeep
├── src/
│   └── your_package/
├── tests/
├── .gitignore
└── README.md
```

# .gitignore の設定例

```gitignore
# 設定ファイル
config.ini
config/*.ini
!config/*.template

# Python
__pycache__/
*.py[cod]
*$py.class
.pytest_cache/

# 仮想環境
venv/
.env

# IDE
.vscode/
.idea/
```

# README.md への追記例

````markdown
# 初期設定

1. 設定ファイルの準備:
   ```bash
   cp config/config.ini.template config/config.ini
   ```
````

2. 設定の編集:

   - `config/config.ini`を環境に合わせて編集
   - 編集が必要な項目:
     - API_KEY
     - DATABASE_URL
     - LOG_LEVEL

3. 動作確認:
   ```bash
   python setup.py
   pytest tests/
   ```

```

# まとめ

- ファイル操作は必ずGitコマンドを使う
- 設定ファイルはテンプレート方式で管理する
- ディレクトリ構造の変更は慎重に計画を立てる
- 変更前には必ずバックアップを取る
- 手順は必ずドキュメント化する

これらの方針に従うことで、設定ファイルの管理とディレクトリ構造の変更をより安全に行える。
```
