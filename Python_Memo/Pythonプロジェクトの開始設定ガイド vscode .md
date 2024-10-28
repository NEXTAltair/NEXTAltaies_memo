# 1. プロジェクト構造の作成

基本的なディレクトリ構造：

プロジェクトとパッケージの違い俺はわかってない

```
your-project/
├── src/
│   └── your_package/
│       ├── __init__.py
│       ├── main.py
│       └── exceptions.py
├── test/
│   ├── __init__.py
│   └── test_main.py
├── .vscode/
│   └── settings.json
└── pyproject.toml
```

# 2. 仮想環境のセットアップ

```bash
# 仮想環境の作成
python -m venv venv

# 仮想環境の有効化（Windows）
venv\Scripts\activate

# 仮想環境の有効化（Unix系）
source venv/bin/activate
```

# 3. pyproject.toml の設定

最小限の設定例：

```toml
[build-system]
requires = ["hatchling"]
build-backend = "hatchling.build"

[project]
name = "your-package"
version = "0.1.0"
description = "Your package description"
requires-python = ">=3.12.4"
dependencies = [
    "requests>=2.31.0",  # 必要な依存関係
]
classifiers = [
    "Operating System :: Microsoft :: Windows :: Windows 11"  # 開発環境情報
]

[tool.pytest.ini_options]
addopts = "-ra -q -v"
testpaths = ["test",]
pythonpath = ["src"]

[project.optional-dependencies]
dev = ["pytest>=8.3.3","pytest-cov>=5.0.0"]

[tool.hatch.build.targets.wheel]
packages = ["src"]  # パッケージのソースディレクトリを指定
```

# 4. VSCode 設定

`.vscode/settings.json`:

```json
{
  // Python設定
  "python.defaultInterpreterPath": "${workspaceFolder}/venv/Scripts/python.exe",

  // パス設定
  "python.analysis.extraPaths": ["${workspaceFolder}/src"],

  // テスト設定
  "python.testing.pytestEnabled": true,
  "python.testing.unittestEnabled": false,
  "python.testing.pytestArgs": ["test"],

    // Pylint設定
    "pylint.enabled": true,
    "pylint.args": [
        "--init-hook",
        "import sys; sys.path.append('src')", # パスを追加してないとインタプリタでインポートできててもpylintがインポートできてない言い出す
	"--disable=W1203" # ロガーで%s 使えという特に意味のない指示(議論が分かれるらしい)を無視
    ]
}
```

# 5. パッケージのインストール

[project.optional-dependencies] で設定したやつが ".[dev]"

今のところ用があったことはない

```bash
# 開発モードでインストール（基本パッケージのみ）
pip install -e .

# または、開発用依存関係も含めてインストール
pip install -e ".[dev]"
```

# 6. インポートパスの設定

プロジェクト内でのインポート例：

```python
# 正しいインポート方法
from your_package.exceptions import CustomError
from your_package.submodule import some_function

# サブパッケージからのインポート
from your_package.subpackage.module import SubClass

# 相対インポートを使用する場合
from ..exceptions import CustomError  # 2階層上から
from .module import function         # 同じディレクトリから
```

# 7. コケたポイントと解決方法

## ImportError: No module named 'exceptions'

- 原因: 絶対インポートパスが正しくない
- 解決: パッケージ名を含めた完全なパスでインポート

```python
# 誤
from exceptions import CustomError
# 正
from your_package.exceptions import CustomError
```

## ModuleNotFoundError

- 原因: パッケージが PYTHONPATH に追加されていない
- 解決:
  1. `pip install -e .` でパッケージをインストール
  2. VSCode の設定で extraPaths を確認

## pytest でテストが見つからない

- 原因: テストディレクトリのパスが正しく設定されていない
- 解決:
  1. `pyproject.toml` の `testpaths` を確認
  2. VSCode の `pytestArgs` を確認

# 8. 開発フロー

1. プロジェクト構造の作成
2. 仮想環境のセットアップ
3. 基本設定ファイルの作成
4. パッケージのインストール
5. VSCode 設定の確認
6. テストの実行確認

# 注意点

1. パッケージ名の一貫性

   - `pyproject.toml` の名前の変更はしない､できない
   - ディレクトリ構造
   - インポートパス
     が一致していることを確認
2. 仮想環境

   - プロジェクトごとに独立した仮想環境を使用
   - 仮想環境の有効化を忘れずに
3. gitignore

```gitignore
# 最小限の設定
__pycache__/
*.py[cod]
venv/
.env
.vscode/
```

4. インポート
   - 可能な限り絶対インポートを使用
   - 相対インポートを使用する場合は明示的に

この設定により、開発環境の一貫性が保たれ、チーム開発やパッケージの配布が容易になります。

# **参考**

1. https://nikkie-ftnext.hatenablog.com/entry/why-dont-you-write-pyproject-toml-instead-of-setup-py
2. https://nikkie-ftnext.hatenablog.com/entry/pyproject-toml-project-keys-and-examples
3. https://zenn.dev/karaage0703/articles/db8c663640c68b
4. https://data.gunosy.io/entry/linter_option_on_pyproject
5. https://github.com/pfnet/pysen
