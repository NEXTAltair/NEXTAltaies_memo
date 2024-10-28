# PySide6 インストールエラー：[WinError 5] アクセスが拒否されました

## エラーの概要

```
ERROR: Could not install packages due to an OSError: [WinError 5] アクセスが拒否されました。:
'h:\\lora\\素材リスト\\スクリプト\\src\\module\\genai-tag-db-tools\\venv\\lib\\site-packages\\pyside6\\msvcp140.dll'
```

## 考えられる原因と解決方法

### 1. プロセスによるファイルロック

> **重要**: プロセス終了時の"Access is denied"エラーについて
>
> 通常の PowerShell セッションでは、他のユーザーや特権プロセスが起動した Python プロセスを終了できません。
> これは主に以下の理由で発生します：
>
> - IDE やバックグラウンドサービスから起動されたプロセス
> - 他のユーザーアカウントで実行されているプロセス
> - システムプロセスとして実行されている Python スクリプト
>
> 推奨される対処順序：
>
> 1. まず IDE や Python アプリケーションを正常終了
> 2. タスクマネージャーから Python プロセスを確認・終了
> 3. 最後の手段として管理者権限でのコマンド実行

#### 原因

- VSCode、Python インタープリタ、または他のプロセスが`msvcp140.dll`を使用中
- PySide6 を使用しているアプリケーションが実行中

#### 解決方法

1. すべての Python プロセスを終了

   ```powershell
   # 実行中のPythonプロセスを確認
   Get-Process python* | Select-Object Id, ProcessName

   # 実行結果例：
   #    Id ProcessName
   #    -- -----------
   #  7888 python
   #  8108 python
   # 11884 python
   # （以下同様に複数のプロセスが表示される）

   # 強制終了（必要な場合）
   Get-Process python* | Stop-Process -Force

   # エラー例：
   # Stop-Process : Cannot stop process "python (7888)" because of the following error: Access is denied
   # このエラーが発生する場合は、管理者権限が必要です。以下の手順で実行してください：

   # 1. PowerShellを管理者として実行
   # 2. または、以下のコマンドを使用（より安全な方法）
   Get-Process python* | ForEach-Object {
       try {
           $_.CloseMainWindow()
           Start-Sleep -Milliseconds 100
           if (!$_.HasExited) {
               $_ | Stop-Process -Force
           }
       } catch {
           Write-Warning "プロセスID $($_.Id)の終了に失敗: $_"
       }
   }

   # 実行結果例：
   # False
   # False
   # False
   #
   # "False"はウィンドウを持たないバックグラウンドプロセスを示します。
   # この場合、Stop-Process -Forceでの強制終了が試みられます。

   # 注意: VSCodeやPyCharmなどのIDEから起動されたPythonプロセスは
   # IDE自体を終了する必要がある場合があります。
   ```

2. VSCode を完全に終了

   ```powershell
   # VSCodeプロセスを確認
   Get-Process code* | Select-Object Id, ProcessName

   # 強制終了（必要な場合）
   Get-Process code* | Stop-Process -Force
   ```

### 2. 権限の問題

#### 原因

- ユーザーアカウントに必要な書き込み権限がない
- アンチウイルスソフトによるブロック

#### 解決方法

1. 管理者権限で PowerShell を実行

# CURSOR を管理者権限で実行後この方法で解決

CURSOR を管理者権限で実行した影響のほうが大きそうに思える

```powershell
# 新しい管理者権限のPowerShellを開き、以下を実行
cd "パスをここに"
.\venv\Scripts\Activate.ps1
pip install --upgrade pyside6 --no-cache-dir
```

2. アンチウイルスソフトの一時的な無効化または除外設定
   - Windows Defender の場合:
     - 設定 → Windows セキュリティ → ウイルスと脅威の防止
     - 除外を追加 → フォルダ → venv フォルダを指定

### 3. 仮想環境の破損

#### 原因

- 仮想環境のファイルが部分的に破損
- パッケージの依存関係の競合

#### 解決方法

1. 仮想環境の再作成

   ```powershell
   # 現在の仮想環境を非アクティブ化
   deactivate

   # 古い仮想環境を削除
   Remove-Item -Recurse -Force venv

   # 新しい仮想環境を作成
   python -m venv venv

   # アクティブ化
   .\venv\Scripts\Activate.ps1

   # パッケージの再インストール
   pip install pyside6
   ```

2. キャッシュクリア付きインストール
   ```powershell
   pip cache purge
   pip install --upgrade pyside6 --no-cache-dir
   ```

### 4. パス名の問題

#### 原因

- パス名に日本語や空白が含まれている
- パスが長すぎる

#### 解決方法

1. 短いパスでの実行

   ```powershell
   # 一時的な作業ディレクトリを作成
   mkdir C:\temp_work
   cd C:\temp_work

   # ここで仮想環境を作成して試行
   python -m venv venv
   .\venv\Scripts\Activate.ps1
   pip install pyside6
   ```

2. 8.3 形式のショートパス名を使用

   ```powershell
   # 8.3形式のパスを取得
   Get-ItemProperty -Path "現在のパス" | Select-Object PSPath

   # 取得したショートパスを使用して実行
   cd "ショートパス"
   ```

## ベストプラクティス

1. インストール前の準備

   - すべての Python プロセスを終了
   - VSCode を完全に終了
   - アンチウイルスソフトの除外設定を確認

2. クリーンな環境での実行

   - 新しい管理者権限の PowerShell ウィンドウを使用
   - キャッシュを使用しないインストール
   - 必要に応じて仮想環境を再作成

3. トラブルシューティング
   - エラーメッセージを注意深く確認
   - プロセスの実行状態を確認
   - 権限とパスの問題を順次確認

## 注意事項

- バックアップ：仮想環境を再作成する前に、必要に応じて`requirements.txt`を作成
- ログ：トラブルシューティング時は`--verbose`オプションを使用して詳細情報を取得
- 回避策：問題が解決しない場合は、異なるバージョンの PySide6 を試すことも検討

## より安全なインストール方法

```powershell
# 段階的なアプローチ
# 1. 既存のパッケージをアンインストール
pip uninstall pyside6 PySide6-Essentials PySide6-Addons shiboken6 -y

# 2. pipをアップグレード
python -m pip install --upgrade pip

# 3. 依存関係を個別にインストール
pip install shiboken6==6.8.0.2
pip install PySide6-Essentials==6.8.0.2
pip install PySide6-Addons==6.8.0.2

# 4. 最後にPySide6本体をインストール
pip install pyside6==6.8.0.2
```
