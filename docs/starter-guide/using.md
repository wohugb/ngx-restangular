# 使用Restangular

## 创建主Restangular对象

有三种方法可以创建一个主Restangular对象。
第一个也是最常见的一个是陈述所有请求的主要路径。
第二个是陈述所有请求的主要路径和对象。

````javascript
// Only stating main route
Restangular.all('accounts')

// Stating main object
Restangular.one('accounts', 1234)

// Gets a list of all of those accounts
Restangular.several('accounts', 1234, 123, 12345);
````

## 让代码与Observables

现在我们有了主要的对象让我们开始玩它。

````javascript
// AppModule is the main entry point into Angular2 bootstraping process
@NgModule({
  bootstrap: [ AppComponent ],
  declarations: [
    AppComponent,
  ],
  imports: [
    // Importing RestangularModule
    RestangularModule,
  ]
})
export class AppModule {
}

@Component({
  ...
})
export class OtherComponent {
  allAccounts;
  accounts;
  account;

  constructor(private restangular: Restangular) {
  }

  ngOnInit() {
    // First way of creating a this.restangular object. Just saying the base URL
    let baseAccounts = this.restangular.all('accounts');

    // This will query /accounts and return a observable.
    baseAccounts.getList().subscribe(accounts => {
      this.allAccounts = accounts;
    });


    let newAccount = {name: "Gonto's account"};

    // POST /accounts
    baseAccounts.post(newAccount);

    // GET to http://www.google.com/ You set the URL in this case
    this.restangular.allUrl('googlers', 'http://www.google.com/').getList();

    // GET to http://www.google.com/1 You set the URL in this case
    this.restangular.oneUrl('googlers', 'http://www.google.com/1').get();

    // You can do RequestLess "connections" if you need as well

    // Just ONE GET to /accounts/123/buildings/456
    this.restangular.one('accounts', 123).one('buildings', 456).get();

    // Just ONE GET to /accounts/123/buildings
    this.restangular.one('accounts', 123).getList('buildings');

    // Here we use Observables
    // GET /accounts
    let baseAccounts$ = baseAccounts.getList().subscribe(accounts => {
      // Here we can continue fetching the tree :).

      let firstAccount = accounts[0];
      // This will query /accounts/123/buildings considering 123 is the id of the firstAccount
      let buildings = firstAccount.getList("buildings");

      // GET /accounts/123/places?query=param with request header: x-user:mgonto
      let loggedInPlaces = firstAccount.getList("places", {query: 'param'}, {'x-user': 'mgonto'});

      // This is a regular JS object, we can change anything we want :)
      firstAccount.name = "Gonto";

      // If we wanted to keep the original as it is, we can copy it to a new element
      let editFirstAccount = this.restangular.copy(firstAccount);
      editFirstAccount.name = "New Name";


      // PUT /accounts/123. The name of this account will be changed from now on
      firstAccount.put();
      editFirstAccount.put();

      // PUT /accounts/123. Save will do POST or PUT accordingly
      firstAccount.save();

      // DELETE /accounts/123 We don't have first account anymore :(
      firstAccount.remove();

    }, () => {
      alert("Oops error from server :(");
    });


    // Get first account
    let firstAccount$ = baseAccounts$.map(accounts => accounts[0]);


    // POST /accounts/123/buildings with MyBuilding information
    firstAccount$.switchMap(firstAccount => {
      var myBuilding = {
        name: "Gonto's Building",
        place: "Argentina"
      };

      return firstAccount.post("Buildings", myBuilding)
    })
    .subscribe(() => {
      console.log("Object saved OK");
    }, () => {
      console.log("There was an error saving");
    });


    // GET /accounts/123/users?query=params
    firstAccount$.switchMap(firstAccount => {
      var myBuilding = {
        name: "Gonto's Building",
        place: "Argentina"
      };

      return firstAccount.getList("users", {query: 'params'});
    })
    .subscribe((users) => {
      // Instead of posting nested element, a collection can post to itself
      // POST /accounts/123/users
      users.post({userName: 'unknown'});

      // Custom methods are available now :).
      // GET /accounts/123/users/messages?param=myParam
      users.customGET("messages", {param: "myParam"});

      var firstUser = users[0];

      // GET /accounts/123/users/456. Just in case we want to update one user :)
      let userFromServer = firstUser.get();

      // ALL http methods are available :)
      // HEAD /accounts/123/users/456
      firstUser.head()
    }, () => {
      console.log("There was an error saving");
    });


    // Second way of creating this.restangular object. URL and ID :)
    var account = this.restangular.one("accounts", 123);

    // GET /accounts/123?single=true
    this.account = account.get({single: true});

    // POST /accounts/123/messages?param=myParam with the body of name: "My Message"
    account.customPOST({name: "My Message"}, "messages", {param: "myParam"}, {})
  }
}
````

