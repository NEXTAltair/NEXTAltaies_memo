# 1. ディレクトリ構造変更の影響範囲

## 追跡されているファイル

- Git は変更を正しく追跡可能
- `git mv` コマンドで履歴を保持したまま移動

## 未追跡ファイル

- 通常は影響を受けない
- ただし以下の場合に問題が発生：
  1. 移動先に同名のファイル/ディレクトリが存在
  2. 親ディレクトリの構造が大きく変わる
  3. 移動元のパスが別の用途で使用される

# 2. 安全な変更手順

## 準備段階

```bash
# 1. 現在の状態を確認
git status

# 2. 未追跡ファイルをリストアップ
git clean -n

# 3. 重要なファイルをバックアップ
tar czf ../backup.tar.gz .
```

## 実行段階

```bash
# 1. 新しいブランチを作成
git checkout -b restructure-directories

# 2. ディレクトリ構造を変更
git mv old/path new/path

# 3. 関連する設定ファイルを更新
git grep -l 'old/path' | xargs sed -i 's|old/path|new/path|g'

# 4. 変更をコミット
git add .
git commit -m "Restructure directories: Move files from old/path to new/path"
```

## 検証段階

```bash
# 1. ビルドやテストの実行
make test

# 2. 未追跡ファイルの確認
git status

# 3. パスの参照が正しく更新されているか確認
git grep 'old/path'
```

# 3. 注意が必要なファイル

## 設定ファイル

- テンプレートと実際の設定ファイルの場所を統一
- 相対パスの参照を更新
- 環境変数での参照を検討

## ビルド関連ファイル

- Makefile やビルドスクリプト
- 依存関係の設定
- コンパイル出力先の設定

## ドキュメント

- README.md などの説明ファイル
- インストール手順
- ディレクトリ構造の説明

# 4. よくある問題と解決策

## 問題 1：パス参照の更新漏れ

```bash
# すべてのファイルでのパス参照を確認
find . -type f -exec grep -l "old/path" {} \;

# 一括置換（慎重に！）
find . -type f -exec sed -i 's|old/path|new/path|g' {} \;
```

## 問題 2：未追跡ファイルの消失

```bash
# 事前にリストアップ
find . -type f -not -path "*.git/*" > tracked_files.txt
git ls-files > git_files.txt
diff tracked_files.txt git_files.txt
```

## 問題 3：異なるブランチ間での競合

```bash
# ブランチ間の差分を確認
git diff main:old/path feature:new/path

# マージ時の注意点を.gitattributesに記載
echo "new/path/* merge=ours" >> .gitattributes
```

# 5. ベストプラクティス

1. **段階的な変更**

   - 小さな変更に分割
   - 各段階でテストを実行
   - こまめにコミット

2. **パス参照の管理**

   - 相対パスより環境変数を使用
   - 設定ファイルでパスを一元管理
   - シンボリックリンクの活用

3. **ドキュメント化**

   ```markdown
   # ディレクトリ構造変更履歴

   # 2024-04-28

   - `/config.ini` → `/config/config.ini`
   - 理由：設定ファイルの集約化
   - 影響：環境変数 `CONFIG_PATH` の導入
   ```

4. **レビュー時のチェックポイント**
   - [ ] すべてのパス参照が更新されているか
   - [ ] テストが正常に通るか
   - [ ] ドキュメントが更新されているか
   - [ ] 未追跡ファイルへの影響は考慮されているか

# 6. 自動化の提案

## スクリプト例：パス更新チェッカー

```bash
#!/bin/bash
# check_paths.sh

old_path="$1"
new_path="$2"

# 古いパスの参照を検索
echo "Checking for old path references..."
git grep -l "$old_path"

# 設定ファイルの確認
echo "Checking config files..."
find . -name "*.conf" -o -name "*.ini" -exec grep -l "$old_path" {} \;

# ビルドファイルの確認
echo "Checking build files..."
find . -name "Makefile" -o -name "*.cmake" -exec grep -l "$old_path" {} \;
```

# まとめ

ディレクトリ構造の変更は慎重に行う必要があります：

1. 事前の影響範囲の調査
2. 段階的な変更の実施
3. 徹底的なテストとレビュー
4. 明確なドキュメント化
5. 自動化ツールの活用

これらの手順を守ることで、安全で確実なディレクトリ構造の変更が可能になります。
