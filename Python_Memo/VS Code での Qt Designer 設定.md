# VS Code での Qt Designer 設定

結論 Qt All Extensions Pack は使わない

Qt C++ は使ったことがないし Qt UI を Qt Designerで開けない

Qt Core と Qt Qml だけをインストールする

## 拡張機能の設定

### Qt for Python 拡張機能

Qt All Extensions Pack はなぜか使えない

なぜかというか基本が C++実装だから Python はおまけでサポートが薄いんだろう

This project is deprecated. Use [the official Qt Extensions](https://marketplace.visualstudio.com/items?itemName=TheQtCompany.qt) instead.って書いてあるけど採取更新

## 注意点

- UI ファイルは必ず対応するディレクトリに配置する
- コンパイル後の Python ファイルは同じディレクトリに生成される
- VS Code 拡張機能の設定はワークスペースごとに行う

## 推奨される作業フロー

1. Qt Designer で UI ファイルを編集（.ui）
2. VS Code 拡張機能で自動コンパイル、または手動でコンパイル実行
3. 生成された UI ファイル（\*\_ui.py）を確認
4. インポートパスが正しいことを確認

## トラブルシューティング

### UI ファイルが認識されない場合

**Qt All Extensions Pack を アンインストールする**

- ファイルパスとディレクトリ構造を確認
- VS Code 拡張機能の設定を確認
- Python 環境の PySide6 インストールを確認

### コンパイルエラーが発生する場合

- PySide6 のバージョンを確認
- UI ファイルの構文エラーを確認
- 出力ディレクトリの書き込み権限を確認
