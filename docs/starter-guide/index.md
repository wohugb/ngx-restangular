# 目录

- [懒惰的读者的快速配置](#quick-configuration-for-lazy-readers)
- [使用Restangular](#using-restangular)
  - [创建主Restangular对象](#creating-main-restangular-object)
  - [让代码与Observables!](#lets-code-with-observables)
  - [这是使用promises的代码示例!](#here-is-example-of-code-with-using-promises)
- [配置Restangular](#configuring-restangular)
  - [属性](#properties)
    - [withConfig](#withconfig)
    - [setBaseUrl](#setbaseurl)
    - [setExtraFields](#setextrafields)
    - [setParentless](#setparentless)
    - [addElementTransformer](#addelementtransformer)
    - [setTransformOnlyServerElements](#settransformonlyserverelements)
    - [setOnElemRestangularized](#setonelemrestangularized)
    - [addResponseInterceptor](#addresponseinterceptor)
    - [addFullRequestInterceptor](#addfullrequestinterceptor)
    - [addErrorInterceptor](#adderrorinterceptor)
    - [setRestangularFields](#setrestangularfields)
    - [setMethodOverriders](#setmethodoverriders)
    - [setDefaultRequestParams](#setdefaultrequestparams)
    - [setFullResponse](#setfullresponse)
    - [setDefaultHeaders](#setdefaultheaders)
    - [setRequestSuffix](#setrequestsuffix)
    - [setUseCannonicalId](#setusecannonicalid)
    - [setPlainByDefault](#setplainbydefault)
    - [setEncodeIds](#setencodeids)
  - [访问配置](#accessing-configuration)
  - [如何全局配置它们](#how-to-configure-them-globally)
    - [在AppModule中配置](#configuring-in-the-appmodule)
    - [在应用了依赖注入的AppModule中进行配置](#configuring-in-the-appmodule-with-dependency-injection-applied)
  - [如何使用与全局服务不同的配置创建Restangular服务](#how-to-create-a-restangular-service-with-a-different-configuration-from-the-global-one)
  - [解耦的Restangular服务](#decoupled-restangular-service)
- [方法说明](#methods-description)
  - [Restangular 方法](#restangular-methods)
  - [元素方法](#element-methods)
  - [收集方法](#collection-methods)
  - [自定义方法](#custom-methods)
- [复制元素](#copying-elements)
- [直接在具有Observables的模板中使用值](#using-values-directly-in-templates-with-observables)
- [URL构建](#url-building)
- [创建新的Restangular方法](#creating-new-restangular-methods)
- [将自定义方法添加到集合](#adding-custom-methods-to-collections)
  - [例](#example)
- [向模型添加自定义方法](#adding-custom-methods-to-models)
  - [例](#example-1)
- [FAQ](#faq)
  - [我该如何处理错误?](#how-can-i-handle-errors)
  - [我需要在每个Restangular请求中发送授权令牌，我该怎么做?](#i-need-to-send-authorization-token-in-every-restangular-request-how-can-i-do-this)
  - [我需要在每个Restangular请求中发送一个标头，我该怎么做?](#i-need-to-send-one-header-in-every-restangular-request-how-can-i-do-this)
  - [如何发送没有正文的删除?](#how-can-i-send-a-delete-without-a-body)
  - [我使用Mongo，元素的ID是_id而不是id作为默认值。 因此，请求被发送到未定义的路由](#i-use-mongo-and-the-id-of-the-elements-is-_id-not-id-as-the-default-therefore-requests-are-sent-to-undefined-routes)
  - [如果我的每个模型都有不同的ID名称，例如CustomerID for Customer，该怎么办？](#what-if-each-of-my-models-has-a-different-id-name-like-customerid-for-customer)
  - [如何使用Restangular在我的请求中发送文件?](#how-can-i-send-files-in-my-request-using-restangular)
  - [如何处理Restangular返回的List中的CRUD操作?](#how-do-i-handle-crud-operations-in-a-list-returned-by-restangular)
  - [从集合中删除元素，保持集合的重新组合](#removing-an-element-from-a-collection-keeping-the-collection-restangularized)
  - [如何访问unrestangularized元素以及restangularized元素?](#how-can-i-access-the-unrestangularized-element-as-well-as-the-restangularized-one)
  - [如何在请求中添加withCredentials参数?](#how-can-add-withcredentials-params-to-requests)