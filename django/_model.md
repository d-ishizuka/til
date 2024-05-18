#　クラス定義の仕方

```
import uuid
from django import models

# modelで継承するものは、rest_frameworkでもMVCでも変わらない
class Book(models.Model):
    
    class Meta:
        db_table = 'book'
        ordering = ['created_at']
        verbose_name = verbose_name_plural = '本'
    
    # 基本的にuser_idを採番ではなくuuidにした方がURLが推測されずセキュリティ的により安全
    id = models.UUIDField(primary_key=True, default=uuid.uuid4, editable=False)
    title = models.CharField(verbose_name = 'タイトル', max_length=20, unique=True)

    price = models.IntegerField(verbose_name = '価格', null=True, blank=True)
    created_at = models.DateTimeFiled(verbose_name='登録日時', auto_now_add=True)

    def __str__(self):
        return self.title
```

## Fieldクラスに指定できるフィールドオプション(一部)
|フィールド|説明|
|-|-|
|verbose_name|フィールドの名前|
|null|null データベースの NOT NULL制約 （デフォルト値は False： NULL  は許可しない）|
|unique|データベースのユニーク制約|
|db_index|データベースのインデックスを設定するかどうか|
|primary_key|primary_key=True を指定したフィールドが主キーになる|
|default|レコード登録時に値が指定されなかったときのデフォルト値|
|max_length|文字列の最大文字数。 CharField では指定必須|
|choices|登録・更新時の入力値の取り得る値を制限する|
|blank|バリデーションに関するオプション。 CharField や TextField の場合、後述する ModelSerializer でのバリデーション時に空白文字列を許可するかどうかを規定 （デフォルト値は False： 許可しない）|
|validators|文字種チェックなどのバリデーションを指定|
|on_delete|OneToOneField および ForeignKey で指定必須。関連先のモデルオブジェクトが削除された際の挙動を規定|

## リレーションの定義
### 「一対一」のリレーション
```
import uuid
from django.db import models

class Book(models.Model):
    
    class Meta:
        db_table = 'book'
        ordering = ['created_at']
        verbose_name = verbose_name_plural = '本'
    
    # 基本的にuser_idを採番ではなくuuidにした方がURLが推測されずセキュリティ的により安全
    id = models.UUIDField(primary_key=True, default=uuid.uuid4, editable=False)
    title = models.CharField(verbose_name = 'タイトル', max_length=20, unique=True)

    price = models.IntegerField(verbose_name = '価格', null=True, blank=True)
    created_at = models.DateTimeFiled(verbose_name='登録日時', auto_now_add=True)

# FKとして、book_idを持つ
class BookStock(models.Model):
    
    class Meta:
        db_table = 'book_stock'
    
    # book_idというカラムができる
    book = model.OneToOneField(Book, verbose_name='本', on_delete=models.CASCADE)

    quantity = models.IntegerField(verbose_name='在庫数', default=0)

※ BookStockオブジェクトからは「book」でBookオブジェクトを参照
※ Bookオブジェクトからは「bookstock」でBookStockオブジェクトを参照

```

### 「多対一」のリレーション
```
import uuid
from django.db import models

class Publisher(models.Model):
    
    class Meta:
        db_table = 'publisher'
    
    id = models.UUIDField(primary_key=True, default=uuid.uuid4, editable=False)
    name = models.CharField(verbose_name = '出版社名', max_length=20)
    created_at = models.DateTimeFiled(verbose_name='登録日時', auto_now_add=True)

class Book(models.Model):
    
    class Meta:
        db_table = 'book'

    id = models.UUIDField(primary_key=True, default=uuid.uuid4, editable=False)
    title = models.CharField(verbose_name = 'タイトル', max_length=20, unique=True)

    price = models.IntegerField(verbose_name = '価格', null=True, blank=True)
    publisher = models.ForeignKey(Publisher, verbose_name='出版社', on_delete=models.PROTECT, null=True, blank=True)

    created_at = models.DateTimeFiled(verbose_name='登録日時', auto_now_add=True)

※ Bookオブジェクトからは「publisher_id」でPublisherの主キーを参照
※ Bookオブジェクトからは「publisher」でPublisherオブジェクトを参照
※ Publisherオブジェクトからは「book_set」でBookのモデルマネージャを参照
```

