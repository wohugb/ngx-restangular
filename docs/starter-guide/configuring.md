# 配置Restangular

## 属性

Restangular comes with defaults for all of its properties but you can configure them. **So, if you don't need to configure something, there's no need to add the configuration.**
You can set all these configurations in **[RestangularModule](#how-to-configure-them-globally) to change the global configuration**, you can also **use the [withConfig](#how-to-create-a-restangular-service-with-a-different-configuration-from-the-global-one) method in Restangular service to create a new Restangular service with some scoped configuration** or **use [withConfig](#withconfig) in component to make specified Restangular**

### withConfig

You can configure Restangular "withConfig" like in example below, you can also configure them globally [RestangularModule](#how-to-configure-them-globally) or in service with [withConfig](#how-to-create-a-restangular-service-with-a-different-configuration-from-the-global-one)

````javascript
// Function for settting the default restangular configuration
export function RestangularConfigFactory (RestangularProvider) {
  RestangularProvider.setBaseUrl('http://www.google.com');
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
// Let's use it in the component
@Component({
  ...
})
export class OtherComponent {
  constructor(private restangular: Restangular) {}

  ngOnInit() {
    restangular.withConfig((RestangularConfigurer) => {
      RestangularConfigurer.setBaseUrl('http://www.bing.com');
    }).all('users').getList()
  }
};
````

### setBaseUrl

The base URL for all calls to your API. For example if your URL for fetching accounts is http://example.com/api/v1/accounts, then your baseUrl is `/api/v1`. The default baseUrl is an empty string which resolves to the same url that Angular2 is running, but you can also set an absolute url like `http://api.example.com/api/v1` if you need to set another domain.

### setExtraFields

These are the fields that you want to save from your parent resources if you need to display them. By default this is an Empty Array which will suit most cases

### setParentless

Use this property to control whether Restangularized elements to have a parent or not. So, for example if you get an account and then get a nested list of buildings, you may want the buildings URL to be simple `/buildings/123` instead of `/accounts/123/buildings/123`. This property lets you do that.

This method accepts 1 parameter, it could be:

* Boolean: Specifies if all elements should be parentless or not
* Array: Specifies the routes (types) of all elements that should be parentless. For example `['buildings']`

### addElementTransformer

This is a hook. After each element has been "restangularized" (Added the new methods from Restangular), the corresponding transformer will be called if it fits.

This should be used to add your own methods / functions to entities of certain types.

You can add as many element transformers as you want. The signature of this method can be one of the following:

* **addElementTransformer(route, transformer)**: Transformer is called with all elements that have been restangularized, no matter if they're collections or not.

* **addElementTransformer(route, isCollection, transformer)**: Transformer is called with all elements that have been restangularized and match the specification regarding if it's a collection or not (true | false)

### setTransformOnlyServerElements

This sets whether transformers will be run for local objects and not by objects returned by the server. This is by default true but can be changed to false if needed (Most people won't need this).

### setOnElemRestangularized

This is a hook. After each element has been "restangularized" (Added the new methods from Restangular), this will be called. It means that if you receive a list of objects in one call, this method will be called first for the collection and then for each element of the collection.

**I favor the usage of `addElementTransformer` instead of `onElemRestangularized` whenever possible as the implementation is much cleaner.**

This callback is a function that has 4 parameters:

* **elem**: The element that has just been restangularized. Can be a collection or a single element.
* **isCollection**: Boolean indicating if this is a collection or a single element.
* **what**: The model that is being modified. This is the "path" of this resource. For example `buildings`
* **Restangular**: The instanced service to use any of its methods

This can be used together with `addRestangularMethod` (Explained later) to add custom methods to an element

````javascript
service.setOnElemRestangularized((element, isCollection, what, Restangular) => {
  element.newField = "newField";
  return element;
});
````

### addResponseInterceptor

The responseInterceptor is called after we get each response from the server. It's a function that receives this arguments:

* **data**: The data received got from the server
* **operation**: The operation made. It'll be the HTTP method used except for a `GET` which returns a list of element which will return `getList` so that you can distinguish them.
* **what**: The model that's being requested. It can be for example: `accounts`, `buildings`, etc.
* **url**: The relative URL being requested. For example: `/api/v1/accounts/123`
* **response**: Full server response including headers

Some of the use cases of the responseInterceptor are handling wrapped responses and enhancing response elements with more methods among others.

The responseInterceptor must return the restangularized data element.

````javascript
 RestangularProvider.addResponseInterceptor((data, operation, what, url, response)=> {
       return data;
     });
 });
````

### addFullRequestInterceptor

This adds a new fullRequestInterceptor. The fullRequestInterceptor is similar to the `requestInterceptor` but more powerful. It lets you change the element, the request parameters and the headers as well.

It's a function that receives the same as the `requestInterceptor` plus the headers and the query parameters (in that order).

It can return an object with any (or all) of following properties:

* **headers**: The headers to send
* **params**: The request parameters to send
* **element**: The element to send

````javascript
RestangularProvider.addFullRequestInterceptor((element, operation, path, url, headers, params)=> {
   return {
     params: Object.assign({}, params, {sort:"name"}),
     headers: headers,
     element: element
   }
 });
````

If a property isn't returned, the one sent is used.

### addErrorInterceptor

The errorInterceptor is called whenever there's an error. It's a function that receives the response, subject and the Restangular-response handler as parameters.

The errorInterceptor function, whenever it returns false, prevents the observable linked to a Restangular request to be executed. All other return values (besides false) are ignored and the observable follows the usual path, eventually reaching the success or error hooks.

The refreshAccesstoken function must return observable. It`s function that will be done before repeating the request, there you can make some actions. In switchMap you might do some transformations to request.

````javascript
// Function for settting the default restangular configuration
export function RestangularConfigFactory (RestangularProvider, authService) {
  RestangularProvider.setBaseUrl('http://api.test.com/v1');

  // This function must return observable
  var refreshAccesstoken = function () {
    // Here you can make action before repeated request
    return authService.functionForTokenUpdate();
  };

  RestangularProvider.addErrorInterceptor((response, subject, responseHandler) => {
    if (response.status === 403) {

      refreshAccesstoken()
      .switchMap(refreshAccesstokenResponse => {
        //If you want to change request or make with it some actions and give the request to the repeatRequest func.
        //Or you can live it empty and request will be the same.

        // update Authorization header
        response.request.headers.set('Authorization', 'Bearer ' + refreshAccesstokenResponse)

        return response.repeatRequest(response.request);
      })
      .subscribe(
        res => responseHandler(res),
        err => subject.error(err)
      );

      return false; // error handled
    }
    return true; // error not handled
  });
}

// AppModule is the main entry point into Angular2 bootstraping process
@NgModule({
  bootstrap: [ AppComponent ],
  imports: [
    // Importing RestangularModule and making default configs for restanglar
    RestangularModule.forRoot([authService], RestangularConfigFactory),
  ],
})
````

### setRestangularFields

Restangular required 3 fields for every "Restangularized" element. These are:

* id: Id of the element. Default: id
* route: Name of the route of this element. Default: route
* parentResource: The reference to the parent resource. Default: parentResource
* restangularCollection: A boolean indicating if this is a collection or an element. Default: restangularCollection
* cannonicalId: If available, the path to the cannonical ID to use. Useful for PK changes
* etag: Where to save the ETag received from the server. Defaults to `restangularEtag`
* selfLink: The path to the property that has the URL to this item. If your REST API doesn't return a URL to an item, you can just leave it blank. Defaults to `href`

Also all of Restangular methods and functions are configurable through restangularFields property.
All of these fields except for `id` and `selfLink` are handled by Restangular, so most of the time you won't change them. You can configure the name of the property that will be binded to all of this fields by setting restangularFields property.

### setMethodOverriders

You can now Override HTTP Methods. You can set here the array of methods to override. All those methods will be sent as POST and Restangular will add an X-HTTP-Method-Override header with the real HTTP method we wanted to do.

````javascript
RestangularProvider.setMethodOverriders(["Get","Put"]);
````

### setDefaultRequestParams

You can set default Query parameters to be sent with every request and every method.

Additionally, if you want to configure request params per method, you can use `requestParams` configuration similar to `$http`. For example `RestangularProvider.requestParams.get = {single: true}`.

Supported method to configure are: remove, get, post, put, common (all)

````javascript
// set params for multiple methods at once
RestangularProvider.setDefaultRequestParams(['remove', 'post'], {confirm: true});

// set only for get method
RestangularProvider.setDefaultRequestParams('get', {limit: 10});

// or for all supported request methods
RestangularProvider.setDefaultRequestParams({apikey: "secret key"});
````

### setFullResponse

You can set fullResponse to true to get the whole response every time you do any request. The full response has the restangularized data in the `data` field, and also has the headers and config sent. By default, it's set to false. Please note that in order for Restangular to access custom HTTP headers, your server must respond having the `Access-Control-Expose-Headers:` set.

````javascript
// set params for multiple methods at once
RestangularProvider.setFullResponse(true);
````

Or set it per service
````javascript
// Restangular factory that uses setFullResponse
export const REST_FUL_RESPONSE = new InjectionToken<any>('RestFulResponse');
export function RestFulResponseFactory(restangular: Restangular) {
  return restangular.withConfig((RestangularConfigurer) => {
    RestangularConfigurer.setFullResponse(true);
  });
}


// Configure factory in AppModule module
// AppModule is the main entry point into Angular2 bootstraping process
@NgModule({
  bootstrap: [ AppComponent ],
  declarations: [
    AppComponent,
  ],
  imports: [RestangularModule],
  providers: [
    { provide: REST_FUL_RESPONSE, useFactory:  RestFulResponseFactory, deps: [Restangular] }
  ]
})
export class AppModule {}


// Let's use it in the component
@Component({
  ...
})
export class OtherComponent {
  users;

  constructor(@Inject(REST_FUL_RESPONSE) public restFulResponse) {
  }

  ngOnInit() {
    this.restFulResponse.all('users').getList().subscribe( response => {
      this.users = response.data;
      console.log(response.headers);
    });
  }
}
````

### setDefaultHeaders

You can set default Headers to be sent with every request. Send format: {header_name: header_value}

````javascript
import { NgModule } from '@angular/core';
import { RestangularModule, Restangular } from 'ngx-restangular';

// Function for settting the default restangular configuration
export function RestangularConfigFactory (RestangularProvider) {
  RestangularProvider.setDefaultHeaders({'Authorization': 'Bearer UDXPx-Xko0w4BRKajozCVy20X11MRZs1'});
}

// AppModule is the main entry point into Angular2 bootstraping process
@NgModule({
  ...
  imports: [
    // Importing RestangularModule and making default configs for restanglar
    RestangularModule.forRoot(RestangularConfigFactory),
  ]
})
export class AppModule {
}
````

### setRequestSuffix

If all of your requests require to send some suffix to work, you can set it here. For example, if you need to send the format like `/users/123.json` you can add that `.json` to the suffix using the `setRequestSuffix` method

### setUseCannonicalId

You can set this to either `true` or `false`. By default it's false. If set to true, then the cannonical ID from the element will be used for URL creation (in DELETE, PUT, POST, etc.). What this means is that if you change the ID of the element and then you do a put, if you set this to true, it'll use the "old" ID which was received from the server. If set to false, it'll use the new ID assigned to the element.

### setPlainByDefault

You can set this to `true` or `false`. By default it's false. If set to true, data retrieved will be returned with no embed methods from restangular.

### setEncodeIds

You can set here if you want to URL Encode IDs or not. By default, it's true.

## 访问配置

You can also access the configuration via `RestangularModule` and `Restangular.provider` via the `configuration` property if you don't want to use the setters. Check it out:

````js
RestangularProvider.configuration.requestSuffix = '/';
````

## 如何全局配置它们

You can configure this in either the `AppModule`.

### 配置在 `AppModule`

````javascript
import { RestangularModule } from 'ngx-restangular';

// Function for settting the default restangular configuration
export function RestangularConfigFactory (RestangularProvider) {
  RestangularProvider.setBaseUrl('http://api.restngx.local/v1');
  RestangularProvider.setDefaultHeaders({'Authorization': 'Bearer UDXPx-Xko0w4BRKajozCVy20X11MRZs1'});
}

// AppModule is the main entry point into Angular2 bootstraping process
@NgModule({
  bootstrap: [ AppComponent ],
  declarations: [
    AppComponent,
  ],
  imports: [
    RestangularModule.forRoot(RestangularConfigFactory),
  ]
})
export class AppModule {
}
````

### Configuring in the `AppModule` with Dependency Injection applied

````javascript
import { RestangularModule } from 'ngx-restangular';

// Function for settting the default restangular configuration
export function RestangularConfigFactory (RestangularProvider, http) {
  RestangularProvider.setBaseUrl('http://api.restngx.local/v1');
  RestangularProvider.setDefaultHeaders({'Authorization': 'Bearer UDXPx-Xko0w4BRKajozCVy20X11MRZs1'});

  // Example of using Http service inside global config restangular
  RestangularProvider.addElementTransformer('me', true, ()=>{
    return http.get('http://api.test.com/v1/users/2', {});
  });
}

// AppModule is the main entry point into Angular2 bootstraping process
@NgModule({
  bootstrap: [ AppComponent ],
  declarations: [
    AppComponent,
  ],
  imports: [
    RestangularModule.forRoot([Http], RestangularConfigFactory),
  ]
})
export class AppModule {
}
````

## 如何使用与全局服务不同的配置创建Restangular服务

Let's assume that for most requests you need some configuration (The global one), and for just a bunch of methods you need another configuration. In that case, you'll need to create another Restangular service with this particular configuration. This scoped configuration will inherit all defaults from the global one. Let's see how.

````javascript
// Function for settting the default restangular configuration
export function RestangularConfigFactory (RestangularProvider) {
  RestangularProvider.setBaseUrl('http://www.google.com');
}

//Restangular service that uses Bing
export const RESTANGULAR_BING = new InjectionToken<any>('RestangularBing');
export function RestangularBingFactory(restangular: Restangular) {
  return restangular.withConfig((RestangularConfigurer) => {
     RestangularConfigurer.setBaseUrl('http://www.bing.com');
   });
}


// AppModule is the main entry point into Angular2 bootstraping process
@NgModule({
  bootstrap: [ AppComponent ],
  declarations: [
    AppComponent,
  ],
  imports: [
    // Global configuration
    RestangularModule.forRoot(RestangularConfigFactory),
  ],
  providers: [
    { provide: RESTANGULAR_BING, useFactory:  RestangularBingFactory, deps: [Restangular] }
  ]
})
export class AppModule {}


// Let's use it in the component
@Component({
  ...
})
export class OtherComponent {
  constructor(
    @Inject(Restangular) public Restangular,
    @Inject(RESTANGULAR_BING) public RestangularBing
  ) {}

  ngOnInit() {
    // GET to http://www.google.com/users
    // Uses global configuration
    Restangular.all('users').getList()

    // GET to http://www.bing.com/users
    // Uses Bing configuration which is based on Global one, therefore .json is added.
    RestangularBing.all('users').getList()
  }
};
````

## 解耦的Restangular服务

There're some times where you want to use Restangular but you don't want to expose Restangular object anywhere. For those cases, you can actually use the `service` feature of Restangular.

Let's see how it works:

````js
// Restangular factory that uses Users
export const USER_REST = new InjectionToken<any>('UserRest');
export function UserRestFactory(restangular: Restangular) {
  return restangular.service('users');
}


// AppModule is the main entry point into Angular2 bootstraping process
@NgModule({
  bootstrap: [ AppComponent ],
  declarations: [
    AppComponent,
  ],
  imports: [RestangularModule],
  providers: [
    { provide: USER_REST, useFactory:  UserRestFactory, deps: [Restangular] } // Configurating our factory
  ]
})
export class AppModule {
}


// Let's use it in the component
export class OtherComponent {
  constructor(@Inject(USER_REST) public User) {
    Users.one(2).get() // GET to /users/2
    Users.post({data}) // POST to /users

    // GET to /users
    Users.getList().subscribe( users => {
      var user = users[0]; // user === {id: 1, name: "Tonto"}
      user.name = "Gonto";
      // PUT to /users/1
      user.put();
    })
  }
}
````

We can also use Nested RESTful resources with this:

````js
var Cars = Restangular.service('cars', Restangular.one('users', 1));

Cars.getList() // GET to /users/1/cars
````

