# Django REST Framework(DEF)と通常のDjangoでMVCを書くときの違い

## シリアライザ
```
from rest_framework import serializer
from .model import Book

# rest_frameworkに含まれるModelSerializerを継承させる
class BookSerializer(serializers.ModelSerializer):
    
    class Meta:
        model = Book
        # 利用するモデルのフィールドを指定
        fields = ['id', 'title', 'price']
```

## ビュー
```
from rest_framework import status, views
from rest_framework.generics import get_object_or_404
from rest_framework.response import Response

from .models import Book
from .serializers import BookSerializer

# rest_frameworkからimportしたviews.APIViewをviewクラスに継承させる
class BookListCreateAPIView(views.APIView):
    """本モデルの取得（一覧） ・登録APIクラス"""
    
    def get(self, request, *args, **kwargs):
        """本モデルの取得（一覧） APIに対応するハンドラメソッド"""
        
        #モデルオブジェクトの一覧を取得
        book_list = Book.objects.all()
        
        #シリアライザオブジェクトを作成(rest_frameworkのModelSerializerを継承しているモデル)
        serializer = BookSerializer(instance=book_list, many=True)
        
        #レスポンスオブジェクトを返す
        return Response(serializer.data)
        
    def post(self, request, *args, **kwargs):
        """本モデルの登録APIに対応するハンドラメソッド"""
        
        #シリアライザオブジェクトを作成
        serializer = BookSerializer(data=request.data)
        
        #バリデーションを実行
        serializer.is_valid(raise_exception=True)
        
        #モデルオブジェクトを登録
        serializer.save()
        
        #レスポンスオブジェクトを返す(ResponseはRestFrameworkからimportしている)
        return Response(serializer.data, status.HTTP_201_CREATED) 
    
class BookRetrieveUpdateDestroyAPIView(views.APIView):
    """本モデルの取得（詳細） ・更新・一部更新・削除APIクラス"""
    
    def get(self, request, pk, *args, **kwargs):
        """本モデルの取得（詳細） APIに対応するハンドラメソッド"""
        
        #モデルオブジェクトを取得(rest_frameworkからimportしたメソッド)
        book = get_object_or_404(Book, pk=pk)
        
        #シリアライザオブジェクトを作成
        serializer = BookSerializer(instance=book)
        
        #レスポンスオブジェクトを返す 
        return Response(serializer.data)

    def put(self, request, pk, *args, **kwargs):
        """本モデルの更新APIに対応するハンドラメソッド"""
        
        #モデルオブジェクトを取得
        book = get_object_or_404(Book, pk=pk)
        
        #シリアライザオブジェクトを作成
        serializer = BookSerializer(instance=book, data=request.data)
        
        #バリデーションを実行
        serializer.is_valid(raise_exception=True)
        
        #モデルオブジェクトを更新
        serializer.save()
        
        #レスポンスオブジェクトを返す
        return Response(serializer.data)
    
    def patch(self, request, pk, *args, **kwargs):
        """本モデルの一部更新APIに対応するハンドラメソッド"""
        
        #モデルオブジェクトを取得
        book = get_object_or_404(Book, pk=pk)
        
        #シリアライザオブジェクトを作成
        serializer = BookSerializer(instance=book, data=request.data, partial=True)
        
        #バリデーションを実行
        serializer.is_valid(raise_exception=True)
        
        #モデルオブジェクトを一部更新
        serializer.save()
        
        #レスポンスオブジェクトを返す
        return Response(serializer.data)
        
    def delete(self, request, pk, *args, **kwargs):
        """本モデルの削除APIに対応するハンドラメソッド"""
        
        #モデルオブジェクトを取得
        book = get_object_or_404(Book, pk=pk)
        
        #モデルオブジェクトを削除
        book.delete()
        
        #レスポンスオブジェクトを返す(rest_frameworkで定義されたstatus)
        return Response(status=status.HTTP_204_NO_CONTENT) 
```

## URL
```
from django.urls import path
from api import views

# viewsはrest_frameworkからimportしたviews.APIViewを継承している
urlpatterns = [
  path('api/books/', views.BookListCreateAPIView.as_view()),
  path('api/books/<pk>', views.BookRetrieveUpdateDestroyAPIView.as_view()),
]
```

### 参考
- 横瀬 明仁. 現場で使える Django REST Framework の教科書