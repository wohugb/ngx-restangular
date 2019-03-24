# URL构建

有时，我们有很多嵌套实体（及其ID），但我们只想要最后一个孩子。
在这些情况下，为最后一个孩子做一切请求是有点过分的。
对于这些情况，我添加了使用与创建新的Restangular对象相同的API创建URL的可能性。
创建此连接时不会发出任何请求。
让我们看看如何做到这一点:

````javascript

var restangularSpaces = Restangular.one("accounts",123).one("buildings", 456).all("spaces");

// 这将做一个到达 /accounts/123/buildings/456/spaces
restangularSpaces.getList()

// 这将做一个到达 /accounts/123/buildings/456/spaces/789
Restangular.one("accounts", 123).one("buildings", 456).one("spaces", 789).get()

// POST /accounts/123/buildings/456/spaces
Restangular.one("accounts", 123).one("buildings", 456).all("spaces").post({name: "New Space"});

// DELETE /accounts/123/buildings/456
Restangular.one("accounts", 123).one("buildings", 456).remove();
````
