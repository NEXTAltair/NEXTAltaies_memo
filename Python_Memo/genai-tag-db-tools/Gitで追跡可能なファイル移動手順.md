# Git で追跡できるプロジェクト構造の移行方法

## はじめに

プロジェクトの構造を変更する際、単純にファイルを移動するだけでは Git の履歴が途切れてしまいます。この記事では、生成 AI 画像のタグ管理ツール「genai-tag-db-tools」を例に、Git の履歴を保持したままプロジェクト構造を変更する方法を解説します。

## プロジェクト構造

### 移行前の構造

```
genai-tag-db-tools/
├── CSVToDatabaseProcessor.py
├── gui/
│   ├── MainWindow.py
│   ├── MainWindow_ui.py
│   ├── TagCleanerWidget.py
│   ├── TagCleanerWidget_ui.py
│   ├── TagRegisterWidget.py
│   ├── TagRegisterWidget_ui.py
│   ├── TagSearchWidget.py
│   ├── TagSearchWidget_ui.py
│   ├── TagStatisticsWidget.py
│   ├── TagStatisticsWidget_ui.py
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
│           ├── windows/
│           │   └── main_window.py    # 旧MainWindow.py
│           └── widgets/
│               ├── tag_cleaner.py    # 旧TagCleanerWidget.py
│               ├── tag_register.py   # 旧TagRegisterWidget.py
│               ├── tag_search.py     # 旧TagSearchWidget.py
│               └── tag_statistics.py # 旧TagStatisticsWidget.py
├── data/
│   └── tags_v3.db
└── main.py
```

## 移行の準備

### 一般的な間違い

PowerShell の `Move-Item` を使用すると Git の履歴が失われます：

```powershell
Move-Item -Path "CSVToDatabaseProcessor.py" `
          -Destination "src/genai_tag_db_tools/core/processor.py"
```

この方法の問題点：

- コミット履歴が途切れる
- 変更履歴が失われる
- マージが困難になる

### 正しい移行手順

#### 1. プロジェクト状態の確認

```powershell
git status
```

#### 2. 新しいディレクトリ構造の定義

```powershell
$dirs = @(
    "src/genai_tag_db_tools/core",
    "src/genai_tag_db_tools/gui/windows",
    "src/genai_tag_db_tools/gui/widgets",
    "data"
)
```

#### 3. ディレクトリの作成

```powershell
foreach ($dir in $dirs) {
    New-Item -Path $dir -ItemType Directory -Force
}
```

#### 4. `__init__.py` ファイルの準備

```powershell
$init_paths = @(
    "src/genai_tag_db_tools/__init__.py",
    "src/genai_tag_db_tools/core/__init__.py",
    "src/genai_tag_db_tools/gui/__init__.py",
    "src/genai_tag_db_tools/gui/windows/__init__.py",
    "src/genai_tag_db_tools/gui/widgets/__init__.py"
)

foreach ($path in $init_paths) {
    New-Item -Path $path -ItemType File -Force
}
```

#### 5. `pyproject.toml` の作成

```powershell
$pyproject_content = @'
[build-system]
requires = ["hatchling"]
build-backend = "hatchling.build"

[project]
name = "genai-tag-db-tools"
version = "0.1.0"
description = "AI生成画像のタグ管理ツール"
requires-python = ">=3.12"
dependencies = [
    "PySide6",
    "pandas",
    "superqt",
    "polars",
    "pytest",
]

classifiers = [
    "Operating System :: Microsoft :: Windows :: Windows 11"
]

[tool.pytest.ini_options]
addopts = "-ra -q -v"
testpaths = ["test"]
pythonpath = ["src"]

[project.optional-dependencies]
dev = ["pytest>=8.3.3", "pytest-cov>=5.0.0"]

[tool.hatch.build.targets.wheel]
packages = ["src"]
'@

Set-Content -Path "pyproject.toml" -Value $pyproject_content
```

#### 6. 準備作業のコミット

```powershell
git add src/ data/ pyproject.toml
git commit -m "build: Add new directory structure and pyproject.toml"
```

## ファイルの移動

### 1. コアファイルの移動

```powershell
# プロセッサの移動
git mv "CSVToDatabaseProcessor.py" "src/genai_tag_db_tools/core/processor.py"

# 検索モジュールの移動
git mv "tag_search.py" "src/genai_tag_db_tools/core/searcher.py"
```

### 2. GUI ファイルの移動

#### メインウィンドウ:

```powershell
git mv "gui/MainWindow.py" "src/genai_tag_db_tools/gui/windows/main_window.py"
git mv "gui/MainWindow_ui.py" "src/genai_tag_db_tools/gui/windows/main_window_ui.py"
```

#### ウィジェットファイル:

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

#### UI ファイル:

```powershell
$ui_files = @{
    "TagCleanerWidget_ui.py" = "tag_cleaner_ui.py"
    "TagRegisterWidget_ui.py" = "tag_register_ui.py"
    "TagSearchWidget_ui.py" = "tag_search_ui.py"
    "TagStatisticsWidget_ui.py" = "tag_statistics_ui.py"
}

