> vue-resouce 的Github ： https://github.com/pagekit/vue-resource 



具体的使用文档：

https://github.com/pagekit/vue-resource/blob/develop/docs/http.md



```javascript
  // GET /someUrl
  this.$http.get('/someUrl').then(response => {

    // get body data
    this.someData = response.body;

  }, response => {
    // error callback
  });


```

发送post请求：

```
postInfo() {
  var url = 'http://127.0.0.1:8899/api/post';
  // post 方法接收三个参数：
  // 参数1： 要请求的URL地址
  // 参数2： 要发送的数据对象
  // 参数3： 指定post提交的编码类型为 application/x-www-form-urlencoded
  this.$http.post(url, { name: 'zs' }, { emulateJSON: true }).then(res => {
    console.log(res.body);
  });
}
```
发送JSONP请求获取数据：

```
jsonpInfo() { // JSONP形式从服务器获取数据
  var url = 'http://127.0.0.1:8899/api/jsonp';
  this.$http.jsonp(url).then(res => {
    console.log(res.body);
  });
}
```
# HTTP

The http service can be used globally `Vue.http` or in a Vue instance `this.$http`.

## Usage

A Vue instance provides the `this.$http` service which can send HTTP requests. A request method call returns a [Promise](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Promise) that resolves to the response. Also the Vue instance will be automatically bound to `this` in all function callbacks.

```js
{
  // GET /someUrl
  this.$http.get('/someUrl').then(response => {
    // success callback
  }, response => {
    // error callback
  });
}
```

## Methods

Shortcut methods are available for all request types. These methods work globally or in a Vue instance.

```js
// global Vue object
Vue.http.get('/someUrl', [config]).then(successCallback, errorCallback);
Vue.http.post('/someUrl', [body], [config]).then(successCallback, errorCallback);

// in a Vue instance
this.$http.get('/someUrl', [config]).then(successCallback, errorCallback);
this.$http.post('/someUrl', [body], [config]).then(successCallback, errorCallback);
```
List of shortcut methods:

* `get(url, [config])`
* `head(url, [config])`
* `delete(url, [config])`
* `jsonp(url, [config])`
* `post(url, [body], [config])`
* `put(url, [body], [config])`
* `patch(url, [body], [config])`

## Config

| Parameter        | Type                           | Description                                                  |
| ---------------- | ------------------------------ | ------------------------------------------------------------ |
| url              | `string`                       | URL to which the request is sent                             |
| body             | `Object`, `FormData`, `string` | Data to be sent as the request body                          |
| headers          | `Object`                       | Headers object to be sent as HTTP request headers            |
| params           | `Object`                       | Parameters object to be sent as URL parameters               |
| method           | `string`                       | HTTP method (e.g. GET, POST, ...)                            |
| responseType     | `string`                       | Type of the response body (e.g. text, blob, json, ...)       |
| timeout          | `number`                       | Request timeout in milliseconds (`0` means no timeout)       |
| credentials      | `boolean`                      | Indicates whether or not cross-site Access-Control requests should be made using credentials |
| emulateHTTP      | `boolean`                      | Send PUT, PATCH and DELETE requests with a HTTP POST and set the `X-HTTP-Method-Override` header |
| emulateJSON      | `boolean`                      | Send request body as `application/x-www-form-urlencoded` content type |
| before           | `function(request)`            | Callback function to modify the request object before it is sent |
| uploadProgress   | `function(event)`              | Callback function to handle the [ProgressEvent](https://developer.mozilla.org/en-US/docs/Web/API/ProgressEvent) of uploads |
| downloadProgress | `function(event)`              | Callback function to handle the [ProgressEvent](https://developer.mozilla.org/en-US/docs/Web/API/ProgressEvent) of downloads |

## Response

A request resolves to a response object with the following properties and methods:

| Property   | Type                       | Description                             |
| ---------- | -------------------------- | --------------------------------------- |
| url        | `string`                   | Response URL origin                     |
| body       | `Object`, `Blob`, `string` | Response body                           |
| headers    | `Header`                   | Response Headers object                 |
| ok         | `boolean`                  | HTTP status code between 200 and 299    |
| status     | `number`                   | HTTP status code of the response        |
| statusText | `string`                   | HTTP status text of the response        |
| **Method** | **Type**                   | **Description**                         |
| text()     | `Promise`                  | Resolves the body as string             |
| json()     | `Promise`                  | Resolves the body as parsed JSON object |
| blob()     | `Promise`                  | Resolves the body as Blob object        |

## Example

```js
{
  // POST /someUrl
  this.$http.post('/someUrl', {foo: 'bar'}).then(response => {

    // get status
    response.status;

    // get status text
    response.statusText;

    // get 'Expires' header
    response.headers.get('Expires');

    // get body data
    this.someData = response.body;

  }, response => {
    // error callback
  });
}
```

Send a get request with URL query parameters and a custom headers.

```js
{
  // GET /someUrl?foo=bar
  this.$http.get('/someUrl', {params: {foo: 'bar'}, headers: {'X-Custom': '...'}}).then(response => {
    // success callback
  }, response => {
    // error callback
  });
}
```

Fetch an image and use the blob() method to extract the image body content from the response.

```js
{
  // GET /image.jpg
  this.$http.get('/image.jpg', {responseType: 'blob'}).then(response => {

    // resolve to Blob
    return response.blob();

  }).then(blob => {
    // use image Blob
  });
}
```

## Interceptors

Interceptors can be defined globally and are used for pre- and postprocessing of a request. If a request is sent using `this.$http` or `this.$resource` the current Vue instance is available as `this` in a interceptor callback.

### Request processing
```js
Vue.http.interceptors.push(function(request) {

  // modify method
  request.method = 'POST';

  // modify headers
  request.headers.set('X-CSRF-TOKEN', 'TOKEN');
  request.headers.set('Authorization', 'Bearer TOKEN');

});
```

### Request and Response processing
```js
Vue.http.interceptors.push(function(request) {

  // modify request
  request.method = 'POST';

  // return response callback
  return function(response) {

    // modify response
    response.body = '...';

  };
});
```

### Return a Response and stop processing
```js
Vue.http.interceptors.push(function(request) {

  // modify request ...

  // stop and return response
  return request.respondWith(body, {
    status: 404,
    statusText: 'Not found'
  });
});
```

### Overriding default interceptors

All default interceptors callbacks can be overriden to change their behavior. All interceptors are exposed through the `Vue.http.interceptor` object with their names `before`, `method`, `jsonp`, `json`, `form`, `header` and `cors`.

```js
Vue.http.interceptor.before = function(request) {

  // override before interceptor

};
```