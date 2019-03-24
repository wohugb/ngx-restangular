# 方法说明

有3套方法。
集合有一些方法，元素有其他。
所有这些方法也有一些共同的方法

## Restangular 方法

这些是可以在Restangular对象上调用的方法。

* **one(route, id)**: 这将创建一个新的Restangular对象，它只是一个指向一个元素的指针，该元素具有路由`route`和指定的id。
* **all(route)**: 这将创建一个新的Restangular对象，它只是指向指定路径的元素列表的指针。
* **oneUrl(route, url)**: This will create a new Restangular object that is just a pointer to one element with the specified URL.
* **allUrl(route, url)**: This creates a Restangular object that is just a pointer to a list at the specified URL.
* **copy(fromElement)**: This will create a copy of the from element so that we can modify the copied one.
* **restangularizeElement(parent, element, route, queryParams)**: Restangularizes a new element
* **restangularizeCollection(parent, element, route, queryParams)**: Restangularizes a new collection

## 元素方法

* **get([queryParams, headers])**: Gets the element. Query params and headers are optionals
* **getList(subElement, [queryParams, headers])**: Gets a nested resource. subElement is mandatory. **It's a string with the name of the nested resource (and URL)**. For example `buildings`
* **put([queryParams, headers])**: Does a put to the current element
* **post(subElement, elementToPost, [queryParams, headers])**: Does a POST and creates a subElement. Subelement is mandatory and is the nested resource. Element to post is the object to post to the server
* **remove([queryParams, headers])**: Does a DELETE. By default, `remove` sends a request with an empty object, which may cause problems with some servers or browsers. [This](https://github.com/mgonto/restangular/issues/193) shows how to configure RESTangular to have no payload.
* **head([queryParams, headers])**: Does a HEAD
* **trace([queryParams, headers])**: Does a TRACE
* **options([queryParams, headers])**: Does a OPTIONS
* **patch(object, [queryParams, headers])**: Does a PATCH
* **one(route, id)**: Used for RequestLess connections and URL Building. See section below.
* **all(route)**: Used for RequestLess connections and URL Building. See section below.
* **several(route, ids*)**: Used for RequestLess connections and URL Building. See section below.
* **oneUrl(route, url)**: This will create a new Restangular object that is just a pointer to one element with the specified URL.
* **allUrl(route, url)**: This creates a Restangular object that is just a pointer to a list at the specified URL.
* **getRestangularUrl()**: Gets the URL of the current object.
* **getRequestedUrl()**: Gets the real URL the current object was requested with (incl. GET parameters). Will equal getRestangularUrl() when no parameters were used, before calling `get()`, or when using on a nested child.
* **getParentList()**: Gets the parent list to which it belongs (if any)
* **clone()**: Copies the element. It's an alias to calling `Restangular.copy(elem)`.
* **plain()**: Returns the plain element received from the server without any of the enhanced methods from Restangular. It's an alias to calling `Restangular.stripRestangular(elem)`
* **save**: Calling save will determine whether to do PUT or POST accordingly

## 集合方法

* **getList([queryParams, headers]): Gets itself again (Remember this is a collection)**.
* **get(id): Gets one item from the collection by id**.
* **post(elementToPost, [queryParams, headers])**: Creates a new element of this collection.
* **head([queryParams, headers])**: Does a HEAD
* **trace: ([queryParams, headers])**: Does a TRACE
* **options: ([queryParams, headers])**: Does a OPTIONS
* **patch(object, [queryParams, headers])**: Does a PATCH
* **remove([queryParams, headers])**: Does a DELETE. By default, `remove` sends a request with an empty object, which may cause problems with some servers or browsers. [This](https://github.com/mgonto/restangular/issues/193) shows how to configure RESTangular to have no payload.
* **putElement(index, params, headers)**: Puts the element on the required index and returns a observable of the updated new array

    ````js
    Restangular.all('users').getList()
    .subscribe( users => {
      users.putElement(2, {'name': 'new name'});
    });
    ````

* **getRestangularUrl()**: Gets the URL of the current object.
* **getRequestedUrl()**: Gets the real URL the current object was requested with (incl. GET parameters). Will equal getRestangularUrl() when no parameters were used, before calling `getList()`, or when using on a nested child.
* **one(route, id)**: Used for RequestLess connections and URL Building. See section below.
* **all(route)**: Used for RequestLess connections and URL Building. See section below.
* **several(route, ids*)**: Used for RequestLess connections and URL Building. See section below.
* **oneUrl(route, url)**: This will create a new Restangular object that is just a pointer to one element with the specified URL.
* **allUrl(route, url)**: This creates a Restangular object that is just a pointer to a list at the specified URL.
* **clone()**: Copies the collection. It's an alias to calling `Restangular.copy(collection)`.

## 自定义方法

* **customGET(path, [params, headers])**: Does a GET to the specific path. Optionally you can set params and headers.
* **customGETLIST(path, [params, headers])**: Does a GET to the specific path. **In this case, you expect to get an array, not a single element**. Optionally you can set params and headers.
* **customDELETE(path, [params, headers])**: Does a DELETE to the specific path. Optionally you can set params and headers.
* **customPOST([elem, path, params, headers])**: Does a POST to the specific path. Optionally you can set params and headers and elem. Elem is the element to post. If it's not set, it's assumed that it's the element itself from which you're calling this function.
* **customPUT([elem, path, params, headers])**: Does a PUT to the specific path. Optionally you can set params and headers and elem. Elem is the element to post. If it's not set, it's assumed that it's the element itself from which you're calling this function.
* **customPATCH([elem, path, params, headers])**: Does a PATCH to the specific path. Accepts the same arguments as customPUT.
* **customOperation(operation, path, [params, headers, elem])**: This does a custom operation to the path that we specify. This method is actually used from all the others in this subsection. Operation can be one of: get, post, put, remove, head, options, patch, trace
* **addRestangularMethod(name, operation, [path, params, headers, elem])**: This will add a new restangular method to this object with the name `name` to the operation and path specified (or current path otherwise). There's a section on how to do this later.

Let's see an example of this:

````javascript
// GET /accounts/123/messages
Restangular.one("accounts", 123).customGET("messages")

// GET /accounts/messages?param=param2
Restangular.all("accounts").customGET("messages", {param: "param2"})
````