foreach ($ui in $ui_files.GetEnumerator()) {
    git mv "gui/$($ui.Key)" "src/genai_tag_db_tools/gui/widgets/$($ui.Value)"
}
```

## データファイルの管理

### データベースファイルの移動とエラー対処

#### エラーの例

```
Changes not staged for commit:
  (use "git add/rm <file>..." to update what will be committed)
  (use "git restore <file>..." to discard changes in working directory)
        deleted:    tags_v3.db

Untracked files:
  (use "git add <file>..." to include in what will be committed)
        data/
```

#### 解決手順

1. `.gitignore` の設定:

   ```powershell
   Add-Content -Path ".gitignore" -Value @"
   # データファイル
   data/
   "@
   ```

2. Git の追跡から外す:

   ```powershell
   git rm --cached tags_v3.db
   ```

3. 設定の保存:

   ```powershell
   git add .gitignore
   git commit -m "build: Exclude database files from git tracking"
   ```

4. ファイルの移動:

   ```powershell
   Move-Item -Path "tags_v3.db" -Destination "data/tags_v3.db"
   ```

### データファイルの追跡管理（オプション）

#### 追跡を再開する場合

1. `.gitignore` の編集:

   ```gitignore
   # 以下をコメントアウトまたは削除
   # data/
   ```

2. 追跡の再開:

   ```powershell
   # 特定のファイルを追加
   git add data/tags_v3.db
   ```

# Qt UI ファイルの自動生成とインポートパス設定

## UI ファイル生成の特徴

- `*_ui.py` ファイルは Qt Designer から自動生成される
- インポートパスは生成時に自動的に設定される
- 手動でのインポートパス修正は生成時に上書きされてしまう

## 正しい設定方法

### 1. VS Code での Qt Designer 設定

settings.json に Qt Designer の設定を追加：

```json
{
  "qt-designer.uiFilePattern": "src/genai_tag_db_tools/gui/**/*.ui",
  "qt-designer.pyuicPath": "${workspaceFolder}/venv/Scripts/pyside6-uic.exe",
  "qt-designer.pythonPath": "${workspaceFolder}/venv/Scripts/python.exe",
  "qt-designer.outputPattern": "${fileDirname}/${fileBasenameNoExtension}_ui.py"
}
```

### 2. UI ファイル生成スクリプトの作成

生成時にインポートパスを正しく設定するスクリプトを作成：

```python
# scripts/generate_ui.py
import os
import subprocess
from pathlib import Path

def generate_ui_files():
    # プロジェクトルートディレクトリ
    root_dir = Path(__file__).parent.parent

    # UIファイルを検索
    ui_files = root_dir.glob("src/genai_tag_db_tools/gui/**/*.ui")

    for ui_file in ui_files:
        # 出力パスの生成
        output_path = ui_file.parent / f"{ui_file.stem}_ui.py"

        # UI ファイルの生成
        subprocess.run([
            "pyside6-uic",
            str(ui_file),
            "-o",
            str(output_path)
        ])

        print(f"Generated: {output_path}")

if __name__ == "__main__":
    generate_ui_files()
```

### 3. 生成スクリプトの実行

```powershell
# 仮想環境を有効化
.\venv\Scripts\activate

# UIファイルの生成
python scripts/generate_ui.py
```

## インポートパスの設定パターン

### ウィジェット UI の場合：

```
src/genai_tag_db_tools/gui/widgets/
├── tag_cleaner.ui          # Qt Designer ファイル
├── tag_cleaner_ui.py       # 生成されるUIファイル
└── tag_cleaner.py          # 実装ファイル
```

生成される相対インポート：

```python
# tag_cleaner.py
from .tag_cleaner_ui import Ui_TagCleanerWidget
```

### メインウィンドウ UI の場合：

```
src/genai_tag_db_tools/gui/windows/
├── main_window.ui          # Qt Designer ファイル
├── main_window_ui.py       # 生成されるUIファイル
└── main_window.py          # 実装ファイル
```

生成される相対インポート：

```python
# main_window.py
from .main_window_ui import Ui_MainWindow
```

## ビルド時の設定

### pyproject.toml への追加

```toml
[tool.hatch.build.hooks.custom]
dependencies = ["PySide6"]

[tool.hatch.build.targets.wheel.hooks.custom]
dependencies = ["PySide6"]
```

### setup.py を使用する場合（従来型）

```python
from setuptools import setup

