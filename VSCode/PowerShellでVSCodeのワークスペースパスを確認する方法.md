# PowerShell で VSCode のワークスペースを正しく開く方法

```powershell
# 正しいワークスペースを開く（絶対パスで指定）
code "パス\to\your-workspace.code-workspace"

# または、まずディレクトリに移動してから開く
cd "パス\to\workspace-directory"
code your-workspace.code-workspace
```

これで ${workspaceFolder} が正しいパスを参照する
