# 创建新的Restangular方法

我们假设您的API需要一些自定义方法才能工作。
如果是这种情况，总是使用所有参数为该方法调用customGET或customPOST是一个痛苦的屁股。
这就是为什么每个元素都有一个`addRestangularMethod`方法。

这可以与钩子`addElementTransformer`一起使用来做一些整洁的东西。
让我们看一个例子来学习这个:

````javascript
// 用于设置默认的restangular配置的功能
export function RestangularConfigFactory (RestangularProvider) {
  // 它将转换所有建筑元素，而不是集合
  RestangularProvider.addElementTransformer('buildings', false, function(building) {
    // 这将添加一个名为evaluate的方法，它将使用NO缺省查询参数进行路径求值，并使用一些默认的头签名（name，operation，path，params，headers，elementToPost）

    building.addRestangularMethod('evaluate', 'get', 'evaluate', undefined, {'myHeader': 'value'});

    return building;
  });

  RestangularProvider.addElementTransformer('users', true, function(user) {
    // 这将添加一个名为login的方法，它将对路径登录签名进行POST（名称，操作，路径，参数，标题，elementToPost）

    user.addRestangularMethod('login', 'post', 'login');

    return user;
  });
}

// AppModule是进入Angular 2引导过程的主要入口点
@NgModule({
  bootstrap: [ AppComponent ],
  imports: [ // 导入Angular的模块
    RestangularModule.forRoot(RestangularConfigFactory),
  ],
})

// 然后，在您的代码中稍后您可以执行以下操作:

// 到达 /buildings/123/evaluate?myParam=param 用标题myHeader: value

// 如果这是一个安全的操作（GET，OPTIONS等），这个“自定义创建”方法的签名是（params，headers，elem）
// 如果这是一个不安全的操作（POST，PUT等），签名是（elem，params，headers）.

// 如果将某些内容设置为此变量，则将覆盖方法创建中的默认设置
// 如果未设置任何内容，则会发送默认值
Restangular.one('buildings', 123).evaluate({myParam: 'param'});

// 到达 /buildings/123/evaluate?myParam=param 用标题myHeader：: specialHeaderCase

Restangular.one('buildings', 123).evaluate({myParam: 'param'}, {'myHeader': 'specialHeaderCase'});

// 这里POST的主体将是{key：value}，因为POST是一个不安全的操作
Restangular.all('users').login({key: value});

````