setup(
    # ...
    setup_requires=["PySide6"],
    # ...
)
```

## 開発フロー

1. Qt Designer で UI ファイル（.ui）を編集
2. 生成スクリプトを実行して UI ファイル（\*\_ui.py）を更新
3. 実装ファイル（.py）でインポートして使用

## 注意点

- UI ファイルは必ず対応するディレクトリに生成する
- 生成されたファイルは手動編集しない
- Git で UI ファイル（.ui）とソースコード（.py）を管理
- 生成されたファイル（\*\_ui.py）も Git で管理（再現性のため）

## インポートパスの更新

# インポートパスの包括的な更新ガイド

## GUI モジュール間の依存関係

### メインウィンドウでのウィジェットインポート

変更前（gui/MainWindow.py）:

```python
# 古い構造（同一ディレクトリからのインポート）
from TagCleanerWidget import TagCleanerWidget
from TagRegisterWidget import TagRegisterWidget
from TagSearchWidget import TagSearchWidget
from TagStatisticsWidget import TagStatisticsWidget
```

変更後（src/genai_tag_db_tools/gui/windows/main_window.py）:

```python
# 新しい構造（パッケージからの相対インポート）
from ..widgets.tag_cleaner import TagCleanerWidget
from ..widgets.tag_register import TagRegisterWidget
from ..widgets.tag_search import TagSearchWidget
from ..widgets.tag_statistics import TagStatisticsWidget

# または絶対インポート
from genai_tag_db_tools.gui.widgets.tag_cleaner import TagCleanerWidget
from genai_tag_db_tools.gui.widgets.tag_register import TagRegisterWidget
from genai_tag_db_tools.gui.widgets.tag_search import TagSearchWidget
from genai_tag_db_tools.gui.widgets.tag_statistics import TagStatisticsWidget
```

### UI ファイルの参照

変更前（gui/TagCleanerWidget.py）:

```python
# 古い構造
from TagCleanerWidget_ui import Ui_TagCleanerWidget
```

変更後（src/genai_tag_db_tools/gui/widgets/tag_cleaner.py）:

```python
# 同じディレクトリ内のUIファイル
from .tag_cleaner_ui import Ui_TagCleanerWidget
```

### メインウィンドウ UI の参照

変更前（gui/MainWindow.py）:

```python
# 古い構造
from MainWindow_ui import Ui_MainWindow
```

変更後（src/genai_tag_db_tools/gui/windows/main_window.py）:

```python
# 同じディレクトリ内のUIファイル
from .main_window_ui import Ui_MainWindow
```

## 相対インポートと絶対インポート

### 相対インポートの使用

- 同じパッケージ内のモジュール間の参照に適している
- パッケージ名が変更されても影響を受けにくい

```python
# 1階層上のディレクトリ
from ..module import Component

# 2階層上のディレクトリ
from ...package.module import Component

# 同じディレクトリ
from .module import Component
```

### 絶対インポートの使用

- プロジェクト全体からの明示的な参照
- コードの可読性が高い
- IDE の補完が効きやすい

```python
from genai_tag_db_tools.core.processor import CSVToDatabaseProcessor
from genai_tag_db_tools.gui.widgets.tag_cleaner import TagCleanerWidget
```

## テストファイルのインポート

テストディレクトリからのインポート:

```python
# tests/test_gui/test_widgets/test_tag_cleaner.py
from genai_tag_db_tools.gui.widgets.tag_cleaner import TagCleanerWidget
```

## インポート更新のベストプラクティス

1. **一貫性のある命名規則**

   - モジュール名は小文字のスネークケース
   - クラス名はキャメルケース

   ```python
   from genai_tag_db_tools.gui.widgets.tag_cleaner import TagCleanerWidget
   ```

2. **明確なインポート文**

   - ワイルドカードインポート（\*）を避ける
   - 必要なコンポーネントを明示的にインポート

   ```python
   # 非推奨
   from genai_tag_db_tools.gui.widgets.tag_cleaner import *

   # 推奨
   from genai_tag_db_tools.gui.widgets.tag_cleaner import (
       TagCleanerWidget,
       TagCleanerConfig
   )
   ```

3. **循環インポートの防止**
   - 相互依存を避ける
   - 必要な場合は遅延インポートを使用
   ```python
   def get_widget():
       # 必要時に動的インポート
       from genai_tag_db_tools.gui.widgets.tag_cleaner import TagCleanerWidget
       return TagCleanerWidget()
   ```

## インポートパス更新の確認手順

1. **静的解析の実行**

   ```powershell
   # Pylintによるチェック
   pylint src/genai_tag_db_tools
   ```

2. **インポートのテスト**

   ```python
   # test_imports.py
   def test_can_import_all_modules():
       from genai_tag_db_tools.core.processor import CSVToDatabaseProcessor
       from genai_tag_db_tools.gui.windows.main_window import MainWindow
       # 各モジュールのインポートをテスト
   ```

3. **実行時の動作確認**
   ```powershell
   # アプリケーションの起動テスト
   python -m genai_tag_db_tools
   ```

## 移行のまとめ

### 重要なポイント

- `git mv` を使って履歴を保持する
- 段階的に移行してテストを繰り返す
- 問題があればすぐに元に戻せるようにする

### 注意点

- `.gitignore` の変更は既存の追跡状態には影響しない
- 機密データの履歴管理には注意が必要
- 大きなバイナリファイルの追跡はリポジトリサイズに影響する

### 最終確認

1. すべてのファイルが正しい場所にあるか
2. インポートパスが正しく更新されているか
3. アプリケーションが正常に動作するか
4. テストが問題なく実行できるか