## 这是使用promises的代码示例

````javascript
@Component({
  ...
})
export class OtherComponent {
  allAccounts;
  accounts;
  account;

  constructor(private restangular: Restangular) {
  }

  ngOnInit() {

    // First way of creating a this.restangular object. Just saying the base URL
    let baseAccounts = this.restangular.all('accounts');

    // This will query /accounts and return a promise.
    baseAccounts.getList().toPromise().then(function(accounts) {
      this.allAccounts = accounts;
    });

    var newAccount = {name: "Gonto's account"};

    // POST /accounts
    baseAccounts.post(newAccount);

    // GET to http://www.google.com/ You set the URL in this case
    this.restangular.allUrl('googlers', 'http://www.google.com/').getList();

    // GET to http://www.google.com/1 You set the URL in this case
    this.restangular.oneUrl('googlers', 'http://www.google.com/1').get();

    // You can do RequestLess "connections" if you need as well

    // Just ONE GET to /accounts/123/buildings/456
    this.restangular.one('accounts', 123).one('buildings', 456).get();

    // Just ONE GET to /accounts/123/buildings
    this.restangular.one('accounts', 123).getList('buildings');

    // Here we use Promises then
    // GET /accounts
    baseAccounts.getList().toPromise().then(function (accounts) {
      // Here we can continue fetching the tree :).

      var firstAccount = accounts[0];
      // This will query /accounts/123/buildings considering 123 is the id of the firstAccount
      this.buildings = firstAccount.getList("buildings");

      // GET /accounts/123/places?query=param with request header: x-user:mgonto
      this.loggedInPlaces = firstAccount.getList("places", {query: 'param'}, {'x-user': 'mgonto'});

      // This is a regular JS object, we can change anything we want :)
      firstAccount.name = "Gonto";

      // If we wanted to keep the original as it is, we can copy it to a new element
      var editFirstAccount = this.restangular.copy(firstAccount);
      editFirstAccount.name = "New Name";


      // PUT /accounts/123. The name of this account will be changed from now on
      firstAccount.put();
      editFirstAccount.put();

      // PUT /accounts/123. Save will do POST or PUT accordingly
      firstAccount.save();

      // DELETE /accounts/123 We don't have first account anymore :(
      firstAccount.remove();

      var myBuilding = {
        name: "Gonto's Building",
        place: "Argentina"
      };

      // POST /accounts/123/buildings with MyBuilding information
      firstAccount.post("Buildings", myBuilding).toPromise().then(function() {
        console.log("Object saved OK");
      }, function() {
        console.log("There was an error saving");
      });

      // GET /accounts/123/users?query=params
      firstAccount.getList("users", {query: 'params'}).toPromise().then(function(users) {
        // Instead of posting nested element, a collection can post to itself
        // POST /accounts/123/users
        users.post({userName: 'unknown'});

        // Custom methods are available now :).
        // GET /accounts/123/users/messages?param=myParam
        users.customGET("messages", {param: "myParam"});

        var firstUser = users[0];

        // GET /accounts/123/users/456. Just in case we want to update one user :)
        this.userFromServer = firstUser.get();

        // ALL http methods are available :)
        // HEAD /accounts/123/users/456
        firstUser.head()

      });

    }, function errorCallback() {
      alert("Oops error from server :(");
    });

    // Second way of creating this.restangular object. URL and ID :)
    var account = this.restangular.one("accounts", 123);

    // GET /accounts/123?single=true
    this.account = account.get({single: true});

    // POST /accounts/123/messages?param=myParam with the body of name: "My Message"
    account.customPOST({name: "My Message"}, "messages", {param: "myParam"}, {})
  }
}
````