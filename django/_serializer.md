# シリアライザ
## 役割
- リクエストのdataを引数として、シリアライザをインスタンス化
  - RailsだとRequestをJsonParseしている&返す時だけシリアライザを使っているが、DjangoではRequestのデータを受け取るときにも使う
- インスタンス化してバリデーション(dict型のオブジェクト)
- 永続化
- dataとして返す

## クラス継承
- REST APIでシリアライザを利用するときは以下のどちらかを継承して作成する
  - rest_framework.serializers.Serializer
    - 単独のリソース
  - ListSerializer
    - 複数のリソース
```
from rest_framework import serializer
from .model import Book

# rest_frameworkに含まれるModelSerializerを継承させる
class BookSerializer(serializers.ModelSerializer):
    
    class Meta:
        model = Book
        # 利用するモデルのフィールドを指定
        fields = ['id', 'title', 'price'] # 全て選ぶなら'__all__', 除くなら'exclude' 
```

## シリアライザのFieldオプション

|フィールドオプション|説明|
|-|-|
|write_only|登録・更新・一部更新時の入力用フィールドには含めるが、出力用フィールドには含めたくない場合にTrueを指定（デフォルト値はFalse）|
|read_only|出力用フィールドには含めるが、 登録・更新・一部更新時の入力用のフィールドには含めたくない場合にTrueを指定。 read_only=Trueを指定したフィールドは登録・更新・一部更新の対象外となる（デフォルト値はFalse）|
|required|入力データにフィールドが指定されなかった場合にバリデーションNGにするかどうか。 なお、defaultオプションが設定されている場合やread_only=Trueの場合はrequired=Falseとなる（デフォルト値はTrue：フィールド必須）|
|default|登録・更新時にフィールドが指定されなかったときに使われるデフォルト値（ただし一部更新のときは使われない）|
|allow_null|入力値にnullを許可するかどうか。Falseの場合、登録・更新・ 一部更新時に入力値がnullになっているとバリデーションNGとなる（デフォルト値は False： 許可しない）|
|allow_blank|入力値に空文字を許可するかどうか。CharField などで利用可|
|source|値を出力する際の参照先をデフォルトから変更したい場合に使う。関連モデルの属性をドット区切りで指定したりすることも可能。参照先が見つからない場合は defaultの値を使う|
|max_length|CharFieldなどのバリデーションで利用される最大桁数|
|min_length|CharFieldなどのバリデーションで利用される最小桁数|
|max_value|IntegerFieldやFloatField のバリデーションで利用される最大値|
|min_value|IntegerFieldやFloatField のバリデーションで利用される最小値|
|validators|文字種チェックなどのバリデーション。listやtuple で指定|
|error_messages|バリデーション NG の場合のエラーメッセージ。 dictで指定|

## Field定義とシリアライザの関係性
- xxすると、モデルのFieldクラスが、シリアライザのFieldクラスへと変換される
- その際に、例えばモデルのFieldクラスで定義されていたフィールドオプションnull=Trueが、シリアライザのallow_null=Trueに引き継がれる

- モデルで定義したFieldオプションをシリアライザ処理では変更したいとき
  - 方法１：クラス変数に同名のフィールドを定義する
  - 方法２：Metaクラスの「extra_kwargs」を使う
```
from rest_framework import serializers
from .models import Book
class BookSerializer(serializers.ModelSerializer):
    price = serializers.CharField(read_only=True) #CharField に変更(方法1)

    class Meta:
        model = Book
        exclude = ['created_at']
        extra_kwargs = {
          'title': {  
            'write_only': True, #入力のみで利用
            'max_length': 10, #バリデーションの最大桁数を10に変更  },
        } 
```

## ListSerializerを使用する
- 複数のBookモデルを扱うためのBookListSerializer
- childというクラス変数に１つのモデルを返すシリアライザーを設定する
```
from rest_framework import serializers
from .models import Book
class BookSerializer(serializers.ModelSerializer):
    class Meta:
        model = Book
        exclude = ['created_at']

class BookListSerializer(serializers.ListSerializer):
    """複数の本モデルを扱うためのシリアライザ"""
    
    #対象のシリアライザを指定
    child = BookSerializer()
```

## モデルに依存しない自由な形式でJSONを返したいとき(重要!!)
- serializerで継承するクラスを、「serializers.ModelSerializer」から「serializers.Serializer」に変更する
- Serializer内で型定義したクラス変数がObject形式で返される
```
import random
from django.utils import timezone
from rest_framework import serializers

# serializers.Serializerを継承
class FortuneSerializer(serializers.Serializer):
    
    birth_date = serializers.DateField()
    blood_type = serializers.ChoiseField(choices=['A', 'B', 'C', "O"])
    #出力時に get_current_date() が呼ばれる
    current_date = serializers.SerializerMethodField()
    #出力時に get_fortune() が呼ばれる
    fortune = serializers.SerializerMethodField()
    
    def get_current_date(self, obj):
        return timezone.localdate()
        
    def get_fortune(self, obj):
        seed = f"{timezone.localdate()}{obj['birth_date']}{obj['blood_type']}"
        random.seed(seed)
        return random.choice(  ['★☆☆☆☆', '★★☆☆☆', '★★★☆☆', '★★★★☆', '★★★★★']  ) 

# JSONレスポンス
{
  "birth_date": "1990-01-01",
  "blood_type": "A",
  "current_date": "2022-11-12",
  "fortune": "★★☆☆☆"
} 
```

### 参考
- 現場で使える Django REST Framework の教科書 （Django の教科書シリーズ）