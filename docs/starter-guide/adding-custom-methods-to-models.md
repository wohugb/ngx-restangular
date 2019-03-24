# 向模型添加自定义方法

使用Restangular.extendModel()为模型创建自定义方法。 这是别名:

```js
  RestangularProvider.addElementTransformer(route, false, fn);
```

## 例

```js
  Restangular.extendModel('accounts', function(model) {
    model.prettifyAmount = function() {};
    return model;
  });

  var account$ = Restangular.one('accounts', 1).get();

  account$.subscribe(function(account) {
    account.prettifyAmount(); // invoke your custom model method
  });
```