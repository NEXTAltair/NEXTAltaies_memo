# Python モダンパッケージング メモ

## トラブルシューティングの経験

### 遭遇した問題と解決までの道のり

1. **初期状態**

   - ローカルパッケージを `pip install -e .`でインストール
   - エラーなくインストール完了
   - `pip list`でもパッケージが表示される
   - しかし、Python からモジュールとして認識されない

2. **誤った解決策の例**

   - GitHub Copilot の提案：
     - `sys.path`への追加を提案 → パッケージングの問題を回避するだけの対症療法
     - `settings.json`の修正を提案 → VSCode の設定であって Python のパッケージ認識とは無関係
     - これらはパッケージングの本質的な問題解決にならない

3. **GitHub Copilot 命名規則に関するハルシネーション**

   - パッケージ名のハイフン(`-`)が問題視される
   - 実際の規則：
     - PyPI パッケージ名（配布名）：ハイフン(`-`)推奨
     - Python モジュール名：アンダースコア(`_`)必須

   ```toml
   [project]
   name = "my-package"  # PyPI/pip用の名前（ハイフン推奨）

   [tool.hatch.build.targets.wheel]
   packages = ["my_package"]  # Pythonモジュール名（アンダースコア必須）
   ```

4. **`Claud 3.5 sonnnet' による 'project.scripts`セクションのハルシネーション**

   - コマンドライン実行用の設定、モジュールのインポートとは無関係

   ```toml
   [project.scripts]
   my-command = "my_package.main:main"  # CLIコマンドの定義
   ```

## 正しいパッケージング構造

### モダンな構成（推奨）

```
my-package/
├── my_package/
│   ├── __init__.py
│   ├── main.py
│   └── core/
├── tests/
└── pyproject.toml
```

### 非推奨だが見かける構成（src layout）

```
my-package/
├── src/
│   └── my_package/
│       ├── __init__.py
│       └── main.py
├── tests/
└── pyproject.toml
```

src layout が非推奨な理由：

- 余分な階層が増える
- 現代のビルドツールでは不要な分離
- 歴史的な `setup.py`時代の名残

  この非推奨構成は Sonnnet が勧めてきたやつなんだけど

## 重要なポイント

### パッケージ名の使い分け

1. **配布パッケージ名**（PyPI、pip install 用）

   - ハイフン(`-`)推奨
   - 例：`my-awesome-package`

2. **Python モジュール名**（import 文で使用）

   - アンダースコア(`_`)必須
   - 例：`import my_awesome_package`

### pyproject.toml の重要設定

```toml
[project]
name = "my-package"  # 配布名

[tool.hatch.build.targets.wheel]
packages = ["my_package"]  # モジュール名
```

### ビルドシステム

- モダンなビルドバックエンド（hatchling 等）を使用
- `setup.py`は非推奨

## デバッグのヒント

1. パッケージがインストールされているか確認

   ```bash
   pip list | grep my-package
   ```

2. モジュールの検索パスを確認

   ```python
   import sys
   print(sys.path)
   ```

3. パッケージの内容を確認

   ```bash
   pip show my-package
   ```

## 一般的な落とし穴

1. src layout の無用な使用
2. パッケージ名とモジュール名の混同
3. `setup.py`時代の古い情報に基づく実装
4. システムパスの強制的な操作による対症療法

## TOML 書き換える

修正のポイント：

1. 現在の `src`ディレクトリの内容をプロジェクトルートに移動：
   git 忘れてうっかりコミットするとファイル消えるからな！！

```bash
git mv src/genai_tag_db_tools ./
```

2. 移動させたファイルをコミット

3. `src` は空なので消し方は好きにやる

```bash
rm -r src
```

4. UI ファイルの整理：

```bash
# designerディレクトリのUIファイルをメインソースとして使用
# widgetsディレクトリの重複ファイルを削除
```

5. TOML の編集

ようわからんから 3.5 sonnnet にコメントつけてもらってもよくわからん

```toml
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
pythonpath = ["genai_tag_db_tools"]          # テスト実行時のPythonパス

# 開発時のみ必要な追加パッケージ
[project.optional-dependencies]
dev = [
    "pytest>=8.3.3",          # テストフレームワーク
    "pytest-cov>=5.0.0",      # テストカバレッジツール
]

# ビルド設定
[tool.hatch.build.targets.wheel]
packages = ["genai_tag_db_tools"]            # パッケージ名アンダースコア推奨

# プロジェクトスクリプト
[project.scripts]
genai-tag-db-tools = "genai_tag_db_tools.main:main"  #インストールしてgenai_tag_db_toolswpターミナルで入力するとmainが実行されるよという話

# Pyright（静的型チェッカー）の設定
[tool.pyright]
include = ["genai_tag_db_tools"]             # 型チェック対象
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
```

6. venv を再作成

一度完全に消してからじゃなくてもパッケージのアンインストールか上書きでも十分じゃね？

```bash
pip uninstall genai-tag-db-tools
```

```bash
py -3.12 -m venv venv
```

7. ローカルパッケージを dev でインストール

```bash
pip install -e ".[dev]"
```

8. チェックする

配布パッケージ名

```bash
pip list | grep genai-tag-db-tools
```

grep : The term 'grep' is not recognized as the name of a cmdlet, function, script file, or operabl
e program. Check the spelling of the name, or if a path was included, verify that the path is corre ct and try again.

とか言われるので

```PowerShell
pip list | sls genai-tag-db-tools
```

```python
import sys
print(sys.path)
```

9. パッケージスクリプトを走らせる

```bash
genai-tag-db-tools
```

```
ModuleNotFoundError: No module named 'genai_tag_db_tools.main'
```

main.py 入れ忘れ

### プロジェクトルートから main.py を移動
