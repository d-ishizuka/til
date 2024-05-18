# セットアップ時のメモ

## djangoプロジェクトを新規作成する
```
mkdir <app_dir>
cd app_dir
django-admin startproject config .  # 設定用のディレクトリを「config」として設定
```

## REST FRAMEWORKを使う前提の設定をする
```
# 設定ファイルの「INSTALLED_APPS」に「rest_framework」を追加する
config/settings.py
INSTALLED_APPS = [
  ...,
  'rest_framework',
]
```

### 参考
- 横瀬 明仁. 現場で使える Django REST Framework の教科書 （Django の教科書シリーズ）