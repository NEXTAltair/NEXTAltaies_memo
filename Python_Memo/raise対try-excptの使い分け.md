# エラー処理の考え方: raise vs try-except

## 基本的な判断基準

そのメソッドで「エラーから回復できるか？」で判断する。

## try-except を使うべきケース（回復できる）

### 1. ファイル処理

```python
def process_files(self, file_list):
    for file_path in file_list:
        try:
            self.process_single_file(file_path)
        except IOError:
            # 1つのファイルの失敗は許容して次へ進める
            self.logger.error(f"スキップ: {file_path}")
            continue
```

### 2. メモリ管理

```python
def resize_large_image(self, image, target_size):
    try:
        return image.resize(target_size)
    except MemoryError:
        # メモリ不足なら小さいサイズで再チャレンジ
        reduced_size = (target_size[0] // 2, target_size[1] // 2)
        return image.resize(reduced_size)
```

### 3. ネットワーク接続

```python
def fetch_data(self, url, max_retries=3):
    for attempt in range(max_retries):
        try:
            return requests.get(url).json()
        except requests.Timeout:
            # タイムアウトならリトライできる
            if attempt == max_retries - 1:
                raise  # 最後は諦めて上位に通知
            time.sleep(2 ** attempt)  # 指数バックオフ
```

## raise を使うべきケース（回復できない）

### 1. 設定エラー

```python
def initialize_processor(self, config):
    if 'api_key' not in config:
        raise ValueError("API keyが設定されていません")
    if not os.path.exists(config['work_dir']):
        raise FileNotFoundError("作業ディレクトリが見つかりません")
```

### 2. データ検証

```python
def validate_user_input(self, data):
    if not isinstance(data, dict):
        raise TypeError("入力は辞書形式である必要があります")
    if not data.get('user_id'):
        raise ValueError("user_idは必須です")
```

### 3. 重大なシステムエラー

```python
def connect_database(self):
    if not self.db_initialized:
        raise RuntimeError("DBが初期化されていません")
```

## 覚えておくポイント

### try-except を使う場合

- 代替手段がある
- 一部失敗しても全体には影響ない
- リトライで解決できる
- ユーザーに通知して続行できる

### raise を使う場合

- 前提条件が満たされていない
- これ以上続けても意味がない
- システムの整合性が保てない
- 上位レベルでの判断が必要

## 実装のコツ

- エラーメッセージは具体的に
- ログはちゃんと残す
- デバッグ情報も含める
- 複数のエラーを区別する
- ユーザー向けメッセージと技術的なログは分ける

## よくあるパターン

```python
class ImageProcessor:
    def process_image(self, image_path):
        # 回復できないチェック
        if not self.is_initialized:
            raise RuntimeError("初期化が必要です")

        try:
            # 回復できる処理
            with Image.open(image_path) as img:
                return self._process(img)
        except IOError as e:
            # エラーログは残すけど続行はできる
            self.logger.error(f"画像読み込みエラー: {e}")
            return None
        except Exception as e:
            # 想定外のエラーは上位に投げる
            self.logger.critical(f"重大なエラー: {e}")
            raise
```

## まとめ

- エラーの種類と影響範囲で判断
- 回復できるなら`try-except`
- 回復できないなら`raise`
- ログとエラーメッセージは丁寧に
- 上位での一括処理も考慮する