### 「多対多」のリレーション
```
import uuid
from django.db import model

class Author(models.Model):
    
    class Meta:
        db_table = 'author'
    
    id = models.UUIDField(primary_key=True, default=uuid.uuid4, editable=False)
    name = models.CharField(verbose_name = '著者名', max_length=20)

    created_at = models.DateTimeFiled(verbose_name='登録日時', auto_now_add=True)

class Book(models.Model):

    class Meta:
        db_table = 'book'

    id = models.UUIDField(primary_key=True, default=uuid.uuid4, editable=False)
    title = models.CharField(verbose_name = 'タイトル', max_length=20, unique=True)

    price = models.IntegerField(verbose_name = '価格', null=True, blank=True)
    authors = models.ManyToManyField(Author, verbose_name='著者', blank=True)

    created_at = models.DateTimeFiled(verbose_name='登録日時', auto_now_add=True)

※ Bookオブジェクトから「authors」でAuthorのモデルマネージャーを参照
※ Authorオブジェクトからは「book_set」でBookのモデルマネージャーを参照
```

### モデルオブジェクトの取得_Basic
- モデルは「モデルマネージャ」というデータベースのテーブルレベルのクエリ操作をするためのクエリセットAPIのインターフェイスを持つ
- モデルマネージャは、通常モデルクラスに「objects」という属性で用意されている

```
# all
Book.objects.all()

list(Book.objects.all())

# filter
Book.objects.filter(price=2000)

# OR条件
from django.db import Q
Book.objects.filter(Q(title='DRFの本' | Q(price=2000)))

# 比較
・フィールド名に「__」をつけて、gt, lt, gte, lteを使う
Book.objects.filter(price__gt=2000)
Book.objects.filter(price__lt=2000)
Book.objects.filter(price__gte=2000)
Book.objects.filter(price__lte=2000)

# IN
Book.objects.filter(price__in = [1500, 2000])
```

### モデルオブジェクトの取得_JOINが必要なもの
- リレーションが貼ってあるデータに関して本来ならばJOINを使う必要があるが、Djangoモデルの場合、「OneToOneField, ForeignKey, ManyToManyField」のリレーションで定義されている場合には簡単にできる。
```
Book.objects.filter(publisher__name='自費出版社')

以下のようなクエリと同じこと
SELECT * FROM book AS b INNER JOIN publisher AS p ON b.publisher_id = p.id WHERE p.name = '自費出版社'
```

### 1件だけ取得する場合
```
Book.objects.get(title='DRFの本')
※ 0件の場合や、複数件の場合はエラーが起こる
※ filterでもできる
```

### トランザクション・ロールバック
- django.db.transaction.atomic()を使う
```
from django.db import transaction

with transaction.atomic():
    Book(xxx).save()
    Book(yyy).save()
```

## マイグレーション
- マイグレーションファイルの作成
- 引数なしで実行すると、「INSTALLED_APPS」に登録されたすべてのアプリケーションに対して処理が行われる
- 個別のアプリケーションに限定するには、アプリケーション名を引数に指定する
```
# 全てのアプリケーション
python manage.py makemigration

# アプリケーションを指定して実行
python manage.py <アプリケーション1> <アプリケーション2>
```

- マイグレーションの実行
- DBに反映する
```
# 全てのアプリケーション
python manage.py migrate

# アプリケーションを指定して実行
python manage.py <アプリケーション1> <アプリケーション2>
```

### 参考
- 現場で使える Django REST Framework の教科書