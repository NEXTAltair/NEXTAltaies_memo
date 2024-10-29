# Gitでプロジェクト構造を移行する方法

## はじめに

プロジェクトの構造を変更するときに単純に移動するだけではGitに移動の履歴が残らない。

例えば､一度コミットして後から'.gitignore'に追加したファイルをD&Dで削除コミットした後に、ブランチを切り替えるとファイルが消える問題が起きた。状況は以下の通り：

## 現在の状況

1. main ブランチでは `config.ini`がルートディレクトリにある
2. `add-network-error-handling`ブランチでは `config/config.ini`に移動
3. ブランチ切り替え時に設定ファイルが予期せず消失

## 重要な注意点

- Git は通常、未追跡ファイルを削除しない
- しかし、ディレクトリ構造の変更が絡むと予期せぬ動作が起きることがある

生成AI画像のタグ管理ツール「genai-tag-db-tools」を､Gitの履歴を保持したまプロジェクト構造を変更する方法を順を追ってメモっておく

## プロジェクト構造

### 移行前の構造

```
genai-tag-db-tools/
├── CSVToDatabaseProcessor.py
├── gui/
│   ├── MainWindow.py
│   ├── MainWindow.ui
│   ├── TagCleanerWidget.py
│   ├── TagCleanerWidget.ui
│   ├── TagRegisterWidget.py
│   ├── TagRegisterWidget.ui
│   ├── TagSearchWidget.py
│   ├── TagSearchWidget.ui
│   ├── TagStatisticsWidget.py
│   ├── TagStatisticsWidget.ui
│   └── __init__.py
├── main.py
├── tags_v3.db
└── tag_search.py
```

### 移行後の目標構造

```
genai-tag-db-tools/
├── src/
│   └── genai_tag_db_tools/
│       ├── core/
│       │   ├── processor.py     # 旧CSVToDatabaseProcessor.py
│       │   └── searcher.py      # 旧tag_search.py
│       └── gui/
│           ├── __init__.py
│           ├── windows/              # メインウィンドウなど、アプリケーションの主要なウィンドウ
│           │   ├── __init__.py
│           │   └── main_window.py   # メインウィンドウの実装
│           ├── widgets/             # カスタムウィジェット (手動で作成)
│           │   ├── __init__.py
│           │   ├── tag_cleaner.py
│           │   ├── tag_register.py
│           │   ├── tag_search.py
│           │   └── tag_statistics.py
│           └── designer/           # Qt Designerで作成したUIファイルとその生成コード
│               ├── __init__.py
│               ├── MainWindow.ui     # Qt Designerで作成したUIファイル
│               ├── MainWindow_ui.py  # uicで生成されたPythonコード
│               ├── TagCleaner.ui
│               ├── TagCleaner_ui.py
│               ├── TagRegister.ui
│               ├── TagRegister_ui.py
│               ├── TagSearch.ui
│               ├── TagSearch_ui.py
│               ├── TagStatistics.ui
│               └── TagStatistics_ui.py
├── data/
│   └── tags_v3.db
└── main.py
```

---

# 移行の準備

### バックアップの作成

```powershell
# プロジェクトディレクトリ全体をバックアップ
Copy-Item -Path "H:\lora\素材リスト\スクリプト\src\module\genai-tag-db-tools" `
          -Destination "H:\lora\素材リスト\スクリプト\src\module\genai-tag-db-tools_backup" `
          -Recurse
```

### 1. プロジェクト状態の確認

```powershell
git status
```

### 2. 新しいディレクトリ構造の定義

```powershell
$dirs = @(
    "src/genai_tag_db_tools/core",
    "src/genai_tag_db_tools/gui/windows",
    "src/genai_tag_db_tools/gui/widgets",
    "src/genai_tag_db_tools/gui/desiner",
    "data"
)

```

### 3. ディレクトリの作成

```powershell
foreach ($dir in $dirs) {
    New-Item -Path $dir -ItemType Directory -Force
}
```

### 4. `__init__.py`ファイルの準備

```powershell
$init_paths = @(
    "src/genai_tag_db_tools/__init__.py",
    "src/genai_tag_db_tools/core/__init__.py",
    "src/genai_tag_db_tools/gui/__init__.py",
    "src/genai_tag_db_tools/gui/windows/__init__.py",
    "src/genai_tag_db_tools/gui/widgets/__init__.py",
    "src/genai_tag_db_tools/gui/designer/__init__.py"
)

foreach ($path in $init_paths) {
    # 親ディレクトリを確実に作成
    $directory = Split-Path -Parent $path
    New-Item -Path $directory -ItemType Directory -Force

    # __init__.pyファイルを作成
    New-Item -Path $path -ItemType File -Force
}
```

