# VSCode Project Manager の projects.json

## 概要

projects.json は、VSCode の Project Manager 拡張機能で使用するプロジェクト設定ファイルだ。複数のプロジェクトを簡単に切り替えられるようにするための設定が含まれている。

## ファイルの場所

Windows の場合、以下のパスに保存される：

```
%APPDATA%\Code\User\globalStorage\alefragnani.project-manager\projects.json
```

## 基本的な構造

```json
{
  "projects": [
    {
      "name": "自分のプロジェクト",
      "rootPath": "C:\\Users\\username\\projects\\my-project",
      "enabled": true,
      "tags": []
    },
    {
      "name": "ライブラリ開発",
      "rootPath": "D:\\workspace\\library",
      "enabled": true,
      "tags": ["ライブラリ"]
    }
  ]
}
```

## 設定項目の説明

### 必須項目

- `name`: プロジェクトの表示名
- `rootPath`: プロジェクトのフォルダパス
  - 絶対パスで指定する
  - バックスラッシュはエスケープが必要（`\\`）

### オプション項目

- `tags`: プロジェクトの分類タグ（配列）
  - 1 つのプロジェクトに複数のタグを設定可能
  - 例：`["frontend", "react", "production"]`
  - タグによるフィルタリングや検索が可能
- `enabled`: プロジェクトの有効/無効（true/false）
- `paths`: サブフォルダのパス（配列）
  - rootPath からの相対パスで指定
  - プロジェクト内の特定のフォルダを個別に管理できる
  - VSCode のマルチルートワークスペース機能と組み合わせて使用可能
  - 例：
    ```json
    "paths": [
        "frontend/src",
        "backend/api",
        "docs"
    ]
    ```
  - 使用ケース：
    - マイクロサービスの各サービスディレクトリ
    - モノレポ内の複数のパッケージ
    - フロントエンド/バックエンドの分離開発
    - ライブラリのサブモジュール

## 設定例

### 単純なプロジェクト

```json
{
  "projects": [
    {
      "name": "フロントエンド開発",
      "rootPath": "C:\\Projects\\frontend",
      "enabled": true
    }
  ]
}
```

### タグ付けしたプロジェクト

```json
{
  "projects": [
    {
      "name": "ウェブアプリ",
      "rootPath": "C:\\Projects\\webapp",
      "tags": ["production", "web", "react"],
      "enabled": true
    },
    {
      "name": "テスト環境",
      "rootPath": "C:\\Projects\\webapp-test",
      "tags": ["staging", "web", "react"],
      "enabled": true
    },
    {
      "name": "バックエンドAPI",
      "rootPath": "C:\\Projects\\api",
      "tags": ["backend", "nodejs", "production"],
      "enabled": true
    }
  ]
}
```

### サブフォルダを含むプロジェクト

```json
{
  "projects": [
    {
      "name": "マイクロサービス",
      "rootPath": "C:\\Projects\\microservices",
      "paths": ["service-a", "service-b", "common-lib"],
      "enabled": true,
      "tags": ["microservices", "development", "nodejs"]
    },
    {
      "name": "モノレポ開発",
      "rootPath": "C:\\Projects\\monorepo",
      "paths": [
        "packages/ui-components",
        "packages/core",
        "apps/web",
        "apps/admin"
      ],
      "enabled": true,
      "tags": ["フロントエンド"]
    },
    {
      "name": "ドキュメントプロジェクト",
      "rootPath": "C:\\Projects\\documentation",
      "paths": ["api-docs/v1", "api-docs/v2", "user-guide", "developer-guide"],
      "enabled": true,
      "tags": ["ドキュメント"]
    }
  ]
}
```

## 注意点

1. パスの指定

   - Windows のパスは必ずバックスラッシュ（`\\`）を使う
   - フォワードスラッシュ（`/`）は使えない

2. 文字コード

   - UTF-8 で保存する
   - 日本語のプロジェクト名やパスも使用可能

3. 編集方法

   - 直接編集よりも VSCode UI からの編集を推奨
   - 手動編集する場合は JSON の文法に注意

4. バックアップ
   - 重要なプロジェクト設定は別途バックアップを推奨
   - 設定を共有する場合は機密情報に注意

## よくある使い方

1. プロジェクトの切り替え

   - `Ctrl + Alt + P`でプロジェクト一覧を表示
   - プロジェクト名を選択して切り替え

2. タグでの管理

   - 開発ステージ別（development, staging, production）
   - 技術スタック別（react, nodejs, python）
   - プロジェクトタイプ別（frontend, backend, fullstack）
   - クライアント別（client-a, client-b）
   - 状態別（active, archived, maintenance）

3. 一時的な無効化
   - `enabled: false`で一時的にリストから除外
   - 完全に削除せずに管理できる
