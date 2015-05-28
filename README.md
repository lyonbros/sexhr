# Sexhr

Sexhr provides a simple API around XHR2 without limiting any of its power. The
goal is to have a tiny promise-enabled library that makes working with XHR2
simpler.

[Example usage](#examples)

## API

Sexhr only has one exported function.

### Sexhr (request)

This function performs the request and returns a promise. If the browser making
the request lacks promises, [please include Bluebird](https://github.com/petkaantonov/bluebird)
or another library like it.

`Sexhr()` takes a single objcet that describes the request. Its options are as
follows:

- `url` - (string) The URL we're requesting. If this is absent, an error is thrown.
- `method` - (string) The HTTP method we're sending. Defaults to 'get'.
- `data` - (any) The data we pass to `XMLHttpRequest.send`. This can be a string,
a blob, form data, etc.
- `querydata` - (object) A set of key/value pairs to write to the querystring. If
the given URL already has a querystring, then we append to it.
- `emulate` - (boolean) If true and `method` is not "get" or "post", then the
value passed to `method` will be put into a URL param in the request. For instance
if `{url: '/slappy', method: 'delete', emulate: true}` is given, then the final
request would be `POST /slappy?_method=DELETE`.
- `response_type` - (string) The type of response we want. See the
[MDN responseType docs](https://developer.mozilla.org/en-US/docs/Web/API/XMLHttpRequest#xmlhttprequest-responsetype)
for possible values. Sexhr resolves with the XMLHttpRequest.response object, so
`response_type` will affect what the value the returned promise gets.
- `timeout` - (number) Milliseconds to wait before the request times out (is
passed directly to `XMLHttpRequest.timeout`).
- `headers` - (object) This is a hash of headers to set into the XHR object.
- `on*` - (function(\*)) Allows setting callbacks directly onto the XHR object. For
instance, you could pass `Sexhr({url: '/file/123', onprogress: myprogressfn})`
and `myprogressfn()` would be called whenever the XHR's progress event fired.
Note that `on*` settings are disabled for `onload`, `onerror`, and `onabort`
because Sexhr uses these directly.
- `upload.on*` - (function(\*)) Allows setting upload callbacks directly onto the
`XMLHttpRequest.upload` object. Example:
`Sexhr({url: '/files', method: 'post', upload: {onprogress: myuploadprogress}})`
- `override` - (function(XHR)) If passed, Sexhr will pass its `XMLHttpRequest`
object to this function just before it sends it out, allowing you to make any
needed overrides here. If you choose to override `onload`, `onerror`, or
`onabort` then you void Sexhr's nonexistent warranty.

Sexhr makes no assumptions about the data you pass or the data that's handed
back, so it doesn't do anything like JSON encoding/decoding. That's all up to
you.

The error handler (see [examples](#examples)) will be triggered if the HTTP
status code is >= 400.

## Examples

GET a JSON object:
```javascript
Sexhr({url: '/data/averages.json'})
    .spread(function(json, xhr) {
        // "json" is returned as a string so we must parse it ourselves
        myapp.averages = JSON.parse(json);

        // we can look at the return headers as well
        console.log('content type: ', xhr.getResponseHeader('Content-Type'));
    })
    .catch(function(err) {
        // access the XHR object:
        var type = err.xhr.getResponseHeader('Content-Type');

        // get the error code. this will be -1 on XHR error, -2 if aborted, and
        // if the HTTP status code of the XHR request was >= 400, it's the value
        // in xhr.status (the HTTP error code)
        var code = err.code;

        // get the error message. this will be 'error' on XHR error, 'aborted'
        // if aborted, and otherwise will be the HTTP response body for the
        // request if the HTTP status code is >= 400
        var errmsg = err.msg;
    });
```

POST some data:
```javascript
Sexhr({url: '/uploads', method: 'post', data: 'mai text file'})
    .spread(function(res) {
        console.log('done: ', res);
    })
```

DELETE a resource:
```javascript
// actual request will be "POST /posts/10?_method=DELETE"
Sexhr({url: '/posts/10', method: 'delete'})
    .then(function() {
        conosle.log('deleted!');
    });
```

## License

MIT.

