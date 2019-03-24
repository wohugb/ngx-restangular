# 将自定义方法添加到集合

使用`Restangular.extendCollection()`为集合创建自定义方法。
这是别名:

```js
  RestangularProvider.addElementTransformer(route, true, fn);
```

## 例

```js
  // 为您的集合创建方法
  Restangular.extendCollection('accounts', function(collection) {
    collection.totalAmount = function() {
      // 这里实施
    };

    return collection;
  });

  var accounts$ = Restangular.all('accounts').getList();

  accounts$.subscribe( accounts => {
    accounts.totalAmount(); // 调用自定义集合方法
  });
```