### 5. `pyproject.toml`の作成

```
$pyproject_content = @'
# ビルドシステムの設定
[build-system]
# Pythonパッケージングツールの指定
requires = ["hatchling"]  # モダンなビルドシステム
build-backend = "hatchling.build"  # ビルドバックエンドの指定

# プロジェクトの基本情報
[project]
name = "genai-tag-db-tools"  # パッケージ名 ハイフン推奨なぜかは知らん
version = "0.1.0"  # バージョン番号
description = "AI生成画像のタグ管理ツール"  # プロジェクトの説明
requires-python = ">=3.12"  # 必要なPythonバージョン

# 必要なパッケージの依存関係
dependencies = [
    "PySide6>=6.8.0.2",        # Qt GUIフレームワーク
    "pandas>=2.2.2",           # データ分析ライブラリ
    "superqt>=0.6.7",          # PySide6の拡張機能
    "polars>=1.9.0",           # 高性能データフレームライブラリ
]

# プロジェクトの分類情報
classifiers = [
    "Operating System :: Microsoft :: Windows :: Windows 11"  # 対応OS
]

# pytestの設定オプション
[tool.pytest.ini_options]
addopts = "-ra -q -v"         # テスト実行時のオプション（結果表示の詳細度など）
testpaths = ["test"]          # テストファイルの場所
pythonpath = ["src"]          # テスト実行時のPythonパス

# 開発時のみ必要な追加パッケージ
[project.optional-dependencies]
dev = [
    "pytest>=8.3.3",          # テストフレームワーク
    "pytest-cov>=5.0.0",      # テストカバレッジツール
]

# ビルド設定
[tool.hatch.build.targets.wheel]
packages = ["src/genai_tag_db_tools"]            # パッケージ名アンダースコア推奨

# プロジェクトスクリプト
[project.scripts]
genai-tag-db-tools = "genai_tag_db_tools.main:main"  #インストールしてgenai_tag_db_toolswpターミナルで入力するとmainが実行されるよという話

# Pyright（静的型チェッカー）の設定
[tool.pyright]
include = ["src"]             # 型チェック対象
exclude = [                   # 型チェック除外対象
    "**/__pycache__",
    "test"
]
venvPath = "."               # 仮想環境のパス
venv = "venv"               # 仮想環境の名前
typeCheckingMode = "basic"  # 型チェックの厳格さ

# PySide6関連の特別な設定
reportGeneralTypeIssues = "warning"     # 一般的な型の問題を警告として報告
reportMissingTypeStubs = false          # 型定義ファイルの不足を無視
reportUnknownMemberType = false         # 不明なメンバー型を無視

# 追加のパス設定
extraPaths = ["src/gui"]     # GUIモジュールへのパス
'@

Set-Content -Path "pyproject.toml" -Value $pyproject_content
```

### 6. 準備作業のコミット

```powershell
git add src/ data/ pyproject.toml
git commit -m "build: Add new directory structure and pyproject.toml"
```

---

# ファイルの移動

## 1. コアファイルの移動

```powershell
# プロセッサの移動
git mv "CSVToDatabaseProcessor.py" "src/genai_tag_db_tools/core/processor.py"

# 検索モジュールの移動
git mv "tag_search.py" "src/genai_tag_db_tools/core/searcher.py"
```

## 2. GUIファイルの移動

### メインウィンドウ

```powershell
git mv "gui/MainWindow.py" "src/genai_tag_db_tools/gui/windows/main_window.py"
git mv "gui/MainWindow.ui" "src/genai_tag_db_tools/gui/windows/MainWindow.ui"
git mv "gui/MainWindow_ui.py" "src/genai_tag_db_tools/gui/windows/MainWindow_ui.py"
```

### ウィジェットファイル

```powershell
$widgets = @{
    "TagCleanerWidget.py" = "tag_cleaner.py"
    "TagRegisterWidget.py" = "tag_register.py"
    "TagSearchWidget.py" = "tag_search.py"
    "TagStatisticsWidget.py" = "tag_statistics.py"
}

foreach ($widget in $widgets.GetEnumerator()) {
    git mv "gui/$($widget.Key)" "src/genai_tag_db_tools/gui/widgets/$($widget.Value)"
}
```

### UIファイル

