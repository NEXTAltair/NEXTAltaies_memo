# pyproject.toml の[tool.pyright]セクションについて

## 概要

`[tool.pyright]`セクションは、Pyright と Pylance（Pyright ベース）の型チェックや解析の動作を設定するために使用する。このセクションはプロジェクトルートの`pyproject.toml`に記述する。

## Language Server との関係

### Python の主要な Language Server

1. **Pylance**

   - Microsoft による TypeScript 製 Language Server
   - Pyright をベースに拡張された機能を提供
   - VS Code/CURSOR との統合が優れている

2. **Pyright**

   - Pylance の基盤となる型チェックエンジン
   - 単体でも Language Server として使用可能
   - CI/CD 環境での型チェックに適している

3. **Jedi Language Server**

   - Pure Python 実装の Language Server
   - 軽量で移植性が高い
   - VS Code 以外のエディタでよく使用される

4. **Python LSP Server**
   - Palantir 社が開発
   - プラグインによる拡張が可能
   - 多様なツール統合をサポート

### Language Server 選択の指針

プロジェクトの特性に応じて最適な Language Server を選択する：

1. **Pylance が適している場合**

   - VS Code/CURSOR を使用
   - パフォーマンスを重視
   - Microsoft のエコシステムを活用

2. **Pyright が適している場合**

   - CI/CD での型チェックが必要
   - エディタに依存しない設定が必要
   - より厳密な型チェックが必要

3. **その他の Language Server が適している場合**
   - 特殊なエディタ環境
   - 最小限の機能のみ必要
   - プラグインによる拡張が必要

## 基本的な設定例

```toml
[tool.pyright]
include = ["src"]
exclude = ["**/node_modules", "**/__pycache__"]
ignore = ["src/oldcode"]
defineConstant = { DEBUG = true }
typeshedPath = "typeshed"
venvPath = "."
venv = "venv"
reportMissingImports = true
reportMissingTypeStubs = false
pythonVersion = "3.12"
pythonPlatform = "Windows"
typeCheckingMode = "basic"
```

## 主要な設定オプション

### パスと環境設定

- `include`: 解析対象のディレクトリやファイル
- `exclude`: 解析から除外するパターン
- `ignore`: 警告を無視するファイルパターン
- `venvPath`: 仮想環境のベースディレクトリ
- `venv`: 使用する仮想環境の名前

### 型チェックの制御

- `typeCheckingMode`: 型チェックの厳密さ
  - `off`: 型チェック無効
  - `basic`: 基本的な型チェック（推奨）
  - `strict`: 厳密な型チェック
- `useLibraryCodeForTypes`: ライブラリコードから型情報を取得

### 警告とエラーの設定

- `reportMissingImports`: 見つからないインポートを報告
- `reportMissingTypeStubs`: スタブファイルの不足を報告
- `reportGeneralTypeIssues`: 一般的な型の問題を報告
- `reportOptionalMemberAccess`: Optional 型のメンバーアクセスを報告

## GUI 開発での推奨設定

PySide6 を使用した GUI アプリケーション開発では、以下の設定が推奨される：

```toml
[tool.pyright]
include = ["src"]
exclude = ["**/__pycache__", "test"]
venvPath = "."
venv = "venv"
typeCheckingMode = "basic"

# PySide6関連の設定
reportGeneralTypeIssues = "warning"
reportMissingTypeStubs = false
reportUnknownMemberType = false

# パス設定
extraPaths = ["src/gui"]
```

## Pylance との関係

Pylance は Pyright をベースにしており、これらの設定の多くは Pylance でも有効。ただし、VS Code/CURSOR で Pylance を使用する場合は、settings.json での設定が優先される。

### settings.json での対応する設定

```json
{
  "python.analysis.typeCheckingMode": "basic",
  "python.analysis.extraPaths": ["src/gui"],
  "python.analysis.diagnosticMode": "workspace",
  "python.analysis.autoImportCompletions": true
}
```

## Language Server 設定の優先順位

1. **VS Code/CURSOR 環境での優先順位**

   ```
   settings.json > pyproject.toml > デフォルト設定
   ```

2. **その他のエディタでの優先順位**

   ```
   エディタ固有の設定 > pyproject.toml > デフォルト設定
   ```

3. **CI/CD 環境での優先順位**
   ```
   CI設定ファイル > pyproject.toml > デフォルト設定
   ```

## CI/CD 環境での使用

CI/CD 環境で Pyright を使用する場合の設定例：

```toml
[tool.pyright]
include = ["src"]
exclude = ["test"]
typeCheckingMode = "strict"
reportMissingImports = true
reportMissingTypeStubs = true
reportGeneralTypeIssues = true
```

## エディタ別の注意点

### VS Code / CURSOR

- Pylance が推奨される
- settings.json の設定が優先される
- pyproject.toml の設定は補助的に使用

### その他のエディタ

- Pyright を直接使用する場合は pyproject.toml の設定が重要
- エディタ固有の設定との整合性に注意

## トラブルシューティング

### よくある問題と解決策

1. 型チェックが厳しすぎる

```toml
[tool.pyright]
typeCheckingMode = "basic"
reportOptionalMemberAccess = false
```

2. PySide6 の型情報エラー

```toml
[tool.pyright]
reportUnknownMemberType = false
reportUnknownArgumentType = false
```

3. サードパーティライブラリの型エラー

```toml
[tool.pyright]
reportMissingTypeStubs = false
reportUnknownVariableType = false
```

## Language Server の選択と設定の推奨フロー

1. **開発環境の評価**

   - 使用するエディタの確認
   - プロジェクトの規模と要件の確認
   - チーム開発かソロ開発かの確認

2. **Language Server の選択**

   - VS Code/CURSOR → Pylance
   - CI/CD 重視 → Pyright
   - クロスプラットフォーム → Jedi/Python LSP Server

3. **設定ファイルの優先順位決定**

   - Pylance 使用時は settings.json 重視
   - その他は pyproject.toml 重視

4. **設定の最適化**
   - プロジェクトの特性に応じた警告レベルの調整
   - パフォーマンスとのバランス考慮
   - チーム内での設定の統一

## まとめ

- Language Server の選択が設定方法に大きく影響
- 小規模プロジェクトでは基本設定で十分
- GUI アプリケーションでは警告レベルを調整
- Pylance ユーザーは settings.json での設定を優先
- CI/CD では厳密な設定を検討
