# Git で追跡可能なファイル移動手順

## 1. PowerShell の Move-Item と Git の違い

### Move-Item を使用した場合

```powershell
Move-Item -Path "CSVToDatabaseProcessor.py" -Destination "src\genai_tag_db_tools\core\processor.py"
```

- Git から見ると「元のファイルの削除」と「新しいファイルの作成」として認識される
- ファイルの履歴が途切れる
- 変更履歴が失われる可能性がある

### git mv を使用した場合

```powershell
git mv "CSVToDatabaseProcessor.py" "src\genai_tag_db_tools\core\processor.py"
```

- Git はファイルの移動として認識
- ファイルの履歴が保持される
- 変更履歴が維持される

## 2. 正しい移行手順

### コアモジュールの移動

```powershell
# ディレクトリの作成（必要な場合）
New-Item -Path "src\genai_tag_db_tools\core" -ItemType Directory -Force

# CSVToDatabaseProcessor.pyの移動
git mv "CSVToDatabaseProcessor.py" "src\genai_tag_db_tools\core\processor.py"

# tag_search.pyの移動
git mv "tag_search.py" "src\genai_tag_db_tools\core\searcher.py"
```

### GUI モジュールの移動

```powershell
# ディレクトリの作成
$gui_dirs = @(
    "src\genai_tag_db_tools\gui\windows",
    "src\genai_tag_db_tools\gui\widgets"
)
foreach ($dir in $gui_dirs) {
    New-Item -Path $dir -ItemType Directory -Force
}

# MainWindow関連ファイルの移動
git mv "gui\MainWindow.py" "src\genai_tag_db_tools\gui\windows\main_window.py"
git mv "gui\MainWindow_ui.py" "src\genai_tag_db_tools\gui\windows\main_window_ui.py"

# Widgetファイルの移動
$widgets = @{
    "TagCleanerWidget.py" = "tag_cleaner.py"
    "TagRegisterWidget.py" = "tag_register.py"
    "TagSearchWidget.py" = "tag_search.py"
    "TagStatisticsWidget.py" = "tag_statistics.py"
}

foreach ($widget in $widgets.GetEnumerator()) {
    git mv "gui\$($widget.Key)" "src\genai_tag_db_tools\gui\widgets\$($widget.Value)"
}

# UIファイルの移動
$ui_files = @{
    "TagCleanerWidget_ui.py" = "tag_cleaner_ui.py"
    "TagRegisterWidget_ui.py" = "tag_register_ui.py"
    "TagSearchWidget_ui.py" = "tag_search_ui.py"
    "TagStatisticsWidget_ui.py" = "tag_statistics_ui.py"
}

foreach ($ui in $ui_files.GetEnumerator()) {
    git mv "gui\$($ui.Key)" "src\genai_tag_db_tools\gui\widgets\$($ui.Value)"
}
```

## 3. 移動の
