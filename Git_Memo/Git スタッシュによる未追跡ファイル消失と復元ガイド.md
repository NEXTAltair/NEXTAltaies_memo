# Git スタッシュによる未追跡ファイル消失と復元ガイド

# ⚠️ 重要な注意: 未追跡ファイルの消失リスク

`git stash`使用時の重大な注意点：

- 通常の `git stash`は**未追跡ファイルを保存しない**
- `-u`オプションを付け忘れると未追跡ファイルが消失する可能性がある
- ブランチ切り替え時に未追跡ファイルが上書きされる可能性がある

# 安全なスタッシュの手順

```bash
# 必ず事前に状態を確認
git status  # 未追跡ファイルを確認

# 未追跡ファイルを含めてスタッシュ
git stash -u  # または git stash --include-untracked
```

# ファイルが消失した場合の復元手順

## 1. 即時の対応

- **すぐに Git の操作を停止**
- これ以上の操作で状況が悪化する可能性がある
- 作業ディレクトリの状態を保持

## 2. VSCode のローカルヒストリーから復元

1. **フォルダを開く**

   - VSCode で**対象ファイルがあったフォルダ**を開く
2. **ローカルヒストリーを表示**

   - コマンドパレットを開く:
     - Windows/Linux: `Ctrl+Shift+P`
     - Mac: `Cmd+Shift+P`
   - 「Show Local History」を検索して選択
3. **ファイルの検索と復元**

   - サイドバーの「LOCAL HISTORY」パネルで:
     1. フォルダ名を展開
     2. 削除されたファイル名で検索（部分一致可）
     3. タイムスタンプを確認（ファイル消失直前の時刻を目安に）
4. **内容の確認と復元**

   - エントリーを右クリック:
     - 「Compare with Current」で内容を確認
     - 「Restore」で復元
     - 「Open File」で内容を確認してから保存

## 3. バックアップの確認

- VSCode のバックアップフォルダを確認:
  ```
  ~/.vscode-server/backups/  # Linux/Mac
  %APPDATA%\Code\Backups\    # Windows
  ```

## 4. Git のリフロッグ確認

```bash
# 全ての操作履歴を表示
git reflog

# スタッシュの履歴を確認
git fsck --no-reflog | grep blob
```

## 5. システムの一時ディレクトリ確認

- `/tmp`（Linux/Mac）
- `%TEMP%`（Windows）

# 予防策

## 1. スタッシュ使用時の基本ルール

- 必ず `git status`で状態確認
- 未追跡ファイルがある場合は `-u`オプションを使用
- 重要なファイルは事前にバックアップ

## 2. VSCode 設定の調整

- ローカルヒストリーの保存間隔を短く
- バックアップの保持期間を長く
- 自動保存を有効化

## 3. 定期的なバックアップ

- 重要な設定ファイルは定期的にバックアップ
- チーム共有の設定は `*.example`として保存
