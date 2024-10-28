# VSCode ワークスペースファイルの Git 競合解決ガイド

# 発生した問題

`add-network-error-handling`ブランチで大規模な変更を行った後、以下の流れでエラーが発生：

1. main ブランチにチェックアウト（成功）
2. dev ブランチへのチェックアウトを試みた際に失敗

## エラーメッセージ

```
error: The following untracked working tree files would be overwritten by checkout:
    NatureRemoMatterControl.code-workspace
Please move or remove them before you switch branches.
Aborting
```

# エラーの原因

## ワークスペースファイルの競合

- 未追跡（untracked）のワークスペース設定ファイル（`.code-workspace`）が存在
- このファイルが切り替え先のブランチでも異なる内容で存在
- 未追跡ファイルが上書きされる可能性があるため、Git が安全のために切り替えを中止

## バージョン管理の経緯

1. **Ver1（最初）**: 基本設定のみ

   - プロジェクトのパス設定
   - 基本的な VSCode 設定
2. **Ver2**: Python 開発環境の設定追加

   - venv 関連の設定
   - デバッグ設定の追加
   - Python エクステンション設定
3. **Ver3**: プロジェクト構成の更新

   - フォルダ構造の整理
   - デバッグ設定の最適化
   - パス設定の更新
4. **Ver4**: ネットワークエラー処理のための変更

   - デバッグ設定の更新
   - 新規エラー処理用の設定追加
   - テスト設定の変更

# 解決方法

## 1. .gitignore の設定

プロジェクト固有の除外設定を `.gitignore`に追加：

```
# .gitignoreに以下を追加
venv/
*.code-workspace  # VSCode workspace設定も除外
.vscode/         # VSCode設定フォルダも除外推奨
```

## 2. 未追跡ファイルの退避

作業中の変更を一時的に退避：

```sh
git stash -u  # -uオプションで未追跡ファイルも含める
```

## 3. ブランチの切り替え

```sh
git checkout dev  # 目的のブランチに切り替え
```

## 4. 退避した変更の復元

```sh
git stash pop
```

## stash pop でエラーが発生した場合

以下のようなエラーが出る場合がある：

```
error: Your local changes to the following files would be overwritten by merge:
    NatureRemoMatterControl.code-workspace
Please commit your changes or stash them before you merge.
Aborting
```

この場合の対応方法：

1. **現在の変更を別のスタッシュとして保存**

   ```sh
   git stash save "current-changes"
   ```

   エラーメッセージが出る場合：

   ```
   No local changes to save
   ```

   これは現在のワーキングディレクトリに保存すべき変更がない状態。
   以下を確認：

   - `git status`で未追跡ファイルの有無を確認
   - 未追跡ファイルがある場合は `git stash -u save "current-changes"`を使用
2. **スタッシュの状態を確認**

   ```sh
   git stash list  # スタッシュの一覧を表示
   # 表示例：
   # stash@{0}: On dev: current-changes
   # stash@{1}: WIP on main: 1234567 Initial commit
   ```
3. **元のスタッシュを適用**

   ```sh
   git stash pop stash@{1}  # 最初に保存したスタッシュを適用
   ```

   エラーメッセージが出る場合：

   ```
   error: unknown switch `e'
   usage: git stash pop [--index] [-q | --quiet] [<stash>]
        -q, --[no-]quiet      be quiet, only report errors
        --[no-]index          attempt to recreate the index
   ```

   これは PowerShell で `@`記号が特別な意味を持つために発生するエラー。
   以下のいずれかの方法で対処：

   1. **引用符で囲む**

   ```sh
   git stash pop "stash@{1}"
   ```

   2. **スタッシュ ID を使用**

   ```sh
   # まずスタッシュリストでIDを確認
   git stash list
   # 数字のみで指定
   git stash pop 1
   ```
4. **新しいスタッシュを適用**

   ```sh
   git stash pop stash@{0}  # 現在の変更を戻す
   ```

または、より安全な方法として：

1. **スタッシュを適用せずに確認**

   ```sh
   git stash show -p  # 変更内容を確認
   ```
2. **必要な変更を手動で適用**

   - 変更内容を確認しながら、必要な部分のみを手動で反映
   - 特に `.code-workspace`ファイルは慎重に変更を統合

# 推奨される管理方法

## 1. テンプレートの使用

```
project-root/
├── .gitignore
├── workspace.code-workspace.example  # 共有用テンプレート
└── .vscode/
    └── settings.json.example        # 共有用設定テンプレート
```

## 2. チーム内での運用ルール

1. **テンプレートのコミット**:

   - `*.example`ファイルを Git で管理
   - プロジェクトの基本設定のみを含める
2. **個人設定の管理**:

   - テンプレートをコピーしてローカルで使用

   ```sh
   cp workspace.code-workspace.example workspace.code-workspace
   ```

## 3. 予防策

1. **ブランチ切り替え前の確認**:
   ```sh
   git status  # 未追跡ファイルの確認
   ```
2. **定期的なテンプレート更新の確認**