```powershell
$ui_files = @{
    "TagCleanerWidget.ui" = "designer/TagCleaner.ui"
    "TagRegisterWidget.ui" = "designer/TagRegister.ui"
    "TagSearchWidget.ui" = "designer/TagSearch.ui"
    "TagStatisticsWidget.ui" = "designer/TagStatistics.ui"
    "TagCleanerWidget_ui.py" = "designer/TagCleaner_ui.py"
    "TagRegisterWidget_ui.py" = "designer/TagRegister_ui.py"
    "TagSearchWidget_ui.py" = "designer/TagSearch_ui.py"
    "TagStatisticsWidget_ui.py" = "designer/TagStatistics_ui.py"
}

foreach ($ui in $ui_files.GetEnumerator()) {
    git mv "gui/$($ui.Key)" "src/genai_tag_db_tools/gui/$($ui.Value)"
}
```

---

# データファイルの管理

## データベースファイルの移動とエラー対処

### エラーの例

```
Changes not staged for commit:
  (use "git add/rm <file>..." to update what will be committed)
  (use "git restore <file>..." to discard changes in working directory)
        deleted:    tags_v3.db

Untracked files:
  (use "git add <file>..." to include in what will be committed)
        data/
```

### エラーが起きた理由

このエラーは、データベースファイル (`tags_v3.db`) の容量がGitで管理するには大きすぎたことだろうか？

他にはデータベースファイルがの更新頻度が高すぎてコミット履歴がふえてGitの追跡が不安定になったって可能性はないなデータベースの変更でいちいちコミットしないし

他のプロセスで接続があったのが原因かも。

### 解決手順

1. `.gitignore`の設定

   ```powershell
   Add-Content -Path ".gitignore" -Value @"
   # データファイル
   data/
   "@
   ```
2. Gitの追跡から外す

   ```powershell
   git rm --cached tags_v3.db
   ```
3. 設定の保存

   ```powershell
   git add .gitignore
   git commit -m "build: データベースを一旦Gitの管理下から外す"
   ```
4. ファイルの移動

   ```powershell
   Move-Item -Path "tags_v3.db" -Destination "data/tags_v3.db"
   ```
5. `.gitignore`の編集

   ```gitignore
   # 以下をコメントアウトまたは削除
   # data/
   ```
6. 追跡の再開

   ```powershell
   git add data/tags_v3.db

   ```

---

# インポートパスの更新

## 格上げ(プロモート)の修正

Qt Designer 格上げされたウィジェット/格上げ先 から 格上げされたクラスを全部消す

カスタムウィジェットを使用している `*.ui` ファイルの編集で \`格上げされたクラス名\` をカスタムウィジェットで定義されたクラス名

ヘッダファイルをカスタムウィジェットクラスを定義してあるパスの相対パスに設定して `格上げ` ボタンを押す

自動生成された `*_ui.py` には自分で書いたカスタムウィジェットクラスの正しいインポート先が指定されてる

｢ヘッダー？ C++ か関係ねえな｣と思ってたらそんなこともなかった&#x20;

## GUIモジュール間の依存関係

### メインウィンドウで格上げされたウィジェットインポート

Qt がやってくれるところ

変更前（gui/MainWindow\_ui.py）:

```python
# 古い構造（同一ディレクトリからのインポート）
from TagCleanerWidget import TagCleanerWidget
from TagRegisterWidget import TagRegisterWidget
from TagSearchWidget import TagSearchWidget
from TagStatisticsWidget import TagStatisticsWidget
```

変更後(src/genai\_tag\_db\_tools/gui/windows/main\_window\.py)

```
from ..widgets.tag_cleaner import TagCleanerWidget
from ..widgets.tag_register import TagRegisterWidget
from ..widgets.tag_search import TagSearchWidget
from ..widgets.tag_statistics import TagStatisticsWidget
```

### UIファイルの参照

変更前（`gui/TagRegisterWidget.py`）:

```python
# 古い構造
from TagRegisterWidget_ui import Ui_TagRegisterWidget
from CSVToDatabaseProcessor import CSVToDatabaseProcessor
```

変更後（`src/genai_tag_db_tools/gui/widgets/tag_cleaner.py`）:

```python
# 同じディレクトリ内のUIファイル
from ..designer.TagRegisterWidget_ui import Ui_TagRegisterWidget
from ...core.processor import CSVToDatabaseProcessor
```

### メインウィンドウUIの参照

変更前（`gui/MainWindow.py`）:

```python
# 古い構造
from MainWindow_ui import Ui_MainWindow
```

変更後（`src/genai_tag_db_tools/gui/windows/main_window.py`）:

```python
# 同じディレクトリ内のUIファイル
from .MainWindow_ui import Ui_MainWindow
```
