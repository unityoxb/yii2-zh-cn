实现 RESTful 风格的 Web 服务 API
=====================================

Yii 提供了一整套的工具来大幅简化有关生成 RESTful Web Service API （[REST风格网络服务API](http://zh.wikipedia.org/zh/REST)）的工作。
尤其是，Yii 为关于 rest 风格的 Api 的以下方面提供支持：

* 支持针对 ActiveRecord 快速构建常用的 Api 原型；
* 设置返回参数的格式 （原生默认支持JSON和XML两种格式）以及 API 的版本问询；
* 自定义对象的序列化过程，支持返回字段的选择性输出；
* 合理地格式化收集的数据与返还的有效性验证报错；
* 高效路由，并 HTTP 方法检查；
* 支持 HTTP 的 `OPTIONS` 和 `HEAD` 方法；
* 身份验证；
* 权限授予；
* HATEOAS（[英文维基百科](http://en.wikipedia.org/wiki/HATEOAS)，HATEOAS作为领域特定协议描述的引擎。这方面中文资料非常有限，译者找到了InfoQ 09年的一篇[翻译](http://www.infoq.com/cn/news/2009/06/hateoas-dsl-protocol-description)，推荐给大家）的支持；
* 通过 `yii\web\HttpCache` 缓存；
* 查询速率限制 ；
* 搜索和筛选： 待定...
* 测试： 待定...
* 自动生成的 API 文档： 待定...


简明案例：
---------------

让我们用一个简单的例子来展示如何使用 Yii 建立 RESTful APIs。
如果你想通过 RESTful APIs 来输出 user 数据，user 数据必须在数据库 user 表中存在，
并且已经建立好 ActiveRecord class(数据模型类) `app\models\User` 供程序访问 user 中的数据。

第一，创建 controller class(控制器类) `app\controllers\UserController` 如下，

```php
namespace app\controllers;

use yii\rest\ActiveController;

class UserController extends ActiveController
{
    public $modelClass = 'app\models\User';
}
```

然后，修改配置文件中的 `urlManager` 如下：

```php
'urlManager' => [
    'enablePrettyUrl' => true,
    'enableStrictParsing' => true,
    'showScriptName' => false,
    'rules' => [
        ['class' => 'yii\rest\UrlRule', 'controller' => 'user'],
    ],
]
```

通过上面步骤的配置，你已经完成了创建 RESTful APIs 访问 user 数据的接口。
创建完成的 APIs 接口包括：

* `GET /users`: list all users page by page;列出所有 users 数据；
* `HEAD /users`: show the overview information of user listing;
* `POST /users`: 创建一个新 user；
* `GET /users/123`: 返回 user 123 的详细数据；
* `HEAD /users/123`: show the overview information of user 123;
* `PATCH /users/123` and `PUT /users/123`: 更新 user 123 的数据；
* `DELETE /users/123`: 删除 user 123；
* `OPTIONS /users`: show the supported verbs regarding endpoint `/users`;
* `OPTIONS /users/123`: show the supported verbs regarding endpoint `/users/123`.

你可以访问你的 APIs 接口通过 `curl` 命令如下，

```
curl -i -H "Accept:application/json" "http://localhost/users"
```

输出的信息如下：

```
HTTP/1.1 200 OK
Date: Sun, 02 Mar 2014 05:31:43 GMT
Server: Apache/2.2.26 (Unix) DAV/2 PHP/5.4.20 mod_ssl/2.2.26 OpenSSL/0.9.8y
X-Powered-By: PHP/5.4.20
X-Pagination-Total-Count: 1000
X-Pagination-Page-Count: 50
X-Pagination-Current-Page: 1
X-Pagination-Per-Page: 20
Link: <http://localhost/users?page=1>; rel=self, 
      <http://localhost/users?page=2>; rel=next, 
      <http://localhost/users?page=50>; rel=last
Transfer-Encoding: chunked
Content-Type: application/json; charset=UTF-8

[
    {
        "id": 1,
        ...
    },
    {
        "id": 2,
        ...
    },
    ...
]
```

尝试改变 content type 为 `application/xml`, 你可以看到返回结果是 XML 数据格式。

```
curl -i -H "Accept:application/xml" "http://localhost/users"
```

```
HTTP/1.1 200 OK
Date: Sun, 02 Mar 2014 05:31:43 GMT
Server: Apache/2.2.26 (Unix) DAV/2 PHP/5.4.20 mod_ssl/2.2.26 OpenSSL/0.9.8y
X-Powered-By: PHP/5.4.20
X-Pagination-Total-Count: 1000
X-Pagination-Page-Count: 50
X-Pagination-Current-Page: 1
X-Pagination-Per-Page: 20
Link: <http://localhost/users?page=1>; rel=self, 
      <http://localhost/users?page=2>; rel=next, 
      <http://localhost/users?page=50>; rel=last
Transfer-Encoding: chunked
Content-Type: application/xml

<?xml version="1.0" encoding="UTF-8"?>
<response>
    <item>
        <id>1</id>
        ...
    </item>
    <item>
        <id>2</id>
        ...
    </item>
    ...
</response>
```

> 小技巧：可以直接通过浏览器键入 URL `http://localhost/users` 访问 APIs 接口。

你可以看到，在返回的头部信息中，有总数，页面总数等等。
其中还包含了连接使你可以访问到其他页面，比如，`http://localhost/users?page=2` 可以访问到下一页的user 数据。

使用 `fields` 和 `expand` 参数, 你可以通过使用 `fields` 来筛选需要返回的数据结果。
比如，URL `http://localhost/users?fields=id,email` 只会返回 包含 `id` 和 `email` 的数据结果：


> 须知：你可能注意到通过 `http://localhost/users` 返回的数据中包含了部分敏感字段，
> 如 `password_hash`, `auth_key`。你肯定不希望这些出现在你的 API 返回数据中。
> 你可以通过过滤这部分的字段来达到目的，这部分会在后面的章节阐述。


在后面的章节中，我们为讲述更多关于实现 RESTful APIs 的细节。

总体结构（General Architecture）
--------------------

Using the Yii RESTful API framework, 
you implement an API endpoint in terms of a controller action, 
and you use a controller to organize the actions that implement the endpoints for a single type of resource.

Resources are represented as data models which extend from the [[yii\base\Model]] class.
If you are working with databases (relational or NoSQL), it is recommended you use ActiveRecord to represent resources.

You may use [[yii\rest\UrlRule]] to simplify the routing to your API endpoints.

While not required, it is recommended that you develop your RESTful APIs as an application, separated from
your Web front end and back end.


创建资源类（Creating Resource Classes）
-------------------------

RESTful APIs 都是关于访问和操作资源的，在 Yii 中的，资源可以是人一个类的对象，然而，如果你创建的资源类是从 [[yii\base\Model]] 或者其子类 （如：[[yii\db\ActiveRecord]]），你可以获得以下好处：

* 输入数据验证；
* 查询，创建，修改，和删除数据，前提是你的了类继承自 [[yii\db\ActiveRecord]]；
* 可定制的数据格式（会在下一节中讲到）。


格式化响应数据（Formatting Response Data）
------------------------

默认情况下，Yii 的RESTful APIs 支持两种响应数据格式：JSON 和 XML。
如果你想使用其他格式，你应该修改 [[yii\rest\Controller::supportedFormats]] 和 [[yii\web\Response::formatters]]。

格式化响应数据一般包含两个步骤：

1. 要返回给用户的对象（包括嵌入的对象）通过 [[yii\rest\Serializer]] 转换成数组格式。
2. 数组数据通过 [[yii\web\ResponseFormatterInterface|response formatters]] 转换成要返回的数据格式（例如：JSON，XML）

步骤2 通常是非常机械化的数据转换处理，可以通过内置的格式化处理方式很好地进行处理。
步骤1 涉及到一些比较重要的开发环节，主要详情如下，

When the [[yii\rest\Serializer|serializer]] converts an object into an array, it will call the `toArray()` method
of the object if it implements [[yii\base\Arrayable]]. If an object does not implement this interface,
its public properties will be returned instead.

For classes extending from [[yii\base\Model]] or [[yii\db\ActiveRecord]], besides directly overriding `toArray()`,
you may also override the `fields()` method and/or the `extraFields()` method to customize the data being returned.

The method [[yii\base\Model::fields()]] declares a set of *fields* that should be included in the result.
A field is simply a named data item. In a result array, the array keys are the field names, and the array values
are the corresponding field values. The default implementation of [[yii\base\Model::fields()]] is to return
all attributes of a model as the output fields; for [[yii\db\ActiveRecord::fields()]], by default it will return
the names of the attributes whose values have been populated into the object.

You can override the `fields()` method to add, remove, rename or redefine fields. For example,

```php
// explicitly list every field, best used when you want to make sure the changes
// in your DB table or model attributes do not cause your field changes (to keep API backward compatibility).
public function fields()
{
    return [
        // field name is the same as the attribute name
        'id',
        // field name is "email", the corresponding attribute name is "email_address"
        'email' => 'email_address',
        // field name is "name", its value is defined by a PHP callback
        'name' => function () {
            return $this->first_name . ' ' . $this->last_name;
        },
    ];
}

// filter out some fields, best used when you want to inherit the parent implementation
// and blacklist some sensitive fields.
public function fields()
{
    $fields = parent::fields();

    // remove fields that contain sensitive information
    unset($fields['auth_key'], $fields['password_hash'], $fields['password_reset_token']);

    return $fields;
}
```

The return value of `fields()` should be an array. The array keys are the field names, and the array values
are the corresponding field definitions which can be either property/attribute names or anonymous functions
returning the corresponding field values.

> Warning: Because by default all attributes of a model will be included in the API result, you should
> examine your data to make sure they do not contain sensitive information. If there is such information,
> you should override `fields()` or `toArray()` to filter them out. In the above example, we choose
> to filter out `auth_key`, `password_hash` and `password_reset_token`.

You may use the `fields` query parameter to specify which fields in `fields()` should be included in the result.
If this parameter is not specified, all fields returned by `fields()` will be returned.

The method [[yii\base\Model::extraFields()]] is very similar to [[yii\base\Model::fields()]].
The difference between these methods is that the latter declares the fields that should be returned by default,
while the former declares the fields that should only be returned when the user specifies them in the `expand` query parameter.

For example, `http://localhost/users?fields=id,email&expand=profile` may return the following JSON data:

```php
[
    {
        "id": 100,
        "email": "100@example.com",
        "profile": {
            "id": 100,
            "age": 30,
        }
    },
    ...
]
```

You may wonder who triggers the conversion from objects to arrays when an action returns an object or object collection.
The answer is that this is done by [[yii\rest\Controller::serializer]] in the [[yii\base\Controller::afterAction()|afterAction()]]
method. By default, [[yii\rest\Serializer]] is used as the serializer that can recognize resource objects extending from
[[yii\base\Model]] and collection objects implementing [[yii\data\DataProviderInterface]]. The serializer
will call the `toArray()` method of these objects and pass the `fields` and `expand` user parameters to the method.
If there are any embedded objects, they will also be converted into arrays recursively.

If all your resource objects are of [[yii\base\Model]] or its child classes, such as [[yii\db\ActiveRecord]],
and you only use [[yii\data\DataProviderInterface]] as resource collections, the default data formatting
implementation should work very well. However, if you want to introduce some new resource classes that do not
extend from [[yii\base\Model]], or if you want to use some new collection classes, you will need to
customize the serializer class and configure [[yii\rest\Controller::serializer]] to use it.
You new resource classes may use the trait [[yii\base\ArrayableTrait]] to support selective field output
as explained above.


### Pagination

For API endpoints about resource collections, pagination is supported out-of-box if you use 
[[yii\data\DataProviderInterface|data provider]] to serve the response data. In particular,
through query parameters `page` and `per-page`, an API consumer may specify which page of data
to return and how many data items should be included in each page. The corresponding response
will include the pagination information by the following HTTP headers (please also refer to the first example
in this chapter):

* `X-Pagination-Total-Count`: The total number of data items;
* `X-Pagination-Page-Count`: The number of pages;
* `X-Pagination-Current-Page`: The current page (1-based);
* `X-Pagination-Per-Page`: The number of data items in each page;
* `Link`: A set of navigational links allowing client to traverse the data page by page.

The response body will contain a list of data items in the requested page.

Sometimes, you may want to help simplify the client development work by including pagination information
directly in the response body. To do so, configure the [[yii\rest\Serializer::collectionEnvelope]] property
as follows:

```php
use yii\rest\ActiveController;

class UserController extends ActiveController
{
    public $modelClass = 'app\models\User';
    public $serializer = [
        'class' => 'yii\rest\Serializer',
        'collectionEnvelope' => 'items',
    ];
}
```

You may then get the following response for request `http://localhost/users`:

```
HTTP/1.1 200 OK
Date: Sun, 02 Mar 2014 05:31:43 GMT
Server: Apache/2.2.26 (Unix) DAV/2 PHP/5.4.20 mod_ssl/2.2.26 OpenSSL/0.9.8y
X-Powered-By: PHP/5.4.20
X-Pagination-Total-Count: 1000
X-Pagination-Page-Count: 50
X-Pagination-Current-Page: 1
X-Pagination-Per-Page: 20
Link: <http://localhost/users?page=1>; rel=self, 
      <http://localhost/users?page=2>; rel=next, 
      <http://localhost/users?page=50>; rel=last
Transfer-Encoding: chunked
Content-Type: application/json; charset=UTF-8

{
    "items": [
        {
            "id": 1,
            ...
        },
        {
            "id": 2,
            ...
        },
        ...
    ],
    "_links": {
        "self": "http://localhost/users?page=1", 
        "next": "http://localhost/users?page=2", 
        "last": "http://localhost/users?page=50"
    },
    "_meta": {
        "totalCount": 1000,
        "pageCount": 50,
        "currentPage": 1,
        "perPage": 20
    }
}
```


### HATEOAS Support

[HATEOAS](http://en.wikipedia.org/wiki/HATEOAS), an abbreviation for Hypermedia as the Engine of Application State,
promotes that RESTful APIs should return information that allow clients to discover actions supported for the returned
resources. The key of HATEOAS is to return a set of hyperlinks with relation information when resource data are served
by APIs.

You may let your model classes to implement the [[yii\web\Linkable]] interface to support HATEOAS. By implementing
this interface, a class is required to return a list of [[yii\web\Link|links]]. Typically, you should return at least
the `self` link, for example:

```php
use yii\db\ActiveRecord;
use yii\web\Link;
use yii\web\Linkable;
use yii\helpers\Url;

class User extends ActiveRecord implements Linkable
{
    public function getLinks()
    {
        return [
            Link::REL_SELF => Url::to(['user', 'id' => $this->id], true),
        ];
    }
}
```

When a `User` object is returned in a response, it will contain a `_links` element representing the links related
to the user, for example,

```
{
    "id": 100,
    "email": "user@example.com",
    ...,
    "_links" => [
        "self": "https://example.com/users/100"
    ]
}
```


Creating Controllers and Actions
--------------------------------

So you have the resource data and you have specified how the resource data should be formatted, the next thing
to do is to create controller actions to expose the resource data to end users.

Yii provides two base controller classes to simplify your work of creating RESTful actions:
[[yii\rest\Controller]] and [[yii\rest\ActiveController]]. The difference between these two controllers
is that the latter provides a default set of actions that are specified designed to deal with
resources represented as ActiveRecord. So if you are using ActiveRecord and you are comfortable with
the provided built-in actions, you may consider creating your controller class by extending from
the latter. Otherwise, extending from [[yii\rest\Controller]] will allow you to develop actions
from scratch.

Both [[yii\rest\Controller]] and [[yii\rest\ActiveController]] provide the following features which will
be described in detail in the next few sections:

* Response format negotiation;
* API version negotiation;
* HTTP method validation;
* User authentication;
* Rate limiting.

[[yii\rest\ActiveController]] in addition provides the following features specifically for working
with ActiveRecord:

* A set of commonly used actions: `index`, `view`, `create`, `update`, `delete`, `options`;
* User authorization in regard to the requested action and resource.

When creating a new controller class, a convention in naming the controller class is to use
the type name of the resource and use singular form. For example, to serve user information,
the controller may be named as `UserController`.

Creating a new action is similar to creating an action for a Web application. The only difference
is that instead of rendering the result using a view by calling the `render()` method, for RESTful actions
you directly return the data. The [[yii\rest\Controller::serializer|serializer]] and the
[[yii\web\Response|response object]] will handle the conversion from the original data to the requested
format. For example,

```php
public function actionSearch($keyword)
{
    $result = SolrService::search($keyword);
    return $result;
}
```

If your controller class extends from [[yii\rest\ActiveController]], you should set
its [[yii\rest\ActiveController::modelClass||modelClass]] property to be the name of the resource class
that you plan to serve through this controller. The class must implement [[yii\db\ActiveRecordInterface]].

With [[yii\rest\ActiveController]], you may want to disable some of the built-in actions or customize them.
To do so, override the `actions()` method like the following:

```php
public function actions()
{
    $actions = parent::actions();

    // disable the "delete" and "create" actions
    unset($actions['delete'], $actions['create']);

    // customize the data provider preparation with the "prepareDataProvider()" method
    $actions['index']['prepareDataProvider'] = [$this, 'prepareDataProvider'];

    return $actions;
}

public function prepareDataProvider()
{
    // prepare and return a data provider for the "index" action
}
```

The following list summarizes the built-in actions supported by [[yii\rest\ActiveController]]:

* [[yii\rest\IndexAction|index]]: list resources page by page;
* [[yii\rest\ViewAction|view]]: return the details of a specified resource;
* [[yii\rest\CreateAction|create]]: create a new resource;
* [[yii\rest\UpdateAction|update]]: update an existing resource;
* [[yii\rest\DeleteAction|delete]]: delete the specified resource;
* [[yii\rest\OptionsAction|options]]: return the supported HTTP methods.


Routing
-------

With resource and controller classes ready, you can access the resources using the URL like
`http://localhost/index.php?r=user/create`. As you can see, the format of the URL is the same as that
for Web applications.

In practice, you usually want to enable pretty URLs and take advantage of HTTP verbs.
For example, a request `POST /users` would mean accessing the `user/create` action.
This can be done easily by configuring the `urlManager` application component in the application
configuration like the following:

```php
'urlManager' => [
    'enablePrettyUrl' => true,
    'enableStrictParsing' => true,
    'showScriptName' => false,
    'rules' => [
        ['class' => 'yii\rest\UrlRule', 'controller' => 'user'],
    ],
]
```

Compared to the URL management for Web applications, the main new thing above is the use of
[[yii\rest\UrlRule]] for routing RESTful API requests. This special URL rule class will
create a whole set of child URL rules to support routing and URL creation for the specified controller(s).
For example, the above code is roughly equivalent to the following rules:

```php
[
    'PUT,PATCH users/<id>' => 'user/update',
    'DELETE users/<id>' => 'user/delete',
    'GET,HEAD users/<id>' => 'user/view',
    'POST users' => 'user/create',
    'GET,HEAD users' => 'user/index',
    'users/<id>' => 'user/options',
    'users' => 'user/options',
]
```

And the following API endpoints are supported by this rule:

* `GET /users`: list all users page by page;
* `HEAD /users`: show the overview information of user listing;
* `POST /users`: create a new user;
* `GET /users/123`: return the details of the user 123;
* `HEAD /users/123`: show the overview information of user 123;
* `PATCH /users/123` and `PUT /users/123`: update the user 123;
* `DELETE /users/123`: delete the user 123;
* `OPTIONS /users`: show the supported verbs regarding endpoint `/users`;
* `OPTIONS /users/123`: show the supported verbs regarding endpoint `/users/123`.

You may configure the `only` and `except` options to explicitly list which actions to support or which
actions should be disabled, respectively. For example,

```php
[
    'class' => 'yii\rest\UrlRule',
    'controller' => 'user',
    'except' => ['delete', 'create', 'update'],
],
```

You may also configure `patterns` or `extraPatterns` to redefine existing patterns or add new patterns supported by this rule.
For example, to support a new action `search` by the endpoint `GET /users/search`, configure the `extraPatterns` option as follows,

```php
[
    'class' => 'yii\rest\UrlRule',
    'controller' => 'user',
    'extraPatterns' => [
        'GET search' => 'search',
    ],
```

You may have noticed that the controller ID `user` appears in plural form as `users` in the endpoints.
This is because [[yii\rest\UrlRule]] automatically pluralizes controller IDs for them to use in endpoints.
You may disable this behavior by setting [[yii\rest\UrlRule::pluralize]] to be false, or if you want
to use some special names you may configure the [[yii\rest\UrlRule::controller]] property.


Authentication
--------------

Unlike Web applications, RESTful APIs should be stateless, which means sessions or cookies should not
be used. Therefore, each request should come with some sort of authentication credentials because
the user authentication status may not be maintained by sessions or cookies. A common practice is
to send a secret access token with each request to authenticate the user. Since an access token
can be used to uniquely identify and authenticate a user, **the API requests should always be sent
via HTTPS to prevent from man-in-the-middle (MitM) attacks**.

There are different ways to send an access token:

* [HTTP Basic Auth](http://en.wikipedia.org/wiki/Basic_access_authentication): the access token
  is sent as the username. This is should only be used when an access token can be safely stored
  on the API consumer side. For example, the API consumer is a program running on a server.
* Query parameter: the access token is sent as a query parameter in the API URL, e.g.,
  `https://example.com/users?access-token=xxxxxxxx`. Because most Web servers will keep query
  parameters in server logs, this approach should be mainly used to serve `JSONP` requests which
  cannot use HTTP headers to send access tokens.
* [OAuth 2](http://oauth.net/2/): the access token is obtained by the consumer from an authorization
  server and sent to the API server via [HTTP Bearer Tokens](http://tools.ietf.org/html/rfc6750),
  according to the OAuth2 protocol.

Yii supports all of the above authentication methods and can be further extended to support other methods.

To enable authentication for your APIs, do the following two steps:

1. Configure [[yii\rest\Controller::authMethods]] with the authentication methods you plan to use.
2. Implement [[yii\web\IdentityInterface::findIdentityByAccessToken()]] in your [[yii\web\User::identityClass|user identity class]].

For example, to enable all three authentication methods explained above, you would configure `authMethods`
as follows,

```php
class UserController extends ActiveController
{
    public $authMethods = [
        'yii\rest\HttpBasicAuth',
        'yii\rest\QueryParamAuth',
        'yii\rest\HttpBearerAuth',
    ];
}
```

Each element in `authMethods` should be an auth class name or a configuration array. An auth class
must implement [[yii\rest\AuthInterface]].

Implementation of `findIdentityByAccessToken()` is application specific. For example, in simple scenarios
when each user can only have one access token, you may store the access token in an `access_token` column
in the user table. The method can then be readily implemented in the `User` class as follows,

```php
use yii\db\ActiveRecord;
use yii\web\IdentityInterface;

class User extends ActiveRecord implements IdentityInterface
{
    public static function findIdentityByAccessToken($token)
    {
        return static::find(['access_token' => $token]);
    }
}
```

After authentication is enabled as described above, for every API request, the requested controller
will try to authenticate the user in its `beforeAction()` step.

If authentication succeeds, the controller will perform other checks (such as rate limiting, authorization)
and then run the action. The authenticated user identity information can be retrieved via `Yii::$app->user->identity`.

If authentication fails, a response with HTTP status 401 will be sent back together with other appropriate headers
(such as a `WWW-Authenticate` header for HTTP Basic Auth).


Authorization
-------------

After a user is authenticated, you probably want to check if he has the permission to perform the requested
action for the requested resource. This process is called *authorization* which is covered in detail in
the [Authorization chapter](authorization.md).

You may use the Role-Based Access Control (RBAC) component to implementation authorization.

To simplify the authorization check, you may also override the [[yii\rest\Controller::checkAccess()]] method
and then call this method in places where authorization is needed. By default, the built-in actions provided
by [[yii\rest\ActiveController]] will call this method when they are about to run.

```php
/**
 * Checks the privilege of the current user.
 *
 * This method should be overridden to check whether the current user has the privilege
 * to run the specified action against the specified data model.
 * If the user does not have access, a [[ForbiddenHttpException]] should be thrown.
 *
 * @param string $action the ID of the action to be executed
 * @param \yii\base\Model $model the model to be accessed. If null, it means no specific model is being accessed.
 * @param array $params additional parameters
 * @throws ForbiddenHttpException if the user does not have access
 */
public function checkAccess($action, $model = null, $params = [])
{
}
```


Rate Limiting
-------------

To prevent abuse, you should consider adding rate limiting to your APIs. For example, you may limit the API usage
of each user to be at most 100 API calls within a period of 10 minutes. If too many requests are received from a user
within the period of the time, a response with status code 429 (meaning Too Many Requests) should be returned.

To enable rate limiting, the [[yii\web\User::identityClass|user identity class]] should implement [[yii\rest\RateLimitInterface]].
This interface requires implementation of the following three methods:

* `getRateLimit()`: returns the maximum number of allowed requests and the time period, e.g., `[100, 600]` means
  at most 100 API calls within 600 seconds.
* `loadAllowance()`: returns the number of remaining requests allowed and the corresponding UNIX timestamp
  when the rate limit is checked last time.
* `saveAllowance()`: saves the number of remaining requests allowed and the current UNIX timestamp.

You may use two columns in the user table to record the allowance and timestamp information.
And `loadAllowance()` and `saveAllowance()` can then be implementation by reading and saving the values
of the two columns corresponding to the current authenticated user. To improve performance, you may also
consider storing these information in cache or some NoSQL storage.

Once the identity class implements the required interface, Yii will automatically use the rate limiter
as specified by [[yii\rest\Controller::rateLimiter]] to perform rate limiting check. The rate limiter
will thrown a [[yii\web\TooManyRequestsHttpException]] if rate limit is exceeded.

When rate limiting is enabled, every response will be sent with the following HTTP headers containing
the current rate limiting information:

* `X-Rate-Limit-Limit`: The maximum number of requests allowed with a time period;
* `X-Rate-Limit-Remaining`: The number of remaining requests in the current time period;
* `X-Rate-Limit-Reset`: The number of seconds to wait in order to get the maximum number of allowed requests.


Error Handling
--------------

When handling a RESTful API request, if there is an error in the user request or if something unexpected
happens on the server, you may simply throw an exception to notify the user something wrong happened. 
If you can identify the cause of the error (e.g. the requested resource does not exist), you should 
consider throwing an exception with a proper HTTP status code (e.g. [[yii\web\NotFoundHttpException]] 
representing a 404 HTTP status code). Yii will send the response with the corresponding HTTP status
code and text. It will also include in the response body the serialized representation of the 
exception. For example,

```
HTTP/1.1 404 Not Found
Date: Sun, 02 Mar 2014 05:31:43 GMT
Server: Apache/2.2.26 (Unix) DAV/2 PHP/5.4.20 mod_ssl/2.2.26 OpenSSL/0.9.8y
Transfer-Encoding: chunked
Content-Type: application/json; charset=UTF-8

{
    "type": "yii\\web\\NotFoundHttpException",
    "name": "Not Found Exception",
    "message": "The requested resource was not found.",
    "code": 0,
    "status": 404
}
```

The following list summarizes the HTTP status code that are used by the Yii REST framework:

* `200`: OK. Everything worked as expected.
* `201`: A resource was successfully created in response to a `POST` request. The `Location` header
   contains the URL pointing to the newly created resource.
* `204`: The request is handled successfully and the response contains no body content (like a `DELETE` request).
* `304`: Resource was not modified. You can use the cached version.
* `400`: Bad request. This could be caused by various reasons from the user side, such as invalid JSON
   data in the request body, invalid action parameters, etc.
* `401`: Authentication failed.
* `403`: The authenticated user is not allowed to access the specified API endpoint.
* `404`: The requested resource does not exist.
* `405`: Method not allowed. Please check the `Allow` header for allowed HTTP methods.
* `415`: Unsupported media type. The requested content type or version number is invalid.
* `422`: Data validation failed (in response to a `POST` request, for example). Please check the response body for detailed error messages.
* `429`: Too many requests. The request is rejected due to rate limiting.
* `500`: Internal server error. This could be caused by internal program errors.


版本化
----------

您的 API 应该进行版本化。与普通的 Web 应用不同，你对客户端和服务器端的代码有全部的掌控能力
对于 API 而言，通常你对于请求你 API 的那些客户端的代码是没有任何掌控的。因此，
只要可能，都应该维护好 API 的 Backward Compatibility（向后兼容性，简称BC），且如果你的API添加了一些不支持反向兼容的更改，
你应该调高版本的序号。具体你可以参考 [Symantic Versioning（英文，语义化版本控制，简而言之，即x.y.z的三段式编号方法）](http://semver.org/) 的这篇文章，
来获取有关为你的 API 设计版本编号的更多信息。

Regarding how to implement API versioning, a common practice is to embed the version number in the API URLs.
For example, `http://example.com/v1/users` stands for `/users` API of version 1. Another method of API versioning
which gains momentum recently is to put version numbers in the HTTP request headers, typically through the `Accept` header,
like the following:

```
// via a parameter
Accept: application/json; version=v1
// via a vendor content type
Accept: application/vnd.company.myapp-v1+json
```

Both methods have pros and cons, and there are a lot of debates about them. Below we describe a practical strategy
of API versioning that is a kind of mix of these two methods:

* Put each major version of API implementation in a separate module whose ID is the major version number (e.g. `v1`, `v2`).
  Naturally, the API URLs will contain major version numbers.
* Within each major version (and thus within the corresponding module), use the `Accept` HTTP request header
  to determine the minor version number and write conditional code to respond to the minor versions accordingly.

For each module serving a major version, it should include the resource classes and the controller classes
serving for that specific version. To better separate code responsibility, you may keep a common set of
base resource and controller classes, and subclass them in each individual version module. Within the subclasses,
implement the concrete code such as `Model::fields()`. As a result, your code may be organized like the following:

```
api/
    common/
        controllers/
            UserController.php
            PostController.php
        models/
            User.php
            Post.php
    modules/
        v1/
            controllers/
                UserController.php
                PostController.php
            models/
                User.php
                Post.php
        v2/
            controllers/
                UserController.php
                PostController.php
            models/
                User.php
                Post.php
```

Your application configuration would look like:

```php
return [
    'modules' => [
        'v1' => [
            'basePath' => '@app/modules/v1',
        ],
        'v2' => [
            'basePath' => '@app/modules/v2',
        ],
    ],
    'components' => [
        'urlManager' => [
            'enablePrettyUrl' => true,
            'enableStrictParsing' => true,
            'showScriptName' => false,
            'rules' => [
                ['class' => 'yii\rest\UrlRule', 'controller' => ['v1/user', 'v1/post']],
                ['class' => 'yii\rest\UrlRule', 'controller' => ['v2/user', 'v2/post']],
            ],
        ],
    ],
];
```

As a result, `http://example.com/v1/users` will return the list of users in version 1, while
`http://example.com/v2/users` will return version 2 users.

Using modules, code for different major versions can be well isolated. And it is still possible
to reuse code across modules via common base classes and other shared classes.

To deal with minor version numbers, you may take advantage of the content type negotiation
feature provided by [[yii\rest\Controller]]:

* Specify a list of supported minor versions (within the major version of the containing module)
  via [[yii\rest\Controller::supportedVersions]].
* Get the version number by reading [[yii\rest\Controller::version]].
* In relevant code, such as actions, resource classes, serializers, etc., write conditional
  code according to the requested minor version number.

Since minor versions require maintaining backward compatibility, hopefully there are not much
version checks in your code. Otherwise, chances are that you may need to create a new major version.


Caching
-------


Documentation
-------------

Testing
-------
