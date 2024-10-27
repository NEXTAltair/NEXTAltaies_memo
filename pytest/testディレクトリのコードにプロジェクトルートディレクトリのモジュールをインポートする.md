# Pytest でプロジェクトモジュールをインポートする方法

## 俺は｢conftest.py｣か.vscode/settings.json しか使ったことはない

Pytest でテストを書く際、テストディレクトリからプロジェクトのソースコードをインポートする方法は複数存在する。
それぞれの方法には長所と短所があるため、プロジェクトの要件に応じて適切な方法を選択する必要がある。

## 実装方法の比較

### 1. pyproject.toml を使用する方法（モダンな方法）

```toml
# pyproject.toml
[build-system]
requires = ["hatchling"]
build-backend = "hatchling.build"

[project]
name = "your-package"
version = "0.1.0"
dependencies = [
    # 依存パッケージをリスト
]
```

インストール:

```bash
pip install -e .
```

#### メリット

- 最新の Python パッケージング標準に準拠
- setup.py よりも簡潔で読みやすい
- 依存関係の管理が容易
- poetry などのモダンなパッケージマネージャーと相性が良い

#### デメリット

- 比較的新しい方法のため、古い環境では対応していない可能性がある

### 2. setup.py を使用する方法（標準的な方法）

```python
# setup.py
from setuptools import setup, find_packages

setup(
    name="your-package",
    version="0.1.0",
    packages=find_packages(),
    install_requires=[
        # 依存パッケージをリスト
    ],
)
```

インストール:

```bash
pip install -e .
```

#### メリット

- Python の標準的なパッケージング方法に準拠
- 依存関係の管理が容易
- 開発モード（-e）でインストールすることで、ソースコードの変更がすぐに反映される
- テスト環境と本番環境で同じ方法でインポートできる

#### デメリット

- 初期設定に少し手間がかかる
- パッケージング関連の知識が必要

### 3. VSCode での設定方法（IDE 統合アプローチ）

#### settings.json での設定

```jsonc
// .vscode/settings.json
{
  "python.analysis.extraPaths": ["${workspaceFolder}"],
  "python.testing.pytestEnabled": true,
  "python.testing.unittestEnabled": false,
  "python.testing.nosetestsEnabled": false,
  "python.testing.pytestArgs": ["tests"]
}
```

#### workspace 設定ファイル

```jsonc
// your-project.code-workspace
{
  "folders": [
    {
      "path": "."
    }
  ],
  "settings": {
    "python.analysis.extraPaths": ["${workspaceFolder}"],
    "python.testing.pytestEnabled": true,
    "python.testing.pytestArgs": ["tests"]
  }
}
```

#### メリット

- IDE の機能として完結
- チーム内で設定を共有しやすい
- インテリセンスやコード補完が正しく機能する
- プロジェクトごとに設定を分けられる

#### デメリット

- VSCode 依存の解決方法
- CI/CD など、IDE 外での実行時は別の設定が必要
- チームメンバーが異なる IDE を使用している場合は統一が難しい

### 4. conftest.py で sys.path を設定する方法（シンプルな方法）

```python
# tests/conftest.py
import sys
from pathlib import Path

def pytest_configure():
    """Pytest設定フック"""
    project_root = Path(__file__).resolve().parent.parent
    if project_root.exists():
        sys.path.insert(0, str(project_root))
    else:
        raise RuntimeError(f"Project root directory not found: {project_root}")
```

#### メリット

- 設定が簡単で直感的
- conftest.py は Pytest が自動的に読み込むため、追加の設定が不要
- プロジェクト固有の設定をまとめて管理できる

#### デメリット

- sys.path を直接操作するのはあまり推奨されない
- 環境依存の問題が発生する可能性がある
- パッケージとして配布する際に問題が生じる可能性がある

## プロジェクト構造に関する補足

推奨されるプロジェクト構造:

```
project/
├── src/
│   └── your_package/
│       ├── __init__.py
│       └── module.py
├── tests/
│   ├── __init__.py
│   └── test_module.py
├── setup.py
└── pyproject.toml
```

この structure を採用することで:

- ソースコードとテストが明確に分離される
- インポートパスが明確になる
- パッケージ化が容易になる

## 結論

複数の方法の中から、以下の優先順位で選択することを推奨:

1. pyproject.toml + src レイアウト（モダンで最も推奨）
2. setup.py（標準的で実績のある方法）
3. VSCode 設定（IDE 統合が重要な場合）
4. conftest.py + sys.path（小規模プロジェクトのみ）

選択の基準:

- プロジェクトの規模
- チームの技術スタック
- 配布の必要性
- 開発環境の制約
- IDE/エディタの統一度

これらの要因を考慮して、最適な方法を選択することが重要。
また、pyproject.toml + VSCode 設定を組み合わせることで、より堅牢な開発環境を構築することができる。
この組み合わせが優れている理由：

1. 開発体験の向上

   - VSCode 設定により、開発時の補完やリンティングが正確に機能
   - デバッグやテスト実行が IDE から直接可能
   - エディタ上でのインポートエラーが適切に検出される
