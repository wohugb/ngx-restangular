
# FAQ

??? faq "我该如何处理错误?"

    可以在订阅的第二个参数上检查错误。

    ````javascript
    Restangular.all("accounts").getList().subscribe( response => {
      console.log("All ok");
    }, errorResponse => {
      console.log("Error with status code", errorResponse.status);
    });
    ````

??? faq "我需要在每个Restangular请求中发送授权令牌，我该怎么做？"

    你可以使用`setDefaultHeaders`或`addFullRequestInterceptor`

    ````javascript
    import { NgModule } from '@angular/core';
    import { AppComponent } from './app.component';
    import { RestangularModule } from 'ngx-restangular';
    import { authService } from '../your-services';

    // Function for settting the default restangular configuration
    export function RestangularConfigFactory (RestangularProvider, authService) {

      // set static header
      RestangularProvider.setDefaultHeaders({'Authorization': 'Bearer UDXPx-Xko0w4BRKajozCVy20X11MRZs1'});

      // by each request to the server receive a token and update headers with it
      RestangularProvider.addFullRequestInterceptor((element, operation, path, url, headers, params) => {
        let bearerToken = authService.getBearerToken();

        return {
          headers: Object.assign({}, headers, {Authorization: `Bearer ${bearerToken}`})
        };
      });
    }

    // AppModule is the main entry point into Angular2 bootstraping process
    @NgModule({
      bootstrap: [ AppComponent ],
      declarations: [
        AppComponent,
      ],
      imports: [
        // Importing RestangularModule and making default configs for restanglar
        RestangularModule.forRoot([authService], RestangularConfigFactory),
      ]
    })
    export class AppModule {
    }
    ````

??? faq "我需要在每个Restangular请求中发送一个标头，我该怎么做？"

    您可以使用`defaultHeaders`属性。 `defaultsHeaders`可以用`withConfig`作为范围，所以它非常酷。

    ??? faq "How can I send a delete WITHOUT a body?"

    You must add a requestInterceptor for this.

    ````js
    RestangularProvider.setRequestInterceptor(function(elem, operation) {
      if (operation === "remove") {
        return null;
      }
      return elem;
    })
    ````

??? faq "我使用Mongo，元素的ID是`_id`而不是`id`作为默认值。 因此，请求被发送到未定义的路由"

    What you need to do is to configure the `RestangularFields` and set the `id` field to `_id`. Let's see how:

    ````javascript
    RestangularProvider.setRestangularFields({
      id: "_id"
    });
    ````

??? faq "如果我的每个模型都有不同的ID名称，例如CustomerID for Customer，该怎么办？"

    In some cases, people have different ID name for each entity. For example, they have CustomerID for customer and EquipmentID for Equipment. If that's the case, you can override Restangular's getIdFromElem. For that, you need to do:

    ````js
    RestangularProvider.configuration.getIdFromElem = function(elem) {
      // if route is customers ==> returns customerID
      return elem[_.initial(elem.route).join('') + "ID"];
    }
    ````

    With that, you'd get what you need :)

??? faq "如何使用Restangular在我的请求中发送文件？"

    This can be done using the customPOST / customPUT method. Look at the following example:

    ````js
    Restangular.all('users')
    .customPOST(formData, undefined, undefined, { 'Content-Type': undefined });
    ````

    This basically tells the request to use the *Content-Type: multipart/form-data* as the header. Also *formData* is the body of the request, be sure to add all the params here, including the File you want to send of course.

??? faq "如何处理Restangular返回的List中的CRUD操作？"

    ````javascript
    Restangular.all('users').getList().subscribe( users => {
      this.users = users;
      var userWithId = _.find(users, function(user) {
        return user.id === 123;
      });

      userWithId.name = "Gonto";
      userWithId.put();

      // Alternatively delete the element from the list when finished
      userWithId.remove().subscribe( () => {
        // Updating the list and removing the user after the response is OK.
        this.users = _.without(this.users, userWithId);
      });

    });
    ````

??? faq "从集合中删除元素，保持集合的重新组合"

    While the example above removes the deleted user from the collection, it also overwrites the collection object with a plain array (because of `_.without`) which no longer knows about its Restangular attributes.

    If want to keep the restangularized collection, remove the element by modifying the collection in place:

    ```javascript
    userWithId.remove().subscribe( () => {
      let index = $scope.users.indexOf(userWithId);
      if (index > -1) this.users.splice(index, 1);
    });
    ```

??? faq "如何访问`unrestangularized`元素以及`restangularized`元素？"

    In order to get this done, you need to use the `responseExtractor`. You need to set a property there that will point to the original response received. Also, you need to actually copy this response as that response is the one that's going to be `restangularized` later

    ```javascript
    RestangularProvider.setResponseExtractor( (response) => {
      var newResponse = response;
      if (_.isArray(response)) {
        _.forEach(newResponse, function(value, key) {
          newResponse[key].originalElement = _.clone(value);
        });
      } else {
        newResponse.originalElement = _.clone(response);
      }

      return newResponse;
    });
    ```

    Alternatively, if you just want the stripped out response on any given call, you can use the .plain() method, doing something like this:

    ````javascript

    this.showData = function () {
      baseUrl.post(someData).subscribe( (response) => {
        console.log(response.plain());
      });
    };
    ````

??? faq "如何在请求中添加withCredentials参数？"

    ````javascript
    // Function for settting the default restangular configuration
    export function RestangularConfigFactory (RestangularProvider) {
      // Adding withCredential parametr to all Restangular requests
      RestangularProvider.setDefaultHttpFields({ withCredentials: true });
    }

    @NgModule({
      bootstrap: [ AppComponent ],
      declarations: [
        AppComponent,
      ],
      imports: [
        // Global configuration
        RestangularModule.forRoot(RestangularConfigFactory),
      ]
    })
    export class AppModule {}
    ````