2. 本番環境での信頼性

   - pyproject.toml による正式なパッケージング
   - CI/CD 環境での安定した動作
   - 依存関係の明示的な管理
3. チーム開発での一貫性

   - プロジェクトのビルド設定が pyproject.toml で統一
   - VSCode 設定をバージョン管理することで開発環境も統一
   - 他の IDE ユーザーも pyproject.toml があれば問題なく開発可能
4. 保守性の向上

   - パッケージの依存関係を一箇所（pyproject.toml）で管理
   - IDE 設定と実行環境の設定が分離されており、それぞれ独立してメンテナンス可能
   - アップグレードパスが明確

このように、開発効率と信頼性の両方を高いレベルで実現できる。

## pyproject.toml + VSCode の実装手順

### 1. プロジェクト構造の作成

まず、以下のような構造でプロジェクトを作成する：

```
your-project/
├── src/
│   └── your_package/
│       ├── __init__.py
│       └── main.py
├── tests/
│   ├── __init__.py
│   └── test_main.py
├── .vscode/
│   └── settings.json
├── pyproject.toml
└── README.md
```

### 2. pyproject.toml の設定

[NatureRemoMatterControl](https://github.com/NEXTAltair/NatureRemoMatterControl/tree/add-network-error-handling)

pyproject.toml の名前は変えない

```toml
[build-system]
requires = ["hatchling"]
build-backend = "hatchling.build"

[tool.hatch.build.targets.wheel]
packages = ["src"]

[project]
name = "NatureRemoMatterControl"
version = "0.1.0"
description = "Nature Remo E で潮流を監視して TP-Link スマートプラグを制御する"
requires-python = ">=3.12.4"
dependencies = ["pytest>=8.3.3"]

# 追加: 開発環境情報
classifiers = ["Operating System :: Microsoft :: Windows :: Windows 11"]

[tool.pytest.ini_options]
addopts = "-ra -q -v"
testpaths = ["test"]
pythonpath = ["src"]

[project.optional-dependencies]
dev = ["pytest>=8.3.3", "pytest-cov>=5.0.0"]


```

### 3. VSCode 設定の実装

`.vscode/settings.json`:

```jsonc
{
    // Python設定
    "python.defaultInterpreterPath": "${workspaceFolder}/venv/Scripts/python.exe",

    // パス設定（統合）
    "python.analysis.extraPaths": [
        "${workspaceFolder}/src",
        "${workspaceFolder}/test"  // もしテストモジュールのインポートも必要なら
    ],

    // テスト設定
    "python.testing.pytestEnabled": true,
    "python.testing.unittestEnabled": false,
    "python.testing.pytestArgs": ["test"],  // あるいは "tests" - プロジェクト構造に合わせる

    // Pylint設定
    "pylint.enabled": true,
    "pylint.args": [
        "--init-hook",
        "import sys; sys.path.append('src')"
    ],
    // エディタ設定
    "editor.formatOnSave": true,
    "editor.codeActionsOnSave": {
        "source.fixAll.eslint": "explicit",
        "source.fixAll.stylelint": "always"
    }
}
```

### 4. 開発環境のセットアップ

1. 仮想環境の作成と有効化：

```bash
# 仮想環境の作成
python -m venv .venv

# 有効化（Unix系）
source .venv/bin/activate
# 有効化（Windows）
.\.venv\Scripts\activate
```

2. 開発用パッケージのインストール：

```bash
# pyproject.tomlからインストール（開発用依存関係含む）
pip install -e ".[dev]"
```

### 5. テストコードの実装例

`src/your_package/main.py`:

```python
def add_numbers(a: int, b: int) -> int:
    return a + b
```

`tests/test_main.py`:

```python
from your_package.main import add_numbers

def test_add_numbers():
    assert add_numbers(1, 2) == 3
```

### 6. テスト実行の確認

1. VSCode のテストエクスプローラーから実行：

   - サイドバーのテストアイコンをクリック
   - テストを選択して実行
2. コマンドラインから実行：

```bash
pytest tests/
```

### 7. トラブルシューティング

よくある問題と解決方法：

1. インポートエラー

   - `src`ディレクトリが PYTHONPATH に含まれていることを確認
   - VSCode で正しい Python 環境が選択されているか確認
2. テストが見つからない

   - `pytest.ini_options`の設定を確認
   - テストファイル名が `test_`で始まっているか確認
3. VSCode の設定が反映されない

   - コマンドパレット（Ctrl+Shift+P）から「Python: Select Interpreter」を実行
   - VSCode を再起動

## 参考

- 【[VScode】pytest による単体テスト実行 &amp; カバレッジ可視化方法](https://qiita.com/moshi/items/21a6ff0a20cd840f71ea)
- [Python における import の仕組みと Pytest](https://zenn.dev/panyoriokome/articles/f34ae72cc33150)
- [python で pytest を使ってテスト駆動開発するときのディレクトリ構造](https://qiita.com/ShortArrow/items/32fa54b2c32e09355fed)
- [pytest を実行するための Python パッケージのディレクトリ構成](https://helve-blog.com/posts/python/pytest-directory/)
-
